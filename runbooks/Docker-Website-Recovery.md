# Runbook: Diagnose and Recover a Docker-Backed Website

**Document role:** Operational pilot runbook  
**Status:** Verified for diagnosis; targeted recovery procedure reviewed against live configuration  
**Applies to:** Docker-backed websites on the Rotom VM behind Nginx Proxy Manager  
**Risk:** Read-only diagnosis; low-to-moderate targeted container recovery  
**Authorization:** Diagnosis is read-only. Starting or recreating a site requires confirmed desired deployment state. Proxy, DNS, TLS, firewall, application-data, or Compose changes require separate scope.  
**Last live verification:** 2026-10-15 — live containers, Compose projects, website Compose definitions, NPM host-network model, listeners, and documented probes inspected  
**Canonical references:** [02-Docker-Services.md](../02-Docker-Services.md), [03-Network-and-Domains.md](../03-Network-and-Domains.md), [07-Users-and-Permissions.md](../07-Users-and-Permissions.md)  
**Maintenance contract:** [00-Rotom-Change-Log.md](../00-Rotom-Change-Log.md)

## 1. Outcome

Identify whether a website failure is in the container, host listener, Nginx Proxy Manager path, or external path. Recover only a website whose desired running state is already established, then verify its direct and proxied responses without changing persistent content or unrelated routes.

## 2. Use this runbook when

- a Docker-backed website is unavailable;
- Nginx Proxy Manager returns `502` for a website;
- a known active website container is stopped or absent;
- the direct host listener works but the HTTPS route does not.

Do **not** use this runbook to:

- decide whether a site should be deployed;
- edit website content, Compose, NPM, DNS, certificates, firewall, or router policy;
- recover a database-backed application;
- restart Docker or Nginx Proxy Manager as a blanket first step.

## 3. Impact and authorization

A read-only diagnosis has no intended outage. Starting an existing stopped container affects only that container. `docker compose up -d` can create or replace the selected project and must be used only after its desired state, Compose path, bind source, and image are confirmed.

Nginx Proxy Manager is shared by all proxied services. Do not restart or recreate it merely because one site fails. Never print environment files, credentials, private keys, or certificate private material.

## 4. Canonical current-state inputs

| Site | Desired-state status | Compose file | Container | Direct check | Proxied check |
|---|---|---|---|---|---|
| Aloha Millworks | Supported active site | `/srv/rotom/stacks/hosted-websites/alohamillworks.com/compose.yaml` | `alohamillworks.com` | `http://127.0.0.1:7778/` → `200` | `https://alohamillworks.com/` → `200` |
| Jared Wines | **Needs Verification** | `/srv/rotom/stacks/hosted-websites/jaredwines.com/compose.yaml` | `jaredwines.com` | No published host port | Current NPM upstream expects port `7777`; `502` remains expected until intent and listener design are resolved |

The live 2026-10-15 audit found `jaredwines.com` running unexpectedly while still publishing no host port. Its runtime presence does not authorize starting, stopping, recreating, exposing, or repairing it.

NPM runs host-networked from `/srv/rotom/stacks/platform/nginx-proxy-manager/compose.yaml`. Current routes use host or loopback listeners rather than cross-project Docker DNS.

## 5. Read-only preflight

Set the exact known-active target. The pilot commands below are deliberately scoped to Aloha Millworks.

**Read-only**

```bash
sudo docker compose ls --all
sudo docker ps -a --filter name='^/alohamillworks\.com$' \
  --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
sudo docker inspect alohamillworks.com \
  --format 'running={{.State.Running}} status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}} restarts={{.RestartCount}} image={{.Config.Image}} compose={{index .Config.Labels "com.docker.compose.project.config_files"}}'
sudo docker compose \
  -f /srv/rotom/stacks/hosted-websites/alohamillworks.com/compose.yaml \
  config --quiet
sudo docker logs --tail 100 alohamillworks.com
sudo ss -ltnp | rg ':7778\b'
curl -sS -o /dev/null -w 'direct %{http_code}\n' \
  --max-time 10 http://127.0.0.1:7778/
curl -sS -o /dev/null -w 'local-sni %{http_code}\n' \
  --max-time 15 --resolve alohamillworks.com:443:127.0.0.1 \
  https://alohamillworks.com/
curl -sS -o /dev/null -w 'public %{http_code} %{remote_ip}\n' \
  --max-time 20 https://alohamillworks.com/
sudo docker inspect nginx-proxy-manager \
  --format 'running={{.State.Running}} status={{.State.Status}} restarts={{.RestartCount}} network={{.HostConfig.NetworkMode}}'
```

Expected result for Aloha is a valid Compose definition, a running container, a TCP listener on `7778`, and HTTP `200` from the direct, local-SNI, and public probes. Public reachability can fail independently of the local application path; preserve the separate results.

