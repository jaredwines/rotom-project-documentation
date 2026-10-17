# 02 - Rotom Docker Services Inventory

**Documentation set:** Rotom Runbook
**Document role:** Canonical source for Docker/Compose service inventory and deployment details
**Scope:** Rotom workload layer; the current runtime baseline is the 2026-10-16 inventory in section 1
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-16 — Reconcile recovery-critical topology
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `03-Network-and-Domains.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1. Current Docker Runtime and Deployment State** and **2. Current Service Contracts** for the supported runtime baseline.
- Use **6. Reconstruction-Critical Information** only after reconciling it with section 1 and current live Compose/container evidence.
- Treat the superseded `infra`, `media`, `gameserver`, `smarthome`, and `web` domain names as retired unless a current section explicitly identifies a surviving artifact.
- Use **Outstanding / Needs Verification** before relying on an unverified behavior, and **Related Documentation** for storage, network, identity, backup, and automation ownership.
- This file owns Docker/Compose deployment facts; it does not replace the canonical storage, network, backup, or permission contracts in documents 03–07.

## Start by Task or Symptom

- For a stopped or missing supported container, failed health check, direct application failure, or Docker-backed website outage, use the procedure immediately below.
- If the direct listener works but proxy, TLS, DNS, routing, or public access fails, continue in [document 03](03-Network-and-Domains.md#diagnose-a-proxy-listener-dns-tls-routing-or-external-access-failure).
- If a bind source is an unavailable NFS path, use [document 04](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount) before changing Docker state.

## Diagnose and Recover a Container or Docker-Backed Website

### 1. Outcome

Identify whether a website failure is in the container, host listener, Nginx Proxy Manager path, or external path. Recover only a website whose desired running state is already established, then verify its direct and proxied responses without changing persistent content or unrelated routes.

### 2. Use this runbook when

- a Docker-backed website is unavailable;
- Nginx Proxy Manager returns `502` for a website;
- a known active website container is stopped or absent;
- the direct host listener works but the HTTPS route does not.

Do **not** use this runbook to:

- decide whether a site should be deployed;
- edit website content, Compose, NPM, DNS, certificates, firewall, or router policy;
- recover a database-backed application;
- restart Docker or Nginx Proxy Manager as a blanket first step.

### 3. Impact and authorization

A read-only diagnosis has no intended outage. Starting an existing stopped container affects only that container. `docker compose up -d` can create or replace the selected project and must be used only after its desired state, Compose path, bind source, and image are confirmed.

Nginx Proxy Manager is shared by all proxied services. Do not restart or recreate it merely because one site fails. Never print environment files, credentials, private keys, or certificate private material.

### 4. Canonical current-state inputs

| Site | Desired-state status | Compose file | Container | Direct check | Proxied check |
|---|---|---|---|---|---|
| Aloha Millworks | Supported active site | `/srv/rotom/stacks/hosted-websites/alohamillworks.com/compose.yaml` | `alohamillworks.com` | `http://127.0.0.1:7778/` → `200` | `https://alohamillworks.com/` → `200` |
| Jared Wines | Supported active site | `/srv/rotom/stacks/hosted-websites/jaredwines.com/compose.yaml` | `jaredwines.com` | `http://127.0.0.1:7777/` → `200` | `https://jaredwines.com/` → `200` |

The final 2026-10-15 recovery established `jaredwines.com` as intentionally active with loopback-only publication `127.0.0.1:7777:80`. Starting or recreating it is authorized only under the same confirmed-active preflight used for Aloha; changing its content, listener, NPM route, or public exposure remains outside this runbook.

NPM runs host-networked from `/srv/rotom/stacks/platform/nginx-proxy-manager/compose.yaml`. Current routes use host or loopback listeners rather than cross-project Docker DNS.

### 5. Read-only preflight

Set the exact known-active target. The example commands below are deliberately scoped to Aloha Millworks.

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

### 6. Stop conditions

Stop before changing state if:

- the target is Jared Wines or any site's desired running state is unclear;
- the live Compose label points to a different file;
- the bind source, image, ownership, or published port differs from the canonical reference;
- Compose rendering fails;
- the logs indicate missing or damaged persistent content rather than a container-start problem;
- more than one unrelated route is failing, NPM itself is unhealthy, or ports `80`/`443` are unavailable;
- recovery would require editing Compose, NPM, DNS, TLS, firewall, router, or application content.

### 7. Procedure

#### Existing supported container is stopped

Start only the exact existing container. This is smaller than recreating the project.

**Changes Rotom**

```bash
sudo docker start alohamillworks.com
```

Wait a few seconds, then perform all verification in section 8.

#### Supported container is absent but Compose and bind source match

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

#### Direct listener works but local-SNI fails

Do not recreate the website. Inspect NPM state and recent logs first.

**Read-only**

```bash
sudo docker inspect nginx-proxy-manager \
  --format 'running={{.State.Running}} status={{.State.Status}} error={{json .State.Error}} restarts={{.RestartCount}}'
sudo docker logs --tail 150 nginx-proxy-manager
sudo ss -ltnp | rg ':(80|443)\b'
```

If NPM is down or its logs show a broader proxy failure, stop and use a separately authorized NPM recovery procedure. Restarting shared NPM is outside this single-site pilot.

### 8. Verification

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

### 9. Rollback or recovery

Starting an existing supported container has no separate rollback; if it immediately fails, leave it stopped, retain its inspection and logs, and investigate the cause.

If `docker compose up -d` created or replaced a container and verification fails, do not run `compose down`, remove images, or delete application data as an automatic rollback. Preserve the current object and logs. Restore a prior image or configuration only from a separately verified applicable recovery point and with explicit scope.

### 10. Troubleshooting branches

- **Container absent + desired state confirmed + Compose valid:** use the targeted Compose procedure.
- **Container stopped + mount/start error:** follow the NAS runbook if the error names `/mnt/nas-*`; otherwise preserve the exact error and investigate the application/container cause.
- **Container running + no direct listener:** compare rendered ports with document 03. Stop if they differ; do not invent a compatibility port.
- **Direct `200` + local-SNI failure:** inspect NPM; do not recreate the site.
- **Local-SNI works + public failure:** investigate public DNS, WAN routing, and certificate state under the network reference; the container is not the failing layer.
- **Jared Wines direct or proxied failure:** its supported current result is `200` through loopback, local-SNI, and public HTTPS. Diagnose the container and route layers without changing the loopback-only publication or shared NPM configuration.

### 11. Evidence to retain

Record the site, desired-state authority, Compose path, initial container state, direct/local-SNI/public response codes, action taken, final state, and any discrepancy. Do not retain cookies, authentication headers, environment values, or certificate private material.

### 12. Related documentation

- [02-Rotom Docker Services Inventory](02-Docker-Services.md)
- [03-Network and Domains](03-Network-and-Domains.md)
- [NAS mount recovery](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount)

### 13. Maintenance notes

Revalidate this runbook when a website's desired deployment state, Compose path, image, bind source, published port, NPM upstream, certificate, or network model changes. Add additional sites only after their direct and proxied expected responses are verified.


## 1. Current Docker Runtime and Deployment State

**Evidence date:** 2026-10-16.

The final live topology contains 23 running containers, 19 Docker networks, two named volumes, and 18 active Compose definitions. The extra network is the unattached `rotom-docs_default` network retained with the inactive rollback stack; its existence is not cleanup authorization. Every supported application is Compose-managed under `/srv/rotom/stacks`.

| Domain | Compose definitions | Supported services |
|---|---:|---|
| Platform | 5 | Nginx Proxy Manager, Homepage with Glances, Arcane, Cloudflare DDNS, Rotom Support |
| Libraries | 7 | Jellyfin, Radarr, Sonarr, Prowlarr, qBittorrentVPN, RomM, Paperless |
| Hosted Websites | 2 | Aloha Millworks, Jared Wines |
| Game Servers | 2 | Palworld Jared, Palworld Fran |
| Smart Home | 2 | Home Assistant, Homebridge |

The runtime uses local Docker root `/var/lib/docker`. Supported mutable state is under `/srv/rotom/appdata/<domain>`; protected inputs are under `/srv/rotom/secrets/<domain>`; declarative Compose definitions are under `/srv/rotom/stacks/<domain>`.

## 2. Current Service Contracts

| Service | Status | Important contract |
|---|---|---|
| Homepage and Glances | Supported active pair | One Compose file at `/srv/rotom/stacks/platform/homepage/compose.yaml`; Glances has no host listener |
| Nginx Proxy Manager | Supported active service | Host networking; authoritative state under Platform appdata; no Docker socket |
| Arcane | Supported active service | Platform stacks are mounted for management; automation policy is recorded in document 06 |
| Rotom Support | Supported active service | Read-only generated site from `/srv/rotom/{stacks,appdata}/platform/mkdocs-rr`; `mkdocs-rr` retains the loopback listener `127.0.0.1:8082` |
| Aloha Millworks | Supported active site | Host publication `7778:80` |
| Jared Wines | Supported active site | Loopback-only `127.0.0.1:7777:80`; NPM proxy host `8` |
| Libraries applications | Supported active services | Authoritative NAS payload under `/mnt/nas-libraries`; non-creating binds where loss would be unsafe |
| Paperless | Supported active service | VM-local database/configuration plus `/mnt/nas-libraries/paperless` media |
| qBittorrentVPN | Supported active service | Libraries torrent/sentinel contract, WireGuard, loopback UI `127.0.0.1:8080` |
| Palworld Jared and Fran | Supported active services | VM-local state; UDP publications unchanged; both healthy at the evidence date |
| Home Assistant and Homebridge | Supported active services | VM-local state under Smart Home appdata; host networking retained for discovery; each Compose file explicitly uses DNS `192.168.1.1` |