If the target is not Aloha, first obtain its exact current container, Compose path, listener, expected response, persistent-data boundary, and desired deployment state from documents 02/03 and live read-only evidence. Do not substitute a guessed port or project name.

## 6. Stop conditions

Stop before changing state if:

- the target is Jared Wines or any site's desired running state is unclear;
- the live Compose label points to a different file;
- the bind source, image, ownership, or published port differs from the canonical reference;
- Compose rendering fails;
- the logs indicate missing or damaged persistent content rather than a container-start problem;
- more than one unrelated route is failing, NPM itself is unhealthy, or ports `80`/`443` are unavailable;
- recovery would require editing Compose, NPM, DNS, TLS, firewall, router, or application content.

## 7. Procedure

### Existing supported container is stopped

Start only the exact existing container. This is smaller than recreating the project.

**Changes Rotom**

```bash
sudo docker start alohamillworks.com
```

Wait a few seconds, then perform all verification in section 8.

### Supported container is absent but Compose and bind source match

Confirm the content root exists and remains read-only in the rendered Compose model.

**Read-only**

```bash
sudo stat -c '%U:%G %a %n' \
  /srv/rotom/appdata/hosted-websites/alohamillworks.com
sudo docker compose \
  -f /srv/rotom/stacks/hosted-websites/alohamillworks.com/compose.yaml \
  config --quiet
```

Create only the selected project. Do not add `--pull`, delete images, or operate on another stack.

**Changes Rotom**

```bash
sudo docker compose \
  -f /srv/rotom/stacks/hosted-websites/alohamillworks.com/compose.yaml \
  up -d
```

### Direct listener works but local-SNI fails

Do not recreate the website. Inspect NPM state and recent logs first.

**Read-only**

```bash
sudo docker inspect nginx-proxy-manager \
  --format 'running={{.State.Running}} status={{.State.Status}} error={{json .State.Error}} restarts={{.RestartCount}}'
sudo docker logs --tail 150 nginx-proxy-manager
sudo ss -ltnp | rg ':(80|443)\b'
```

If NPM is down or its logs show a broader proxy failure, stop and use a separately authorized NPM recovery procedure. Restarting shared NPM is outside this single-site pilot.

## 8. Verification

**Read-only**

```bash
sudo docker ps --filter name='^/alohamillworks\.com$' \
  --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
sudo docker inspect alohamillworks.com \
  --format 'running={{.State.Running}} restarts={{.RestartCount}} image={{.Config.Image}}'
curl -sS -o /dev/null -w 'direct %{http_code}\n' \
  --max-time 10 http://127.0.0.1:7778/
curl -sS -o /dev/null -w 'local-sni %{http_code}\n' \
  --max-time 15 --resolve alohamillworks.com:443:127.0.0.1 \
  https://alohamillworks.com/
curl -sS -o /dev/null -w 'public %{http_code} %{remote_ip}\n' \
  --max-time 20 https://alohamillworks.com/
sudo docker logs --tail 50 alohamillworks.com
```

Success requires the Aloha container to remain running, restart count not to increase unexpectedly, the `7778` listener to remain present, and all three probes to return `200`. The website bind must remain the existing read-only source, and no unrelated container should be recreated.

## 9. Rollback or recovery

Starting an existing supported container has no separate rollback; if it immediately fails, leave it stopped, retain its inspection and logs, and investigate the cause.

If `docker compose up -d` created or replaced a container and verification fails, do not run `compose down`, remove images, or delete application data as an automatic rollback. Preserve the current object and logs. Restore a prior image or configuration only from a separately verified applicable recovery point and with explicit scope.

## 10. Troubleshooting branches

- **Container absent + desired state confirmed + Compose valid:** use the targeted Compose procedure.
- **Container stopped + mount/start error:** follow the NAS runbook if the error names `/mnt/nas-*`; otherwise preserve the exact error and investigate the application/container cause.
- **Container running + no direct listener:** compare rendered ports with document 03. Stop if they differ; do not invent a compatibility port.
- **Direct `200` + local-SNI failure:** inspect NPM; do not recreate the site.
- **Local-SNI works + public failure:** investigate public DNS, WAN routing, and certificate state under the network reference; the container is not the failing layer.
- **Jared Wines `502`:** expected under the unresolved current design. Stop and request a deployment-intent decision.

## 11. Evidence to retain

Record the site, desired-state authority, Compose path, initial container state, direct/local-SNI/public response codes, action taken, final state, and any discrepancy. Do not retain cookies, authentication headers, environment values, or certificate private material.

## 12. Related documentation

- [02-Rotom Docker Services Inventory](../02-Docker-Services.md)
- [03-Network and Domains](../03-Network-and-Domains.md)
- [NAS Mount Recovery](NAS-Mount-Recovery.md)

## 13. Maintenance notes

Revalidate this runbook when a website's desired deployment state, Compose path, image, bind source, published port, NPM upstream, certificate, or network model changes. Add additional sites only after their direct and proxied expected responses are verified.