Container state is time-sensitive. Diagnose current runtime state with the runbook above instead of relying on the evidence-date count alone.

Home Assistant's ZHA integration is configured for a ConBee II at `/dev/ttyACM0`. PVE VM 100 has USB passthrough for vendor/product `1cf1:0030`, configured by Jared. After the 2026-10-16 PVE reboot, the device was visible on Rotom and in the Home Assistant container. The numbered device path can change, and individual Zigbee device operation was not verified. Historical ConBee disconnection during PVE stability testing did not establish the cause of earlier resets; monitor PVE stability after reconnection.

For a ZHA missing-device error, check the device on Rotom and inside Home Assistant before changing configuration. If absent on Rotom, Jared must inspect the physical adapter and PVE passthrough. If present on Rotom but absent in the container, assess container startup timing and the existing Compose configuration before any targeted recovery. Preserve Zigbee state and avoid replacing the coordinator or resetting its network as a diagnostic step.

## 3. Docker Networking

Active workloads use private Compose-created networks, documented host networking, or host/loopback listeners. The former shared external bridges `rotom-proxy`, `rotom-arr`, and `rotom-monitoring` are not referenced by active Compose definitions.

Homepage reaches Glances through service DNS on their private Compose network. NPM remains host-networked and reaches supported upstreams through documented host or loopback listeners. Prowlarr, Radarr, Sonarr, and qBittorrentVPN do not require a cross-project shared Docker bridge.

## 4. NAS-Backed Container Safety

The scoped recovery helper validates `/mnt/nas-libraries`, its required `library/` and `paperless/` directories, and the applicable sentinel before considering container recovery. It may recover only supported NAS-dependent containers with an eligible restart policy and a recorded NAS/mount startup-failure signature. It leaves running and intentionally stopped containers unchanged.

Do not recreate a missing bind source as a local directory. Stop if an expected path is not the documented NFS filesystem or if the source, ownership, mode, Compose definition, or recovery intent differs.

## 5. Retired Services and Deployment Material

Uptime Kuma, Syncthing, Gamarr, JDownloader, the qBittorrent Games importer, Igir automation, and the browser-upload bridge are retired. Their former Compose/appdata roots and dedicated listeners are absent. Do not redeploy them from repository history or dated change-log entries.

The former standalone Glances stack is retired, but Glances itself is current as the second service in Homepage's Compose file.

The former Downloader, Documents, Filesync, singular Library, Infra, Media, Web, Gameserver, and Smarthome deployment roots are historical names. Current roots use the five domains in section 1.

The local `/srv/rotom/backup-staging` root is absent. Historical statements about fast local rollback archives below that path do not establish current availability; use document 05 to select and verify a retained recovery point.

## 6. Reconstruction-Critical Information

- Preserve numeric service identities and ownership contracts from document 07.
- Preserve the exact current NFS sources and mountpoints from document 04.
- Preserve non-creating NAS binds, qBittorrent sentinel checks, and current container restart policy.
- Preserve NPM database/certificate state, Home Assistant/Homebridge state, both Palworld worlds, Paperless VM-local state, and authoritative NAS payload.
- Do not place secret values in Compose files or the RR.
- Use a selected, verified backup or snapshot rather than reconstructing a retired service from historical prose.

## 7. Outstanding / Needs Verification

- Arcane automatic update and auto-heal are enabled; exact exclusions and intended policy require owner confirmation.
- Two Arcane-related named volumes and any unreferenced images require authenticated Arcane-policy and exact-object review before cleanup.
- Actual residual Docker network/image objects were not enumerated during the documentation cleanup because privileged Docker access was unavailable.
- Palworld persistence intent relative to the earlier stopped state remains unresolved.

## 8. Related Documentation

- [03 — Network and Domains](03-Network-and-Domains.md)
- [04 — NAS and Storage](04-NAS-and-Storage.md)
- [05 — Backup and Restore](05-Backup-and-Restore.md)
- [06 — Maintenance and Automation](06-Maintenance-and-Automation.md)
- [07 — Users and Permissions](07-Users-and-Permissions.md)

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate this document after a Compose path, container, image policy, service identity, bind source, listener, network, restart policy, healthcheck, recovery rule, or supported deployment changes.
