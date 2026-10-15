# 02 - Rotom Docker Services Inventory

**Documentation set:** legacy documentation set  
**Document role:** Canonical source for Docker/Compose service inventory and deployment details  
**Scope:** Rotom workload layer; the current runtime baseline is the 2026-10-15 inventory in section 1A  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-15 — Final acceptance reconciliation
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `03-Network-and-Domains.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1A. Current Docker Runtime and Deployment State** for the supported runtime baseline and newest dated overlays.
- Use **Reconstruction-Critical Information** only after reconciling it with section 1A and current live Compose/container evidence.
- Sections whose titles name `infra`, `media`, `gameserver`, `smarthome`, `web`, or other superseded domains are preserved migration history unless a newer current section explicitly reaffirms them.
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


## 1. Purpose and Scope

### Current Paperless Libraries Compose deployment — 2026-10-13

Paperless, `paperless-db` (PostgreSQL 18), and `paperless-broker` (Valkey 9) are active from `/srv/rotom/stacks/libraries/paperless/compose.yaml`. The restored non-media state is VM-local at `/srv/rotom/appdata/libraries/paperless/{broker,database,data,export,consume}`; protected environment inputs are root-owned below `/srv/rotom/secrets/libraries/paperless`. The web container maps processes to Libraries `127:5000`, publishes only `127.0.0.1:8000`, and uses only `paperless_default`; database and broker have no host ports. Its former `rotom-proxy` attachment is retired.

The sole media bind is `/mnt/nas-libraries/paperless -> /usr/src/paperless/media`. It uses `create_host_path: false`; the target is `127:5000`, mode `2770`. Five restored media files total 384,771 bytes and their SHA-256 manifest matches the retained recovery copy. Compose rendering succeeded, all three containers run, and Paperless is healthy. Historical Documents/Paperless paths below are recovery evidence, not active Compose guidance.

### Current Libraries local Compose roots — 2026-10-13

The active local Library Compose, appdata, and secrets roots are `/srv/rotom/{stacks,appdata,secrets}/libraries`; the `libraries` Linux identity retains `127:5000`. Jellyfin, Radarr, Sonarr, RomM, qBittorrentVPN, and Prowlarr were recreated from those roots and their live container mounts use plural appdata paths. The singular local roots and account are absent. This does not alter `/mnt/nas-libraries` or its internal `library/` media directory. The transient singular appdata tree created during the rename was later retired; `/srv/rotom/backup-staging` was absent on 2026-10-14.

### Current JAR-96 Library mount contract — 2026-10-13

Jellyfin, Radarr, Sonarr, RomM, qBittorrentVPN, and Nginx Proxy Manager were recreated with Library binds from `/mnt/nas-libraries`; no active container bind uses `/mnt/nas-library` or `/mnt/nas-media`. qBittorrent keeps `/mnt/nas-libraries/torrents` at `/media/torrents` and the read-only, non-creating sentinel bind at its unchanged container path. NPM's internal game-library alias and Homepage's Library storage metric also use the new host mount. qBittorrent `wg0` and all affected local HTTP checks passed after recreation. After its reboot bind-startup race, NPM was restored and added to the targeted NAS recovery helper; public Homepage HTTPS then returned `200`. The retired local `/mnt/nas-media` directory is absent; historical Media-bind records below are not current Compose guidance.

### Current local-domain alignment — 2026-10-13

The active website Compose/appdata roots are `/srv/rotom/{stacks,appdata}/hosted-websites`, owned by `hosted-websites` `902:5004`; old `web` paths are historical. Aloha Millworks was recreated from the new roots and public HTTPS returned `200`. Jared Wines remained undeployed at this 2026-10-13 checkpoint; the complete 2026-10-15 runtime audit below supersedes that live-state observation. The guest `/mnt/nas-web` contract is retired locally and is not an application bind.

The active Palworld Compose/appdata/secrets roots are `/srv/rotom/{stacks,appdata,secrets}/game-servers`, owned by `game-servers` `995:5001`; old `gameserver` paths are historical. Jared and Fran were running and healthy on 2026-10-15. Their labels and binds resolve only to the new local roots; no world data or NAS Gameserver content was changed by this documentation reconciliation. The reason and intended persistence of the change from the earlier stopped state remain unverified.

### Current Smart Home local identity — 2026-10-13

The active Smart Home identity is `smart-home` (`126:5003`), with NSS home field `/home/smart-home` pointing to an absent physical directory, root-administered Compose modules under `/srv/rotom/stacks/smart-home`, and mutable state under `/srv/rotom/appdata/smart-home`. `smarthome` is absent from NSS and the current local-root layout. No container binds `/mnt/nas-smart-home`; that empty NFS boundary remains separate from Home Assistant and Homebridge's VM-local state.

This inventory is based on the uploaded `rotom-docker-containers.txt` (container names, images, statuses, and ports) and `rotom-compose-locations.txt` (Compose file paths), with verified Docker inspection and status updates from the conversation incorporated on 2026-09-14. The original table statuses are recorded observations rather than continuously live values; later dated verification sections incorporate fresh Docker state and health checks where they were explicitly captured. The original source files do not specify a capture timestamp.

### Documentation-audit reconciliation — 2026-10-13

The current deployment paths are the plural `libraries`, `platform`, `hosted-websites`, `game-servers`, and `smart-home` roots described in the current overlays. Older entries that call singular `library`, `infra`, `web`, `gameserver`, `smarthome`, `/mnt/nas-library`, or `/mnt/nas-media` current are historical migration/recovery evidence. Final 2026-10-15 inspection found 23 container objects, all running, including active Jared Wines and both healthy Palworld containers; it also found 18 networks, two named volumes, and 18 Compose definitions. Treat runtime status as time-sensitive and recheck it read-only when troubleshooting.

### Evidence provenance

Latest complete runtime inventory: 2026-10-15. Earlier JAR checkpoints remain dated migration and recovery evidence.
Project: **Rotom-Home-Server**

The 2026-09-16 identity update below uses supplied Rotom/UNAS command output from the historical “Torrent Seeding Explanation” conversation. The 2026-09-18 backup-access work additionally verified the live Homepage/Glances Compose project, removed Glances' backup-share bind, recreated that two-service project, and confirmed both services healthy/running afterward. Later 2026-09-18 supplied output verified the Palworld service-account rename to `game`, home move to `/home/game`, Compose recreation from the new paths, live bind mounts, runtime numeric identity, health, and unchanged world identifiers. Supplied 2026-09-19 output then verified the separate `game` GID migration from `985` to `5001`, both Palworld Compose identities at `995:5001`, healthy recreated containers against the existing data trees, and the dedicated `/mnt/nas-game` NFS commissioning. Later supplied 2026-09-19 migration output verified Prowlarr copied in a stopped state from `/home/media/docker/prowlarr` to the then-current `/home/rotom/docker/prowlarr`, recreated with `PUID=997` / `PGID=986`, running as the then-named `rotom:rotom`; the 2026-09-21 service-account rename subsequently moved that tree to `/home/infra/docker/prowlarr` without changing numeric IDs, preserving port `9696`, `/config` data, indexers, history, Sonarr/Radarr integrations, and reverse-proxy behavior. The same day qBittorrentVPN was migrated from `/home/media/docker/qbittorrentvpn` to the then-current `/home/rotom/docker/qbittorrentvpn`, changed to `PUID=997` / `PGID=5000`; the 2026-09-21 rename subsequently moved that project to `/home/infra/docker/qbittorrentvpn`, given Media/Game torrent-only binds and NAS fail-closed guards, and reverified for VPN/kill-switch behavior. Sonarr/Radarr were recreated with `/mnt/nas-media -> /media`. Gamarr was installed under `/home/game/docker/gamarr` at `995:5001`, port `6767`, with `/mnt/nas-game -> /game`, and the end-to-end Game download/import/hardlink/seeding workflow was accepted by the user.

Services are grouped by the Linux account directory containing their active Compose file. These directory associations do not by themselves establish the user inside each container. The original inventory had complete Compose working-directory metadata for only a subset of services, but the later 2026-09-19 full verification reconciled all 17 documented container records against the current Compose paths and Docker metadata. Application process identity remains a separate claim and is documented only where it was independently verified.

**Port notation:** Entries containing `->` are published host-to-container mappings. Bare ports are listed separately as container-only ports with no host mapping shown. “None shown” means the source PORTS field was blank; it does not establish network mode or lack of connectivity.

## 1A. Current Docker Runtime and Deployment State

### Current Homepage, Glances, and Jared Wines deployment — 2026-10-15

`homepage` and `glances` are now the two services in `/srv/rotom/stacks/platform/homepage/compose.yaml`, sharing the existing private `homepage_default` bridge. Glances preserves its host PID namespace plus read-only Docker-socket, host-root, and configuration binds, but no longer publishes TCP `61208` on the host. Homepage's eight Glances widgets use `http://glances:61208`, including the Docker Containers metric, and all required Glances API endpoints returned `200` from Homepage. The active Glances configuration is `/srv/rotom/appdata/platform/homepage/glances/glances.conf`; `/srv/rotom/stacks/platform/glances` and `/srv/rotom/appdata/platform/glances` are retired and absent. No new Docker network was created; `glances_default` was removed when the standalone stack was retired.

`jaredwines.com` is active with a read-only site-content bind and loopback-only publication `127.0.0.1:7777:80`. NPM proxy host `8` retains the `127.0.0.1:7777` HTTP upstream and forced TLS. After regenerating its stale active proxy configuration from that record, the loopback, local-SNI TLS, and public HTTPS routes all returned `200` with the expected portfolio page. Homepage's internal container-host mapping for `jaredwines.com` routes its monitor through NPM; it does not alter public DNS or make port `7777` LAN-reachable.

The final live topology contains 23 running containers, 18 Docker networks, two named volumes, and 18 Compose definitions: two Game Servers, two Hosted Websites, seven Libraries modules, five Platform modules, and two Smart Home modules. The reduction from nineteen definitions is the expected result of integrating Glances into the Homepage Compose project and retiring the standalone Glances definition.

This overlay supersedes earlier same-day standalone-Glances, loopback-`61208`, and unavailable-Jared-Wines statements.

### Historical same-day post-cleanup runtime audit — 2026-10-15

Superseded by **Current Homepage, Glances, and Jared Wines deployment** above. This checkpoint predates the final Jared Wines listener restoration and Homepage/Glances integration.

The final read-only snapshot contained 23 containers, all running, with zero restarts and `restart: unless-stopped`; 19 networks; two named volumes; and 23 images occupying approximately 21 GB. Docker Engine/client are `29.9.0`, Compose is `v5.6.0`, and containerd is `2.4.1`. All 19 Compose definitions under `/srv/rotom/stacks` rendered successfully. Every container is Compose-managed, no one-off container exists, and each live service's Compose file, project, service, and config hash match the current definition. Six configured Docker healthchecks were healthy: Homepage, Arcane, Paperless, RomM database, and both Palworld servers. The remaining images define no Docker healthcheck.

The intended 22-container deployment is complete and functional: documented bind sources exist; Library NAS mounts and modes match Compose; qBittorrentVPN retains `wg0` at `10.2.0.2/32`; all documented unauthenticated local and local-SNI HTTP probes returned their expected `200`/`302` responses; and no intended service is missing, stopped, unhealthy, or dependent on a removed shared bridge. `/mnt/nas-libraries` and `/mnt/nas-rotom-restic-backup` were active NFSv3 read/write mounts. The Game Servers, Smart Home, and Hosted Websites automounts were armed but idle and were deliberately not activated by this read-only audit.

`jaredwines.com` is unexpectedly deployed as the twenty-third container, with `jaredwinescom_default` as the nineteenth network. It has no published port, so NPM's retained `127.0.0.1:7777` upstream still returns `502`. Docker events show Arcane stopped, destroyed, recreated, and started this container at 10:40 PDT while replacing `nginx:alpine`; Arcane recorded the same event as a successful non-dry-run automatic update. The updater left the prior Nginx image untagged and unreferenced. The cause of Jared Wines' original inclusion in the earlier full recreation and its desired deployment state remain **Needs Verification**.

Both named volumes are unattached and report `0B`: `arcane-backups` and `arcane-rustic-cache`. The latter carries Arcane internal volume-helper labels. `ghcr.io/rustic-rs/rustic:v0.11.2` is not referenced by a container, while Arcane's system/volume backup policy and run tables are empty. Removal safety for either volume or the Rustic image is unverified: first confirm Arcane's authenticated backup settings/API and inspect Arcane activity/log evidence around their 04:15 PDT creation. The prior Nginx image likewise requires a final exact-ID ancestor check plus confirmation that no Arcane update/rollback is in flight before any separately authorized cleanup. The untagged RomM, MariaDB, and pinned Rotom Docs images are active container images and are not cleanup candidates. The retired Syncthing image is absent.

### Historical same-day shared Docker-network retirement and verified recovery — 2026-10-15

Superseded for current container, network, volume, Compose-count, Jared Wines, and Glances state by the first section 1A overlay above. The shared-network retirement itself remains current.

At Jared's direction, the existing Docker container objects were deleted and all intended deployed stacks were recreated; Jared Wines remains intentionally undeployed. The three former external bridge contracts `rotom-proxy`, `rotom-arr`, and `rotom-monitoring` were removed from all active Compose definitions and then removed from Docker. No active Compose file or running network references any of those names.

NPM remains host-networked and uses the documented host/loopback upstream listeners. The retained Arr/Prowlarr application settings use `192.168.1.69` host addresses, so Prowlarr/Radarr/Sonarr/qBittorrentVPN no longer require cross-project Docker DNS. Glances is published only as `127.0.0.1:61208:61208`; Homepage uses its existing `host.docker.internal` gateway mapping and eight Glances widgets now target `http://host.docker.internal:61208`. The change does not expose Glances to the LAN.

All 19 active Compose definitions rendered successfully. Final verification found 22 running container objects, 18 Docker networks, one named volume, healthy Homepage/Arcane/Paperless/RomM database/Palworld containers, qBittorrentVPN `wg0`, working local HTTP endpoints, and successful Homepage-to-Glances API access. Root-only pre-change archives are retained at `/root/rotom-proxy-compose-pre-removal-20261015-012109.tar.gz`, `/root/rotom-arr-compose-pre-removal-20261015-013309.tar.gz`, and `/root/rotom-monitoring-pre-removal-20261015-013632.tar.gz`; retention is **Needs Verification**. The previously documented `/srv/rotom/stacks/RUNTIME-POLICY.md` was absent during this work and was not recreated.

### Historical pre-restoration runtime evidence boundary — 2026-10-15

Superseded by the final section 1A overlay above. Retain this checkpoint only as evidence of the state before Jared Wines was deliberately restored and Glances joined the Homepage Compose project.

Read-only Compose, process, listener, NPM-route, and HTTP evidence confirms the current Platform, Libraries, Smart Home, Hosted Websites, and Paperless topology. Privileged 2026-10-15 Docker inspection found 22 container objects, all running, with one named volume, seventeen networks, and restart count zero for every object. Both Palworld containers were healthy with their expected UDP publications and use `unless-stopped`. Docker inspection dates both starts to 2026-10-14 12:24 PDT while the daemon remained continuously active from the 2026-10-13 boot. The journal records both starts thirty seconds after an operator-run Docker system prune but no explicit start command; do not infer causation or owner intent from that timing. This supersedes the 2026-10-14 stopped-state observation, while intended persistence remains unverified. The active stack tree contains 19 Compose files: two Game Servers, two Hosted Websites, seven Libraries modules, six Platform modules, and two Smart Home modules; Paperless and RomM each define their documented database/broker companions. Jared Wines remains intentionally undeployed.

A later 2026-10-15 read-only follow-up again found all 22 containers running with zero restart counts. The current NAS recovery helper was inspected: after validating `/mnt/nas-libraries` plus its required `library/` and `paperless/` directories, it conditionally recovers only qBittorrentVPN, Radarr, Sonarr, Jellyfin, RomM, Paperless, and Nginx Proxy Manager when a stopped container has an eligible restart policy and a recorded NAS/mount startup-failure signature. It leaves already-running containers and intentionally stopped containers unchanged.

At the 2026-10-14 checkpoint Jared Palworld had exited `143` after SIGTERM and Fran Palworld had exited `137` after an explicit stop request and timeout, with `OOMKilled=false`; those records remain historical stop evidence rather than evidence of a crash. Both containers were running and healthy on 2026-10-15. Jared confirmed review of the retired Syncthing image, untagged images, and dormant Docker artifacts; no resulting retain/remove disposition was supplied, so no cleanup is inferred. No container uses the Syncthing image.

The Smart Home stack directories are `root:root`, but the live `home-assistant/compose.yaml` and `homebridge/compose.yaml` files are owned by `smart-home:smart-home`. The owner-confirmed unintended `game-servers` ACL entries were removed on 2026-10-14. Ownership, modes `0660`/`0670`, Jared's named access, Smart Home read/write access, file contents, and running services were preserved; both Compose definitions passed `docker compose config --quiet`.

### Downloader-remnant retirement — 2026-10-13

The retired Downloader compatibility artifacts are absent: `/home/downloaders`, the JAR-84 local Downloader archive/staging directory, and Arcane's read-only bind of the former home. Active Arcane Compose rendered successfully; its live mounts are limited to Platform appdata, Docker socket, Platform secrets, and Platform stacks. Arcane was `running healthy` and its HTTPS endpoint returned `200`. Active Library Prowlarr/qBittorrentVPN services and NAS data were not changed.

### Current Homepage Platform layout — 2026-10-13

The active Homepage configuration names its management section **Platform**. It contains Nginx Proxy Manager, Arcane, PVE, and Rotom Docs; the former standalone Documents section/layout key is absent. The stale Uptime Kuma card and its `siteMonitor` are absent, consistent with JAR-95's retired container/image and NPM route. The former **Library** service section and matching layout key are named **Libraries**. Its persistent `/srv/rotom/appdata/platform/homepage/config` bind now includes presentation-only `custom.js` and `custom.css`: for the three Backup Schedule cards, `Last Result` renders green for `success` and red for `failed`/`failure`. The custom JavaScript observes Custom API refreshes; neither the backup endpoints nor schedules are changed. Homepage Compose validation passed; Homepage remained healthy after the 2026-10-14 retirement of all 28 inventoried Homepage `.pre-*`/`.orig` rollback copies. Those local copies are no longer available; historical records and applicable Restic snapshots remain separate evidence.

### JAR-95 Uptime Kuma retirement — 2026-10-13

Uptime Kuma is retired. No `uptime-kuma` container or image, Platform Compose stack/appdata, NPM route, or Homepage mapping remains. The recorded post-retirement set includes Homepage, NPM, Arcane, Glances, Rotom Docs, Cloudflare DDNS, Libraries services, Hosted Websites, Smart Home, and the deliberately stopped Palworld containers; no Uptime Kuma object is present. This is not a complete current inventory. It supersedes unqualified current Uptime Kuma statements below, which are retained as dated historical evidence. JAR-95 makes no assertion that a public status page existed.

### Historical JAR-94 Media-to-Library NAS cutover — 2026-10-13

At this checkpoint, Jellyfin, Radarr, Sonarr, RomM, qBittorrentVPN, and Nginx Proxy Manager were recreated against `/mnt/nas-library`. JAR-96 later moved those live binds to `/mnt/nas-libraries`; this section is retained solely for the cutover evidence.

The prior JAR-94 heading below records a separate historical workload-retirement task and must not be used as current service state.

### Historical pre-Libraries qBittorrentVPN/Prowlarr restoration — 2026-10-13

This restoration used singular `library` roots before the local identity was renamed to `libraries`. The current paths are `/srv/rotom/{stacks,appdata,secrets}/libraries`; the remaining runtime checks are historical evidence. At this checkpoint Paperless was absent. The current Paperless deployment is documented in the first current overlay above.

### Historical JAR-94 Platform recovery and retired workloads — 2026-10-13

Superseded for current service/presentation state by the JAR-95 and Homepage overlays above; retained as recovery evidence.

The active Platform deployment contains Nginx Proxy Manager, Homepage, Arcane, Glances, Uptime Kuma, Rotom Docs, and Cloudflare DDNS below `/srv/rotom/{stacks,appdata,secrets}/platform`; the runtime identity is `platform` (`997:5002`). They were recovered from Restic snapshot `02c9c4e7` into a restricted staging directory, reviewed, path-renamed from the snapshot's stale `infra` layout, and promoted before Compose validation/recreation. NPM, Homepage, Arcane, Glances, Uptime Kuma, Rotom Docs, and DDNS are running; Arcane, Homepage, and Uptime Kuma are healthy.

Syncthing, Paperless, Prowlarr, and qBittorrentVPN are retired and have no container objects or active Compose/appdata/secrets roots. The retired Prowlarr/qBittorrent and Paperless network contracts are absent. At that historical checkpoint Arcane still carried a read-only bind of the empty root-owned `/home/downloaders/docker` compatibility remnant; the current Downloader-remnant retirement above records its later removal. Historical sections below preserve former service facts only.

### JAR-93 Platform-domain migration — 2026-10-13

Superseded for current live identity and service state by JAR-94 above; this section is retained as dated JAR-93 evidence.

The active deployment roots for Nginx Proxy Manager, Homepage, Arcane, Glances, Uptime Kuma, Cloudflare DDNS, Rotom Docs, and Syncthing are now `/srv/rotom/stacks/platform/<service>` with corresponding mutable state under `/srv/rotom/appdata/platform/<service>` and protected inputs under `/srv/rotom/secrets/platform` where a service has them. Compose validation passed before recreation. No image, published port, Docker network, service identity, NFS bind, URL, reverse-proxy route, or persistent-data contract changed.

Syncthing's module and state are Platform-owned but its container continues as `903:5007` and keeps `/mnt/nas-filesync:/filesync`. The old `infra` and `filesync` v2 stack/state directories were removed after the moves; the unused `homepage/compose.yaml.orig` file was removed. Homepage now has a Platform section for NPM, Arcane, PVE, Uptime Kuma, Syncthing, and Rotom Docs. PVE has no monitor; the remaining Homepage groupings, including Library and hosted sites, are unchanged.

The Proxmox VE host `pve` remains hypervisor-only; the application Docker runtime is inside the Debian `rotom` VM. Current runtime foundation is Docker Engine `29.9.0`, containerd `2.3.6`, Docker Compose `v5.6.0`, and local Docker root `/var/lib/docker`. Package-created Docker group GID `989` now has one human member, Fran, whose Docker access is intentionally root-equivalent; Jared and all service identities remain outside the group.

JAR-34 created the initially empty v2 declarative namespace at `/srv/rotom`. Subsequent Phase D tickets populated the current domain modules and state described below; the original empty-reservation state is historical. `/srv/rotom` is a local Git repository for only `stacks/`, `scripts/`, and support files; `.gitignore` excludes `appdata/`, `secrets/`, `backup-staging/`, databases, logs, caches, generated state, and environment files.

JAR-88 restores the Gameserver name for the active Palworld domain: locked `gameserver` is `995:5001` with home `/home/gameserver`; `downloader` remains `901:5005`. The unused Customapps identity (`904:5008`) and local future-domain material were retired on 2026-10-12. Palworld Compose/appdata/environment paths are under `/srv/rotom/{stacks,appdata,secrets}/gameserver`; each module was recreated separately against the same VM-local world. Existing active numeric PUID/PGID values remain unchanged. JAR-87 Media-ROM and qBittorrent contracts do not use Gameserver NAS storage.

JAR-36 created external bridge networks `rotom-proxy` (`172.29.0.0/16`), `rotom-arr` (`172.30.0.0/16`), and `rotom-monitoring` (`172.31.0.0/16`). Later migrations attached the applicable services as documented in the current domain sections; workloads use only the cross-stack networks they need and may retain private default networks.

JAR-37 is the accepted Phase C runtime gate. Docker's daemon-wide `json-file` cap is `max-size=10m`, `max-file=3`; the current long-running containers use `restart: unless-stopped`. Healthchecks remain image-specific rather than synthetic: at the JAR-37 checkpoint, Arcane, Homepage, and the then-active Gamarr reported healthy; a missing healthcheck does not by itself indicate a fault. The reusable local policy is `/srv/rotom/stacks/RUNTIME-POLICY.md`: it requires backup/recovery readiness, configuration review, target-stack-only pull/recreation, readiness and application/storage/network/proxy verification, and target-stack-only rollback. Future mutable bind state belongs under `/srv/rotom/appdata/<domain>/...` where image support permits; Compose files must not contain secret values.

### Current NAS-backed recovery verification — 2026-10-10

### Historical JAR-92 Library-domain migration — 2026-10-13

JAR-94 above supersedes the Paperless, Prowlarr, and qBittorrentVPN portions of this dated section; the remaining Library history is retained as evidence.

`library` retains numeric `127:5000` after the active `media` account/group rename. Former Media-domain module paths are now `/srv/rotom/stacks/library/{jellyfin,radarr,sonarr,prowlarr,romm}` with matching appdata/secrets roots. qBittorrentVPN is `/srv/rotom/stacks/library/qbittorrentvpn`, uses `PUID=127`/`PGID=5000`, and retains its `8080` listener, `rotom-arr` network, WireGuard, `/mnt/nas-media/torrents` read/write bind, and `/mnt/nas-media/.rotom-qbt-media-ready` read-only non-creating guard bind. The recovery helper no longer mounts or validates the obsolete Downloader sentinel before considering qBittorrent recovery.

Paperless is `/srv/rotom/stacks/library/paperless` with `USERMAP_UID=127`/`USERMAP_GID=5000`; its web workers run as Library. Its database, broker, configuration/search data, consume, exports, and secrets remain VM-local under the Library Paperless roots, with PostgreSQL/Valkey image-owned state preserved. The sole Paperless NAS media bind remains `/mnt/nas-documents/paperless -> /usr/src/paperless/media` and `create_host_path: false`. Targeted recreation, recovery-helper execution, container/media checks, and HTTPS `302` passed.

The enabled root-owned `/usr/local/sbin/rotom-nas-docker-recovery` service remains deliberately targeted rather than imposing a global Docker-to-NAS boot dependency. It starts and confirms Media, Documents, and Filesync NFS mounts; validates the Media qBittorrent guard, Media library, Paperless media directory, and Filesync root; then leaves already-running containers untouched. It can start qBittorrentVPN, Radarr, Sonarr, Jellyfin, Paperless, RomM, or Syncthing only when the existing container has `unless-stopped`/`always` restart policy and a recorded NAS/mount startup failure. A live recovery run remounted Documents, Media, and Filesync, recovered Paperless, RomM, and Syncthing, and completed successfully. Paperless and Syncthing became healthy; their local endpoints returned `302` and `200`, while RomM returned `200`. Fresh `docker ps -a` then showed 24 container objects, all running, and no failed systemd units.

### JAR-75 Filesync Syncthing — 2026-09-29

Syncthing `2.0.0` runs from `/srv/rotom/stacks/filesync/syncthing/compose.yaml` as numeric `903:5007`, with persistent configuration/database below `/srv/rotom/appdata/filesync/syncthing`. Its sole payload bind is `/mnt/nas-filesync -> /filesync` with `create_host_path: false`; its GUI is loopback-only at `127.0.0.1:8384` for NPM. The disposable acceptance peer and payload were removed, so no production folder or peer is currently configured.

### JAR-77 Gamarr retirement — 2026-09-30

Gamarr is retired. JAR-77 verified its live v2 Compose labels before removing `/srv/rotom/stacks/media/gamarr`, `/srv/rotom/appdata/media/gamarr`, and the legacy `/home/game/docker/gamarr`/standalone Gamarr config. Its container, private default network, port `6767`, Homepage card, NPM route, and recovery-worker branch are absent. At JAR-77 closeout, RomM retained its read-only `/mnt/nas-media/library/games` bind; JAR-76 later superseded that bind with the current read-only Game library path. Prowlarr retains only Radarr/Sonarr applications. At JAR-77 closeout, qBittorrentVPN's `Games` category was retained because its active/seeding state was not authenticated; JAR-79 later removed it after live zero-torrent verification. The targeted root-protected restore archive is `/srv/rotom/backup-staging/media/jar77-20260930/gamarr-config-and-compose.tar.gz`.

### JAR-79 Game torrent-directory retirement — 2026-10-01

At this checkpoint, qBittorrentVPN's local API reported zero torrents; its unused `Games` category was removed after the two associated `Downloader/.data` directories were confirmed empty, then removed with non-recursive `rmdir`. Live Compose inspection established that qBittorrentVPN bound `/mnt/nas-media/torrents` at both `/media/torrents` and `/game/torrents`; JAR-79 made no change to that payload topology. Later JAR-87 material records the final Games-path contract separately. This is dated correction evidence for the conflicting older Downloader-path claims, which remain historical unless separately reconciled.

### Historical JAR-83 minimal Game qBittorrent marker — 2026-10-01 (superseded by JAR-87/JAR-88)

At that checkpoint, the `Game/.data` export was again automounted at `/mnt/nas-game` and contained only `.rotom-qbt-media-ready/` (`901:5000`, mode `2750`) below its `988:5001` mode-`2770` root. qBittorrentVPN then required that marker alongside the Media marker. JAR-87 made the guard Media-only and JAR-88 removed the active Game guest mount; do not restore this marker or bind as current configuration. The controlled marker-absence test remains useful historical fail-closed evidence.

### JAR-87 Media ROM pipeline and Games torrents — 2026-10-01

RomM binds `/mnt/nas-media/library/games:/romm/library:ro`. JDownloader, the Media inbox, Igir, qBittorrent Games importer, and browser-upload bridge were retired on 2026-10-05. qBittorrentVPN continues to use `/mnt/nas-media/torrents` plus the read-only Media sentinel; its `Games` category uses `/media/torrents/games` and `/media/torrents/incomplete/games`. The RomM library and separate review directory remain preserved, and the Game source library remains rollback material.

### JAR-89 qBittorrent Games importer — retired 2026-10-05

### JAR-90 authenticated ROM upload bridge — retired 2026-10-05

The former bridge service, binary, protected credential, loopback `8090` listener, and NPM host `26` were removed on 2026-10-05. No browser-upload ingress remains. RomM's Media library bind remains read-only.

For the RomM route, NPM also has the read-only host bind `/mnt/nas-media/library/games:/mnt/nas-media/library/games:ro` and proxy-host `21`'s generated internal `/library/` alias. The NPM Compose file has no `user`, `PUID`, or `PGID` override: its root container identity is the retained image default, not a JAR-90 privilege change. NPM has no Docker socket. RomM independently binds the same library read-only and its internal Nginx workers run as `romm`.

The importer timer/service, API snapshot helper, importer binary, and durable local state were removed on 2026-10-05 with the inbox. qBittorrentVPN and existing torrent payloads remain unchanged.

### JAR-88 Gameserver module paths — 2026-10-01

JDownloader was moved to `/srv/rotom/stacks/gameserver/jdownloader` with matching local configuration and protected inputs under JAR-88; that standalone module was retired on 2026-10-05, along with its container and private network. Palworld Jared and Fran run from `/srv/rotom/stacks/gameserver/palworld-{jared,fran}` with local binds `/srv/rotom/appdata/gameserver/palworld-{jared,fran} -> /palworld`, not NFS. Both were healthy after individual recreations and retained their existing UDP mappings.

### Historical pre-JAR-88 Game NAS recovery finding — 2026-10-01

During the 12:57 PDT guest boot, the initial `/mnt/nas-game` NFS mount failed while the NAS network route was unavailable. Docker consequently left JDownloader and RomM stopped with Game-bind startup errors; qBittorrentVPN, Radarr, Sonarr, and Jellyfin also remained stopped after their NAS bind startup failures. After the NAS returned, targeted remounts restored Game and Filesync, the expected Game inbox/library and Game/Media marker paths were verified, and the six affected containers were started successfully.

This incident is preserved as dated diagnostic evidence. JAR-87 subsequently made the ROM and qBittorrent contracts Media-only, and JAR-88 removed the active `/mnt/nas-game` guest contract in favor of `/mnt/nas-gameserver`. Do not use the incident's former Game marker/bind wording as current reconstruction guidance.

### JAR-84 legacy service-home Docker-tree retirement — 2026-10-01

Fresh root Docker inspection found every active application Compose label under `/srv/rotom/stacks`, except the retained active NPM project under `/home/infra/docker/nginx-proxy-manager`. The inactive legacy `/home/downloaders/docker/{prowlarr,qbittorrentvpn}` and `/home/media/docker/{radarr,sonarr}` trees were archived, checksum-verified, covered by, and byte-read back from Restic snapshot `3c85d5be` before removal. `/home/game/docker` was verified empty and removed. At that time `/home/downloaders/docker` remained because Arcane read-only mounted its parent; the parent and bind were removed on 2026-10-13. `/home/media/docker/jellyfin` and held `/home/web/docker` were not changed. In particular, JAR-84 supersedes the prior JAR-49 current-location conclusion: active Jellyfin is `/srv/rotom/stacks/media/jellyfin/compose.yaml`, while its former service-home tree remains retained legacy material.

### JAR-82 Nginx Proxy Manager Infra v2 migration — 2026-10-01

NPM now runs from `/srv/rotom/stacks/infra/nginx-proxy-manager/compose.yaml` with authoritative persistent state at `/srv/rotom/appdata/infra/nginx-proxy-manager/{data,letsencrypt}`. It retains host networking, its established image/container identity, DNS, restart policy, listeners, proxy-host database, access-list configuration, TLS material, and compatibility upstreams. The former `/home/infra/docker/nginx-proxy-manager` Compose/data tree is retained rollback material only and was not modified after the final state copy.

### JAR-73 Uptime Kuma — 2026-09-30

`uptime-kuma` uses `louislam/uptime-kuma:2` from `/srv/rotom/stacks/infra/uptime-kuma/compose.yaml`. Its SQLite data is VM-local at `/srv/rotom/appdata/infra/uptime-kuma`; it joins `rotom-proxy` and publishes only `127.0.0.1:3002:3001`. It has no Docker socket, NAS mount, or broad host listener. NPM routes `uptime.rotom.casa` with TLS and WebSockets; the administrative UI is authenticated and no public status page is enabled.

### PVE web UI Homepage entry — 2026-09-30

NPM serves the PVE HTTPS UI at `https://pve.rotom.casa`, forwarding over HTTPS to `192.168.1.68:8006`; the route has forced TLS, WebSocket upgrade, and an enabled access list. Homepage's **Rotom Management** section includes a **PVE** link to that URL without a `siteMonitor`, so the dashboard does not create a separate monitor request to the privileged PVE UI. Local-SNI HTTPS returned the Proxmox login page; Homepage stayed healthy and returned local HTTP `200`. The route did not require a Compose, container, listener, DNS, certificate, PVE-service, or network-policy change.

Homepage's targeted `extra_hosts` mapping resolves `uptime.rotom.casa` to `192.168.1.69`, so its Uptime Kuma `siteMonitor` does not depend on container DNS availability.

**Historical JAR-78 state, superseded by JAR-88:** JAR-78 established `/srv/rotom/{stacks,appdata,secrets}/game`. JAR-88 later renamed those active roots to `gameserver`, preserved both VM-local worlds and UDP mappings, and left original Game material only as rollback evidence.

Docker socket access is exceptional: Arcane has the raw read-write socket and currently has automatic updates and auto-heal enabled; Homepage has the read-only socket for dashboard metadata; Glances has the read-only socket plus its read-only host-root monitoring bind. A read-only socket mount does not prove a read-only Docker API. All other current application containers have no socket. Current deliberate runtime exceptions are Home Assistant (host networking, privileged, read-only D-Bus), Homebridge/Nginx Proxy Manager/Cloudflare DDNS (host networking), qBittorrentVPN (`CAP_NET_ADMIN` for WireGuard), and the stated Arcane/Glances mounts. Phase D workload tickets must rejustify rather than inherit any exception.

### JAR-68 runtime snapshot — 2026-09-28

At JAR-68 acceptance there were **16 total container objects and 14 intended running containers**. Both Palworld containers were intentionally stopped by the administrator to save resources; they remained preserved with `unless-stopped`, were not dead/restarting/OOM, and their world save trees were reverified. The table below is that dated runtime snapshot, not a current inventory.

| Container | JAR-68 state / role |
|---|---|
| `arcane` | Running |
| `cloudflare-ddns` | Running |
| `homepage` | Running |
| `glances` | Running |
| `nginx-proxy-manager` | Running |
| `jellyfin` | Running |
| `radarr` | Running |
| `sonarr` | Running |
| `prowlarr` | Running |
| `qbittorrentvpn` | Running; `wg0` verified `10.2.0.2/32`; NAS sentinel/mount checks passed |
| `palworld-server-jared` | **Intentionally stopped**; world `DB40338954B844C28CEA21471A392F98` preserved |
| `palworld-server-fran` | **Intentionally stopped**; world `396F5898378F4E9CAE89461F403653D9` preserved |
| `gamarr` | Running |
| `home-assistant` | Running |
| `homebridge` | Running |
| `alohamillworks.com` | Running |

At JAR-68, `jaredwines.com` was intentionally undeployed and had no container object. Final post-reboot acceptance verified all twelve NAS automounts, qBittorrentVPN, Homepage PVE CPU monitoring, guest Restic timer state, and zero failed guest systemd units. Both former Downloads read-only bindfs compatibility views were retired later. The earlier 16-running observations below remain valid historical checkpoints; JAR-68 superseded them at that time with its 14-intended-running result.

**Post-JAR-68 changes:** JAR-75 subsequently deployed Syncthing; JAR-77 retired Gamarr and removed its container; JAR-84 found Syncthing exited before its preflight. The following supplied 2026-10-01 `docker ps -a` inventory supersedes the JAR-68 count and runtime-state claims.

### Historical Docker inventory — supplied 2026-10-01 output

The supplied root `docker ps -a` output recorded **25 container objects: 24 running and one exited**. `syncthing` was the sole exited object (`Exited (0)`); both Palworld containers were running and healthy; Gamarr was absent, consistent with JAR-77 retirement. The running set included `jdownloader`, `romm`, `nginx-proxy-manager`, `qbittorrentvpn`, both Palworld servers, `jellyfin`, `homepage`, `prowlarr`, `uptime-kuma`, `rotom-docs`, `romm-valkey`, `romm-db`, `paperless`, `paperless-db`, `paperless-broker`, `alohamillworks.com`, `homebridge`, `home-assistant`, `sonarr`, `radarr`, `arcane`, `cloudflare-ddns`, and `glances`.

At that snapshot, `jdownloader` was the private JAR-87 direct-download service. Its former Compose path, runtime identity, NAS bind, configuration, and secret boundary are historical only: the complete module was retired on 2026-10-05. Do not recreate it from this paragraph.

### JAR-38 Infra v2 convergence — 2026-09-28

The four migrated Infra modules each have an independent root-owned Compose definition: `/srv/rotom/stacks/infra/homepage/compose.yaml`, `glances/compose.yaml`, `arcane/compose.yaml`, and `cloudflare-ddns/compose.yaml`. These definitions were validated and recreated without an image pull; the verified cached images are Homepage, Glances, Arcane, and Cloudflare DDNS.

Homepage configuration is now `/srv/rotom/appdata/infra/homepage/config`; Glances configuration is `/srv/rotom/appdata/infra/glances/glances.conf`; Arcane data is `/srv/rotom/appdata/infra/arcane`, owned `997:5002`. Protected inputs are in `/srv/rotom/secrets/infra` and are Git-ignored. Homepage uses the supported `HOMEPAGE_VAR_*` substitution for its protected provider credential; its configuration contains no credential value. Homepage and Glances attach to `rotom-monitoring` and resolve `glances` through Docker DNS. Homepage additionally attaches `rotom-proxy`; Arcane attaches `rotom-proxy`. Glances exposes no host port. Homepage retains host `3001` and Arcane `3552` only as compatibility upstreams for the still-host-networked NPM; those paths returned local/proxied HTTP 200. NPM remains intact on 80/81/443 for JAR-39.

Cloudflare DDNS now uses its ordinary private `cloudflare-ddns_default` bridge, preserves its read-only token mount/capability drop/no-new-privileges contract, and immediately reported `rotom.casa` current. At this historical checkpoint Arcane retained its documented raw Docker socket, Infra-stack write bind, and read-only Downloader Compose compatibility bind; the latter was removed on 2026-10-13. `autoUpdate=false` and `autoHealEnabled=false` were reverified. To avoid expanding privileged secret access, Arcane has no v2 secret-directory bind; protected-env stack discovery warnings are intentional and do not authorize it to read secrets. The legacy `/home/infra/docker` module directories and `arcane_arcane-data` volume remain untouched rollback artifacts.

A controlled guest reboot recovered all four modules; Homepage/Arcane health, Homepage→Glances DNS/API, local and HTTPS Homepage, Arcane HTTPS, DDNS reconciliation, NPM compatibility, qBittorrent `wg0`, and zero failed systemd units passed. JAR-48 later adds baseline dashboard coverage using these existing components only; notification delivery is intentionally not configured.

## 2. Infrastructure — Preserved `infra` Domain Record

This section combines dated migration evidence with selectively updated service rows. It is not a current Platform reconstruction procedure; use section 1A and the current Platform overlays above for active paths and state.

### JAR-5 private Rotom documentation portal — 2026-09-29

`rotom-docs` runs the pinned Nginx static image from `/srv/rotom/stacks/infra/rotom-docs/compose.yaml`; its pinned `mkdocs.yml` lives beside that Compose definition. The portal binds only `127.0.0.1:8082` for retained host-networked NPM compatibility and joins `rotom-proxy`; it has no Docker socket, privileges, or NAS mount. Its root filesystem and `/srv/rotom/appdata/infra/rotom-docs/site` bind are read-only. A one-shot pinned MkDocs Material builder reads `/srv/rotom/appdata/infra/rotom-docs/source` read-only and writes the generated site only; it runs with all capabilities dropped and no network.

The root-owned `scripts/publish-source` is the controlled publication mechanism. It copies only the nine approved legacy documentation members `00`–`08` from the canonical checkout into a derived local source tree, produces the portal landing page, and wraps the canonical `08` text tree in rendered Markdown without modifying the canonical source. Run it deliberately followed by the build profile after future verified legacy documentation changes; the live portal is not a write path into the legacy documentation.

### JAR-40 Media v2 convergence — 2026-09-28

### JAR-10 RomM deployment — 2026-09-29

RomM 5.3.1 with MariaDB 11 and Valkey 9 runs from `/srv/rotom/stacks/library/romm/compose.yaml`. Its state, database, config, assets, resources, cache, and verified logical export are VM-local under `/srv/rotom/appdata/library/romm`; protected database/application inputs are root-only under `/srv/rotom/secrets/library/romm`. JAR-87 binds `/mnt/nas-media/library/games:/romm/library:ro` and retains scheduled quick scans at minutes 7, 22, 37, and 52.

Jellyfin, Radarr, Sonarr, and Prowlarr are active v2 modules under `/srv/rotom/stacks/library/<app>/compose.yaml`; JAR-84 directly verified all four Compose labels and active bind sources. Jellyfin retains its read-only `/mnt/nas-media/library` bind, while Radarr/Sonarr mount `/mnt/nas-media/torrents` at `/media/torrents`. qBittorrentVPN retains its Library stack/appdata paths and Media payload binds. The former service-home Radarr/Sonarr/Prowlarr/qBittorrentVPN trees are retired recovery artifacts; the Jellyfin tree is deliberately retained. All proxyable services attach `rotom-proxy`; Radarr/Sonarr/Prowlarr attach `rotom-arr`; qBittorrentVPN also attaches `rotom-arr`. Existing ports remain NPM compatibility upstreams.

The table below is the original service inventory with later path/status corrections applied selectively. Use the v2 convergence sections above—not the `/home` labels in older rows—as current reconstruction authority.

| Container name | Observed image | Dated status | Published ports | Container-only ports shown | Recorded Compose file |
| --- | --- | --- | --- | --- | --- |
| `arcane` | `ghcr.io/getarcaneapp/manager:latest` | Up 21 hours (healthy) | `0.0.0.0:3552->3552/tcp`, `[::]:3552->3552/tcp` | None shown | `/home/infra/docker/arcane/compose.yaml` |
| `cloudflare-ddns` | `favonia/cloudflare-ddns:1` | Up 3 days | None shown | None shown | `/home/infra/docker/cloudflare-ddns/compose.yaml` |
| `homepage` | `ghcr.io/gethomepage/homepage:latest` | Up (healthy), re-created 2026-09-18 | `0.0.0.0:3001->3000/tcp`, `[::]:3001->3000/tcp` | None shown | `/home/infra/docker/homepage/compose.yaml` |
| `glances` | `nicolargo/glances:latest-full` | Up, re-created 2026-09-18 | `127.0.0.1:61208->61208/tcp` | `61209/tcp` | `/home/infra/docker/homepage/compose.yaml` |
| `nginx-proxy-manager` | `jc21/nginx-proxy-manager:latest` | Running; JAR-82 recreated | None shown | None shown | `/srv/rotom/stacks/infra/nginx-proxy-manager/compose.yaml` |

### Homepage identity — September 16 update

From “Change Homepage URL,” Homepage's canonical URL is **`https://rotom.casa`**. The container remains `homepage`, with the same Compose path and host port `3001`. NPM proxy host 9 forwards the apex hostname to `http://192.168.1.69:3001`; supplied output confirms `server_name rotom.casa;` and a successful HTTPS response. `homepage.rotom.casa` is no longer a Homepage proxy hostname, even though DNS still resolves it through the observed wildcard behavior.

The adopted Homepage settings are `HOMEPAGE_ALLOWED_HOSTS=rotom.casa` in the existing Compose configuration and the following top-level settings in `/home/infra/docker/homepage/config/settings.yaml`:

```yaml
title: Rotom
description: Home automation and server management
```

The chat confirms the apex URL works and records the user's selected title and description. The final verification did not separately print these settings or the exact environment value; do not treat this documentation update as a fresh configuration inspection. See document 03 for DNS and HTTPS evidence.

### JAR-69/JAR-70 backup schedule cards — 2026-09-28/29

Homepage's top-level **Backup Schedule** section appears immediately above **Rotom Monitoring** and contains these custom-API cards in order: **Rotom VM Backup**, **Rotom Restic Backup**, and **PVE Restic Backup**. Their endpoints remain `http://192.168.1.69:8787/backup-status`, `http://192.168.1.68:8789/rotom-vm-backup-status`, and `http://192.168.1.68:8789/pve-restic-backup-status`, respectively. Each preserves `mdi-backup-restore`, a 60-second refresh, and mappings for Schedule, Last Result, Next Run, and Last Run. The presentation layer colors that mapped `Last Result` green for `success` and red for `failed`/`failure`; this does not change Custom API behavior or backup execution. Homepage's `settings.yaml` layout map lists **Backup Schedule** first with `style: row` and `columns: 3`, which renders the section at the top as one row of three equal cards. VZDump remains the technical name for the Rotom VM Backup mechanism. On 2026-10-14, the two stale Restic display strings in `/srv/rotom/appdata/platform/homepage/config/services.yaml` were corrected and Homepage was recreated successfully; endpoints and widgets were unchanged.


### Glances backup-mount removal — September 18 update

Glances no longer has any bind mount for `/mnt/nas-rotom-backup`. The existing `/home/infra/docker/homepage/compose.yaml` was inspected first; only the Glances volume line for the backup mount was removed. Compose validation passed. Because Compose attempted to reconcile the shared `homepage` network while Homepage was attached, the project was then cleanly recreated with `docker compose down` **without `-v`** followed by `docker compose up -d`. Homepage returned healthy, Glances returned running, the Homepage backend returned HTTP 200 with the intended `Host: rotom.casa` header, and `https://rotom.casa/` returned HTTP 200.

Verified current Glances mounts are:

```text
/ -> /mnt/host-root ro
/home/infra/docker/homepage/glances.conf -> /glances/conf/glances.conf ro
/var/run/docker.sock -> /var/run/docker.sock ro
```

The former `/mnt/nas-rotom-backup -> /mnt/nas-rotom-backup:ro` bind is intentionally absent. Loss of the Glances/Homepage NAS-backup storage metric is intentional; no replacement backup-capacity monitoring was added in this change. The live shared `homepage` bridge was reverified as `172.24.0.0/16`, with Homepage at `172.24.0.3` and Glances at `172.24.0.2` in the later full-network audit.

### Historical Prowlarr ownership/path migration — September 19 pre-JAR-21 state

At the September 19 pre-JAR-21 stage, Prowlarr was an active `infra` service at `/home/infra/docker/prowlarr/compose.yaml`, with the former `/home/media/docker/prowlarr` tree retained temporarily as rollback material. The stopped source was copied with metadata preservation and `diff -qr` showed no content differences before the target copy was re-owned. The active target Compose file preserves `lscr.io/linuxserver/prowlarr:latest`, `restart: unless-stopped`, `TZ=America/Los_Angeles`, `UMASK=002`, and host port `9696`, while using the JAR-6-final LinuxServer identity `PUID=997` / `PGID=5002` and the absolute `/config` bind to `/home/infra/docker/prowlarr/config:/config`.

Post-cutover Docker metadata verified `working_dir=/home/infra/docker/prowlarr`, the target config bind, a single running `prowlarr` container on `prowlarr_default`, and the application process running with numeric identity `997:5002` after the JAR-6 Infra storage-GID migration. Local and proxied requests both remained HTTP `302`, matching preflight. The targeted migration output observed runtime IPv4 `172.22.0.2` but did not capture the bridge subnet/gateway; the later full 2026-09-19 Docker-network audit resolved that limitation and verified `prowlarr_default` as `172.22.0.0/16` with gateway `172.22.0.1` and Prowlarr at `172.22.0.2`. Prowlarr UI acceptance confirmed the existing indexers, history, Sonarr and Radarr application entries, successful representative indexer and application tests, and no first-run/database-corruption condition. Arcane recognizes the single current Prowlarr project under the new path and showed auto-update enabled. Prowlarr has no `/mnt/nas-media` bind.

### Historical central qBittorrentVPN migration — September 19 pre-JAR-21 state

At the September 19 pre-JAR-21 stage, qBittorrentVPN was an `infra` service at `/home/infra/docker/qbittorrentvpn/compose.yaml`. The active config bind is `/home/infra/docker/qbittorrentvpn/config -> /config`; the config tree was reconciled to numeric `997:5000`. Runtime settings are `PUID=997`, `PGID=5000`, `UMASK=002`, port `8080`, ProtonVPN, and WireGuard. VPN credentials are referenced from a root-service-readable project `.env` file owned `infra:infra` mode `0600`; credential values are intentionally not documented. The old `/home/media/docker/qbittorrentvpn` rollback tree was deleted after acceptance, and the temporary `compose.yaml.pre-nas-guard` copy was removed after it was found to retain credential-variable lines.

The storage binds verified at that historical stage were:

```text
/home/infra/docker/qbittorrentvpn/config -> /config                 RW
/mnt/nas-media/torrents                   -> /media/torrents         RW
/mnt/nas-game/torrents                    -> /game/torrents          RW
/mnt/nas-media/.rotom-qbt-nas-ready       -> /run/rotom-nas-media   RO
/mnt/nas-game/.rotom-qbt-nas-ready        -> /run/rotom-nas-game    RO
/etc/localtime                             -> /etc/localtime          RO
```

The Media and Game torrent binds and both sentinel binds use long Compose bind syntax with `create_host_path: false`. Temporarily renaming either sentinel caused `docker compose up -d` to fail with a missing bind-source error and no qBittorrent container, proving the startup/recreation fail-closed behavior. Both sentinels restored cleanly and qBittorrent restarted. This is a startup/recreation guard, not a tested runtime watchdog for an already-running container after an NFS outage.

The qBittorrent categories are:

```text
Movies: /media/torrents/movies; incomplete /media/torrents/incomplete/movies
Shows:  /media/torrents/shows;  incomplete /media/torrents/incomplete/shows
Games:  /game/torrents/games;   incomplete /game/torrents/incomplete/games
```

Disposable post-UID-migration writes verified Media ownership `997:5000` and Game ownership `997:5001`. WireGuard remained active after the move. An earlier deliberate `wg0`-down test caused external egress to fail, confirming the VPN kill switch. The user intentionally cleared the old torrent/download queue during the `/data` → `/media` path migration, so preserving the historical queue is not part of the final as-built state.

## 2A. Downloads — Final Pre-Migration JAR-21 State

This entire subsection is historical. Current Prowlarr and qBittorrentVPN ownership, paths, and storage are defined by the Libraries overlays in section 1A.

JAR-21 moved the active download stack to the dedicated `downloads` account (`901:5005`). The old Infra project trees were retained during acceptance and then removed after checksum/active-path guards and a successful Restic backup.

| Container name | Image | Dated verification | Published ports | Historical Compose file |
| --- | --- | --- | --- | --- |
| `prowlarr` | `lscr.io/linuxserver/prowlarr:latest` | Active; HTTP 302; Downloads-owned | `9696:9696` | `/home/downloads/docker/prowlarr/compose.yaml` |
| `qbittorrentvpn` | `binhex/arch-qbittorrentvpn:latest` | Active; HTTP 200; WireGuard `wg0` present; Downloads-owned | `8080:8080` | `/home/downloads/docker/qbittorrentvpn/compose.yaml` |

Prowlarr uses Downloads identity `PUID=901` / `PGID=5005` and keeps its persistent `/config` beneath `/home/downloads/docker/prowlarr`. qBittorrentVPN runs as numeric `901:5005`; its persistent `/config` is `/home/downloads/docker/qbittorrentvpn/config`, and its project `.env` remains secret material that must never be printed or documented.

qBittorrent's active storage contract is one host tree, `/mnt/nas-downloads/torrents`, mounted read/write at both `/media/torrents` and `/game/torrents` to preserve existing category paths. Its startup guard is the read-only `/mnt/nas-downloads/.rotom-qbt-nas-ready -> /run/rotom-nas-downloads` bind with `create_host_path: false`. Default and category paths remain `/media/torrents`, `/media/torrents/incomplete`, Movies/Shows under `/media/torrents/...`, and Games under `/game/torrents/...`. VPN egress and the kill switch were reverified during JAR-21.

Radarr/Sonarr/Gamarr do not receive write access to the Downloads share. Persistent bindfs units expose `/mnt/nas-downloads` through identity-mapped read-only views, and Compose mounts only the torrent subtree from those views at the existing `/media/torrents` or `/game/torrents` container paths. Imports from Downloads to Media/Game are therefore cross-filesystem copies, not hardlinks.

Arcane remains owned by Infra. For project discovery it has `/home/downloads/docker:/home/downloads/docker:ro`, follows `/home/infra/docker/downloads -> /home/downloads/docker`, and has `followProjectSymlinks=true`. Final Arcane registry verification showed only the correct Prowlarr and qBittorrentVPN records at the symlinked Downloads paths; JAR-21 duplicate Infra/Rotom records are gone.

## 3. Media — Preserved `media` Domain Record

This section preserves the former Media-domain inventory and selectively appended later evidence. Current Libraries identity, Compose roots, and NAS contracts are authoritative in section 1A and documents 04 and 07.

The table below preserves the original Media inventory; its Compose-file column carries later verified v2 paths where available.

| Historical container name | Observed image | Dated status | Published ports | Container-only ports shown | Historical Compose file |
| --- | --- | --- | --- | --- | --- |
| `jellyfin` | `lscr.io/linuxserver/jellyfin:latest` | Running at JAR-84 verification | `0.0.0.0:8096->8096/tcp`, `[::]:8096->8096/tcp` | `8920/tcp` | `/srv/rotom/stacks/media/jellyfin/compose.yaml` |
| `radarr` | `lscr.io/linuxserver/radarr:latest` | Running at JAR-84 verification | `0.0.0.0:7878->7878/tcp`, `[::]:7878->7878/tcp` | None shown | `/srv/rotom/stacks/media/radarr/compose.yaml` |
| `sonarr` | `lscr.io/linuxserver/sonarr:latest` | Running at JAR-84 verification | `0.0.0.0:8989->8989/tcp`, `[::]:8989->8989/tcp` | None shown | `/srv/rotom/stacks/media/sonarr/compose.yaml` |

### Historical Media identity and NAS access

At this historical checkpoint, the Rotom `media` account was UID **127** and primary GID **5000** (formerly **129**). The then-current media-owned services—Jellyfin, Radarr, and Sonarr—used the established media identity pattern:

```text
PUID=127
PGID=5000
UMASK=002
```

Prowlarr and qBittorrentVPN were Downloads-owned services at `901:5005` after JAR-21. Radarr, Sonarr, and Jellyfin used numeric `127:5000`; Jellyfin additionally had GPU-related supplementary groups needed for `/dev/dri` access. The current Libraries identity and `/mnt/nas-libraries` contract are defined in the current overlays above.

`/mnt/nas-media` has root ownership `988:5000`, mode `2770`. Existing mixed owner UIDs in the library are preserved. The JAR-21 move to Downloads is historical; JAR-56/JAR-87 subsequently made `/mnt/nas-media/torrents` the active qBittorrent payload, written as UID:GID `901:5000`.

Sonarr and Radarr mount the complete Media NAS at `/media` and bind `/mnt/nas-media/torrents` read-only at `/media/torrents`; Jellyfin's library mount remains read-only. qBittorrentVPN binds `/mnt/nas-media/torrents` read/write. JAR-86 retired the unused `/mnt/nas-downloads-media-ro` bindfs view after confirming it had no container or Compose consumer. Only qBittorrent uses the VPN and its kill switch was directly verified. **JAR-49 current state:** PVE attaches the NUC's Iris Plus Graphics 655 as optional `hostpci0`; Rotom maps its Intel `01:00.0` GPU to `/dev/dri/card1` and `/dev/dri/renderD128`. Active Jellyfin maps `/dev/dri`, adds guest GIDs `44` (video) and `992` (render), and runs under its existing `127:5000` service identity. Jellyfin is deliberately configured for QSV at `/dev/dri/renderD128`, with H.264/HEVC hardware processing enabled. A forced live H.264 transcode initialized Intel `iHD`/VA-API and used `h264_qsv`, then exited `0`; this attachment, device mapping, and Jellyfin runtime survived a complete PVE reboot. QSV is optional: remove `hostpci0` and the Compose device/group mappings to return Jellyfin to software transcoding on portable hardware. Coffee Lake Gen9.5 supports H.264 plus HEVC 8/10-bit acceleration but not AV1; H.264 High 10 and HEVC RExt remain software-only limitations. Low-Power encoder and VPP tone-mapping remain disabled because HuC firmware was not commissioned.

Ordinary `jared` access to the media NAS is intentionally excluded. Administration uses `sudo -iu media` on Rotom after connecting as Jared. Jared's final account listing confirms he is not a member of media GID `5000`. UNAS's observed `--manage-gids` behavior means adding a client-side supplementary group alone does not grant the tested NFS access; using primary GID 5000 succeeded. See 04 and 07 for identity details and verification commands.

## 4. Game Servers — Current Summary and Preserved `gameserver` History

Palworld services use the locked `game-servers` identity at UID/GID `995:5001` and local state under `/srv/rotom/appdata/game-servers/`. Palworld stays on VM-local storage. Both current Compose modules are under `/srv/rotom/stacks/game-servers`, and their UDP mappings are unchanged. Jared and Fran were running and healthy on 2026-10-15; the reason and intended persistence of the change from the 2026-10-14 stopped state remain unverified. Gamarr is retired; original Game material is rollback-only.

The following table is the dated JAR-68/pre-retirement snapshot. Current JAR-88 paths and health are stated above.

| Historical container name | Observed image | Dated status | Published ports | Container-only ports shown | Historical Compose file |
| --- | --- | --- | --- | --- | --- |
| `palworld-server-fran` | `thijsvanloef/palworld-server-docker:latest` | **Intentionally stopped** at final JAR-68 acceptance; later healthy after JAR-88 | `0.0.0.0:8212->8212/udp`, `[::]:8212->8212/udp`, `0.0.0.0:27016->27016/udp`, `[::]:27016->27016/udp` | `25575/tcp` | `/home/game/docker/palworld-server-fran/compose.yaml` (retired legacy path) |
| `palworld-server-jared` | `thijsvanloef/palworld-server-docker:latest` | **Intentionally stopped** at final JAR-68 acceptance; later healthy after JAR-88 | `0.0.0.0:8211->8211/udp`, `[::]:8211->8211/udp`, `0.0.0.0:27015->27015/udp`, `[::]:27015->27015/udp` | `25575/tcp` | `/home/game/docker/palworld-server-jared/compose.yaml` (retired legacy path) |
| `gamarr` | `ghcr.io/gamarr-app/gamarr:latest` | Historical 2026-09-19 running/healthy state; retired by JAR-77 | `0.0.0.0:6767->6767/tcp`, `[::]:6767->6767/tcp` | None shown | `/home/game/docker/gamarr/compose.yaml` (retired) |

### Palworld account/home migration — 2026-09-18

At completion of the 2026-09-18 account/home migration, the Linux service user and group were both named `game`, preserving UID `995` and then-current GID `985`; the canonical home became `/home/game`. Both Compose files use the relative bind `.:/palworld`, so no Compose volume-line rewrite was required when the project directories moved. After recreation, Docker metadata verified these live host binds and Compose locations:

```text
palworld-server-jared: /home/game/docker/palworld-server-jared -> /palworld (rw)
  project=palworld-jared
  working_dir=/home/game/docker/palworld-server-jared
  config_files=/home/game/docker/palworld-server-jared/compose.yaml

palworld-server-fran: /home/game/docker/palworld-server-fran -> /palworld (rw)
  project=palworld-fran
  working_dir=/home/game/docker/palworld-server-fran
  config_files=/home/game/docker/palworld-server-fran/compose.yaml
```

Host process output showed both servers running under account `game` with numeric UID/GID `995:985`, and both Docker health states returned `healthy`. Before restart, SHA-256 manifests of all `.sav` files compared byte-for-byte unchanged across the home move: 140 files for Jared and 112 for Fran. Current world identifiers are `DB40338954B844C28CEA21471A392F98` (Jared) and `396F5898378F4E9CAE89461F403653D9` (Fran). Never recreate these stacks against empty replacement directories or initialize new worlds over the existing trees.

### Game GID migration — 2026-09-19

The separate game-storage identity migration changed the `game` primary group from GID `985` to `5001` while preserving UID `995` and `/home/game`. Before the change, 10,732 objects under `/home/game` used GID `985`; the targeted migration left zero GID-985 objects under `/home/game`. Containerd overlay objects outside `/home/game` were deliberately not modified.

Both Palworld Compose files now explicitly use:

```text
PUID=995
PGID=5001
```

Compose validation succeeded and both existing containers were recreated against their unchanged persistent directories. Live inspection showed `PUID=995`, `PGID=5001`; application processes resolved as `game:game`; both containers returned healthy. A stopped-state SHA-256 manifest covering 234 `.sav` files compared byte-for-byte identical after recreation. The Jared Palworld `.env` still contains stale `PUID=1003` / `PGID=1003` lines, but the current Compose file uses literal `995:5001`, so those `.env` values are not the deployed runtime identity.

### Historical Gamarr deployment — September 19 update

Gamarr is deployed at `/home/game/docker/gamarr` with image `ghcr.io/gamarr-app/gamarr:latest`, `PUID=995`, `PGID=5001`, `TZ=America/Los_Angeles`, port `6767`, local config at `/home/game/docker/gamarr/config:/config`, and `/mnt/nas-game:/game`. The application reported version `1.0.3.0` during commissioning. A disposable library write created host ownership `995:5001`.

This historical state was superseded by JAR-40 and JAR-52: Gamarr now uses `/srv/rotom/stacks/media/gamarr/compose.yaml`, appdata at `/srv/rotom/appdata/media/gamarr/config`, `PUID=995` / `PGID=5000`, and only `/mnt/nas-media/library/games:/game/library/games` for its final library. It retains the read-only Downloader Game view at `/game/torrents`; `/mnt/nas-game/library/games` stays intact as rollback material.

The intended Game library roots are `/game/library/games/pc` for PC releases and `/game/library/games/roms` when ROM content is intentionally used. The qBittorrent download client uses the `Games` category and host `192.168.1.69:8080`. Gamarr and qBittorrent share the same `/game` path prefix for completed Game torrents, so no remote path mapping is required.

Prowlarr Torznab integration uses base URL `http://192.168.1.69:9696/2`; Gamarr appends `/api` itself. Supplying a URL that already ends in `/api` produced `/api/api`, an HTML login redirect, and XML parsing failure. API keys are secrets and are intentionally omitted. The corrected base successfully returned search results and a release was sent through Prowlarr to qBittorrent. The final commissioning test was user-confirmed for download, import into the intended PC library, hardlink behavior, and continued seeding. A later live audit verified the container as `running` / `healthy`. Docker inspection shows the healthcheck comes from the image (`/etc/s6-overlay/s6-rc.d/svc-gamarr/data/check`, 30-second interval, 10-second timeout, 120-second start period, 3 retries); `/home/game/docker/gamarr/compose.yaml` has no explicit `healthcheck:` directive. Gamarr now also has verified named HTTPS access through Nginx Proxy Manager at `gamarr.rotom.casa -> 192.168.1.69:6767`; the NPM record uses certificate ID `24`, forces SSL, has no access list, and returned HTTP `302` in the route probe.

## 4A. Documents — Preserved Deployment History

The Documents-domain paths in this section are historical. Current Paperless deployment and media ownership are defined by the Libraries overlays in section 1A and documents 04, 05, and 07.

### JAR-8 Paperless-ngx deployment — 2026-09-29

Paperless-ngx `v3.2.1`, PostgreSQL `18`, and Valkey `9` now run from `/srv/rotom/stacks/library/paperless/compose.yaml`. JAR-92 maps its application processes to Library `127:5000`; mutable Paperless data, exports, consume, database, and broker state are VM-local below `/srv/rotom/appdata/library/paperless`, while authoritative media remains at the Documents NAS boundary. Credentials remain only under `/srv/rotom/secrets/library/paperless`. The web service has loopback-only `8000` access and joins `rotom-proxy`; database/broker have no host ports. NPM route `paperless.rotom.casa -> 127.0.0.1:8000` uses the existing wildcard certificate.

A harmless local PDF import completed OCR, title search, original retrieval, and survived web-service recreation. Its supported export (manifest, metadata, PDF, thumbnail) is protected local backup staging. Generic guest Restic includes `/srv`, but no verified snapshot includes this deployment. Rotom's public-IP hairpin TLS cannot test the route locally, but an independent public TLS assessment verified the expected login redirect and HTTP `200` login page through `paperless.rotom.casa`.

### Historical JAR-91 Paperless media NAS cutover — 2026-10-05 (superseded by JAR-92)

At the JAR-91 checkpoint, Paperless media became authoritative at `/mnt/nas-documents/paperless`, mounted in the web service at `/usr/src/paperless/media`. The Compose definition used long bind syntax with `create_host_path: false`, so an absent Documents NFS mount could not become an empty local media directory. Its then-current `USERMAP_UID=900`/`USERMAP_GID=5004` and `900:5004` ownership are superseded by the current Libraries `127:5000` deployment; the fail-closed behavior remains current at `/mnt/nas-libraries/paperless`.

At this historical checkpoint, PostgreSQL, Valkey, Paperless data/configuration/search state, consume, exports, and secrets remained VM-local at their Documents paths. NPM routing was unchanged. The five-file, 384,771-byte media transfer matched SHA-256 manifests before and after cutover; Paperless recreated healthy and the local/public routes returned the expected login redirect. The then-retained rollback path `/srv/rotom/backup-staging/documents/jar91-20261005-220500/` is no longer local because `/srv/rotom/backup-staging` was absent on 2026-10-14.

The authenticated existing-original download, controlled web upload/OCR/search/thumbnail check, NAS automount/reconnect behavior, and isolated NAS snapshot restore are still required acceptance checks. Do not remove the local rollback material or the original local media tree until those checks are accepted.

## 5. Smart Home — Preserved `smarthome` Domain Record

This section preserves the earlier singular-domain paths and dated verification. Current `smart-home` Compose/state paths and permissions are defined in section 1A and document 07.

### JAR-43 Smart Home v2 convergence — 2026-09-29

Home Assistant and Homebridge now use independent root-administered Compose modules at `/srv/rotom/stacks/smarthome/home-assistant/compose.yaml` and `/srv/rotom/stacks/smarthome/homebridge/compose.yaml`. Their authoritative mutable state is explicitly bound from `/srv/rotom/appdata/smarthome/home-assistant:/config` and `/srv/rotom/appdata/smarthome/homebridge:/homebridge`, respectively. The state was copied with preserved numeric metadata while each service was stopped. JAR-51 later retired the inactive legacy `/home/smarthome/docker/{home-assistant,homebridge}` rollback trees after no active configuration reference was found and the current Compose/services were verified.

Both services retain `network_mode: host` and `restart: unless-stopped`; this preserves Home Assistant discovery and Homebridge/Avahi mDNS behavior. No Docker bridge/network, port, NPM proxy-host, certificate, NAS mount, service UID/GID, or backup-policy change was made. Home Assistant's `/etc/localtime` and `/run/dbus` read-only binds remain intact. Homebridge retains its explicit JSON-file limit of 10 MB with one retained file.

Post-reboot verification confirmed both containers running from the v2 mounts; local `8123`/`8581` and `https://home-assistant.rotom.casa` / `https://homebridge.rotom.casa` each returned HTTP 200. Both Compose modules parsed, Home Assistant and Homebridge SQLite `quick_check` results were `ok`, mDNS/SSDP listeners were present, the guest Restic timer was active, and zero failed systemd units were reported. The existing guest Restic `/srv` source scope covers the v2 state; no backup or restore was run in JAR-43.

| Current container name | Observed image | Observed status | Published ports | Container-only ports shown | Compose file |
| --- | --- | --- | --- | --- | --- |
| `home-assistant` | `ghcr.io/home-assistant/home-assistant:stable` | Running; JAR-43 post-reboot verification passed | None shown (host network) | None shown | `/srv/rotom/stacks/smarthome/home-assistant/compose.yaml` |
| `homebridge` | `homebridge/homebridge:latest` | Running; JAR-43 post-reboot verification passed | None shown (host network) | None shown | `/srv/rotom/stacks/smarthome/homebridge/compose.yaml` |

The Homebridge container name was corrected on 2026-09-19 from the historical misspelling `homebrige` to `homebridge`. The Compose directory and image name were already correct. Homebridge started successfully after the change and restored its cached accessories; the whole-home scan reported no remaining `homebrige` references.

## 6. Web Hosting — Preserved `web` Domain Record

This section preserves the earlier `web` paths and dated verification. Current `hosted-websites` identity and Compose/state roots are defined in section 1A and document 07.

### JAR-45 Web v2 convergence — 2026-09-29

Aloha Millworks now uses `/srv/rotom/stacks/web/alohamillworks.com/compose.yaml` and a read-only `/srv/rotom/appdata/web/alohamillworks.com:/var/www/html` bind. It remains on its private default bridge plus external `rotom-proxy`, with `unless-stopped`. Retained NPM is host-networked and proxies through the LAN compatibility listener, so the existing `7778:80` mapping remains necessary; no NPM, DNS, TLS, certificate, or network redesign occurred. Local and public HTTPS probes returned HTTP 200 and NPM configuration validated.

Jared Wines has v2 content at `/srv/rotom/appdata/web/jaredwines.com` and an unstarted Compose module at `/srv/rotom/stacks/web/jaredwines.com`; it has no container object or published listener. On 2026-09-30, Homepage's **Hosted Websites** card was restored with its existing public URL and `siteMonitor`; the target returned HTTP `502`, consistent with the intentional undeployed state. Source-to-v2 comparisons found no differences before Jared explicitly retired both `/home/web/docker` legacy project trees on 2026-09-29. Guest Restic already covers `/srv`; no backup or restore was run.

| Service / current container name | Observed image | Observed status | Published ports | Container-only ports shown | Compose file |
| --- | --- | --- | --- | --- | --- |
| `alohamillworks.com` | `php:8.4-apache` | Running; JAR-45 local/public verification passed | `0.0.0.0:7778->80/tcp`, `[::]:7778->80/tcp` | None shown | `/srv/rotom/stacks/web/alohamillworks.com/compose.yaml` |
| `jaredwines.com` | `nginx:alpine` | Intentionally unstarted; no container object | No active mapping | None shown | `/srv/rotom/stacks/web/jaredwines.com/compose.yaml` |

JAR-19 verified `web` remains UID/GID `902:902` with Docker membership. Aloha Compose metadata points to `/home/web/docker/alohamillworks.com`; both local port `7778` and `https://alohamillworks.com` returned HTTP 200 after cutover. The old `web-host 128:130` user/group/home were removed only after dependency and ownership guards passed. Rollback evidence remains at `/root/jar-19-20260921-024634`; final Restic snapshot `e886c55d` captures the accepted state.

## 6A. Infrastructure / Smarthome Identity-Path Migration — 2026-09-21

The earlier name/home migration renamed `rotom` to `infra` and `smart-home` to `smarthome` while preserving UIDs and then-current primary GIDs, moving the homes to `/home/infra` and `/home/smarthome`. Docker supplementary membership and the documentation ACL model were preserved. That rename-stage GID state is historical; JAR-6 later changed the primary service GIDs as recorded below.

## 6B. JAR-6 Infra/Service Storage-GID Migration — 2026-09-21

JAR-6 changed the eight service primary GIDs while preserving UIDs: Infra `997:5002`, Smarthome `126:5003`, Documents `900:5004`, Downloads `901:5005`, Web `902:5006`, Filesync `903:5007`, Apps `904:5008`, and Auth `905:5009`. Matching ownership beneath each service home was reconciled; no old primary-GID objects remain in those eight home trees. Arcane's live named-volume data was also migrated from old Infra GID `986` to `5002`. Historical backup material and containerd snapshot-layer objects retaining old numeric GIDs were intentionally left unchanged.

Docker changes were identity-only for the affected Infra services: Arcane now uses `PGID=5002`; Cloudflare DDNS uses container user `997:5002`; Prowlarr uses `PUID=997` / `PGID=5002`. Those three services were recreated and verified running. qBittorrentVPN was deliberately not recreated for the GID migration, remains `PUID=997` / `PGID=5000`, and its Compose file later compared byte-for-byte with the pre-migration backup. Its Media/Game torrent binds, read-only NAS sentinel binds, `create_host_path: false` protection, VPN/WireGuard settings, and kill-switch configuration were not changed.

JAR-6 added **no Docker integration** for the eight new service NAS shares. No running container or Compose file references `/mnt/nas-infra`, `/mnt/nas-smarthome`, `/mnt/nas-documents`, `/mnt/nas-downloads`, `/mnt/nas-web`, `/mnt/nas-filesync`, `/mnt/nas-apps`, or `/mnt/nas-auth`. Home Assistant/Homebridge, Palworld, and website content remain on local ext4-backed `/home` paths.

## 7. Shared Docker Architecture and Storage Notes

- Current active application Compose definitions are predominantly under `/srv/rotom/stacks/<domain>/<service>/compose.yaml`; NPM is also v2. Retained `/home/<service>/docker` trees are compatibility or rollback material unless a current service entry explicitly identifies one as active. Documents (Paperless) and Filesync (Syncthing) have deployed workloads; Auth and Customapps are retired.
- JAR-22 records 16 running containers and 16 total container objects. `jaredwines.com` has a retained Compose project at `/home/web/docker/jaredwines.com/compose.yaml` but no current container object. `glances` shares the Homepage Compose stack at `/home/infra/docker/homepage/compose.yaml`.
- Published mappings include both IPv4 and IPv6 bindings for most mapped services. Glances shows an IPv4 loopback binding only. These listings do not establish firewall rules, router forwarding, or external reachability.
- The original container/path inventories alone do not establish volumes, networks, environment variables, dependencies, VPN behavior, or reverse-proxy routing. Later evidence adds the media identity settings and mounts above; use 03 and 04 for separately audited network and storage details. Full current Compose contents still need inspection before editing any service.

### Historical service groups and NAS mounts — through JAR-88

JAR-94 above supersedes the `infra`, `documents`, `downloader`, and `filesync` rows and the retired NAS contracts below; those rows remain dated identity/storage evidence.

This dated table preserves the then-current guest identity/storage relationships; it is superseded by the JAR-96 and current-identity overlays above.

| Service account | Current guest UID:GID | Current guest NAS mount | Current Docker/application use |
|---|---:|---|---|
| media | `127:5000` | `/mnt/nas-media` | Jellyfin/Radarr/Sonarr; active torrent path is `/mnt/nas-media/torrents` |
| gameserver | `995:5001` | `/mnt/nas-gameserver` | Both Palworld servers; Palworld is VM-local. JDownloader, Igir, qBittorrent importer, and browser-upload bridge were retired on 2026-10-05, so their former `995:5000` Media ROM-inbox contract is historical only. |
| infra | `997:5002` | `/mnt/nas-infra` | Active modules/state are under `/srv/rotom`; retained `/home/infra` material is compatibility/rollback only where documented |
| smarthome | `126:5003` | `/mnt/nas-smarthome` | Home Assistant/Homebridge definitions and state are under `/srv/rotom` |
| documents | `900:5004` | `/mnt/nas-documents` | Paperless application state is VM-local; authoritative media is `/mnt/nas-documents/paperless` with a fail-closed bind |
| downloader | `901:5005` | `/mnt/nas-downloaders` | Active Prowlarr/qBittorrentVPN owner; compatibility home `/home/downloaders`; current NFS source `Downloader/.data`; qBittorrent `wg0` verified |
| web | `902:5006` | `/mnt/nas-web` | Aloha active; Jared Wines retained but intentionally undeployed |
| filesync | `903:5007` | `/mnt/nas-filesync` | Syncthing deployed; no production folder/payload configured |

Current application definitions use the per-domain `/srv/rotom/stacks` modules described above. JAR-86 retired the unused `/mnt/nas-downloads-media-ro` compatibility view and its recovery-helper branch; `/mnt/nas-downloaders` and its sentinel remain as the retained Downloader boundary, while qBittorrent's payload and guard are Media-only. JAR-88 replaced the active Game guest mount with Gameserver.

The current `docker` group should be re-read before future privilege changes. The 2026-10-14 explicit verification supersedes the JAR-30/JAR-31 empty-group checkpoint: GID `989` now lists only human administrator Fran. No service identity was added.

## 8. Historical Consolidated Verification — 2026-09-19

A fresh `docker ps -a`, network audit, and Compose-label reconciliation verified all 17 then-documented service/container records and all 16 active Compose paths. A later comprehensive same-day audit reconfirmed this state and additionally showed Gamarr healthy through its image-provided healthcheck while qBittorrentVPN had no Docker healthcheck. **At that 2026-09-19 historical checkpoint**, the bridge networks were: Radarr `172.18.0.0/16`, qBittorrent `172.19.0.0/16`, Jellyfin `172.20.0.0/16`, Sonarr `172.21.0.0/16`, Prowlarr `172.22.0.0/16`, Arcane `172.23.0.0/16`, Homepage/Glances `172.24.0.0/16`, Jared Wines `172.25.0.0/16`, Aloha Millworks `172.26.0.0/16`, Jared Palworld `172.27.0.0/16`, Fran Palworld `172.28.0.0/16`, and Gamarr `172.29.0.0/16`. Host-network services remain Home Assistant, Homebridge, NPM, and Cloudflare DDNS.

`jaredwines.com` now has direct JAR-19 evidence at `/home/web/docker/jaredwines.com/compose.yaml` with `web:web` ownership and remains stopped by explicit migration intent. Aloha is active from `/home/web/docker/alohamillworks.com`; Compose metadata points to the new path and local/proxied HTTP checks returned 200.

qBittorrent is running from `/home/downloads/docker/qbittorrentvpn` as numeric `901:5005`, with active WireGuard `wg0`, `/mnt/nas-downloads/torrents` bound read/write at both `/media/torrents` and `/game/torrents`, and the read-only Downloads sentinel. JAR-21 verified VPN egress, kill-switch behavior, local/proxied HTTP access, and fail-closed container recreation when the Downloads sentinel was absent.


## 8A. Historical JAR-9 Rollback Verification — 2026-09-21 (superseded for qBittorrent storage by JAR-21)

At JAR-9 rollback closeout, the pre-JAR-9 Docker contracts documented above had been restored. JAR-21 later superseded the qBittorrent storage portion of this historical state. Both Palworld containers were recreated from `/home/game/docker/...` and returned healthy with their original worlds. Gamarr is again `/home/game/docker/gamarr` with `/mnt/nas-game -> /game`. qBittorrentVPN again has `/mnt/nas-game/torrents -> /game/torrents` plus the Game sentinel alongside its Media mounts, and its Games category uses `/game/torrents/games` and `/game/torrents/incomplete/games`. The JAR-9 Gamarr state is retained only as rollback material under `/home/media/migration-rollback/JAR-9/gamarr-j9-state`.

## 8B. Historical JAR-6 Docker Regression Verification — 2026-09-21 (superseded for Prowlarr/qBittorrent by JAR-21)

At the JAR-6 regression checkpoint, Arcane, Cloudflare DDNS, Prowlarr, and qBittorrentVPN were verified running. At that pre-JAR-21 point, qBittorrent still used Media/Game torrent binds and sentinels. Its Compose file remained byte-for-byte unchanged from the pre-migration backup. No container or Compose file uses any of the eight new JAR-6 NAS mountpoints.

## 8C. JAR-22 Final Pre-Migration Docker Verification — 2026-09-22

JAR-22 supersedes the 2026-09-19 **current container-object count** while preserving the earlier audit as historical evidence. A fresh `docker ps -a` and Compose-label capture shows **16 running containers and 16 total container objects**. Active Compose working directories remain aligned with service ownership: Infra (`arcane`, `cloudflare-ddns`, Homepage/Glances, NPM), Downloads (`prowlarr`, `qbittorrentvpn`), Media (`jellyfin`, `radarr`, `sonarr`), Game (`gamarr` and both Palworld servers), Smarthome (Home Assistant/Homebridge), and Web (Aloha Millworks). The only named Docker volume remains `arcane_arcane-data`.

Jared Wines is intentionally not active, but its present-day Docker state is more minimal than the JAR-19 closeout state: `/home/web/docker/jaredwines.com/compose.yaml` exists as `web:web`, parses the `website` service, and `docker compose ps -a` returns no container. `docker ps -a` likewise returns no `jaredwines.com` object. The historical `jaredwinescom_default` network is also absent from the current Docker network list. Do not recreate the container or network solely to make historical inventory wording true.

JAR-22 also verified zero failed systemd units. No Compose file, container, Docker network, volume, or service state was changed by the audit.


## 8D. JAR-23 Critical-Application Preservation Verification — 2026-09-25

JAR-23 revalidated every current stateful Compose project using the owning service identity. All expected Compose files parsed successfully, all 16 expected deployed containers were running at final acceptance, and zero failed systemd units remained. Jared Wines is intentionally undeployed: `/home/web/docker/jaredwines.com/compose.yaml` validates, but no `jaredwines.com` container object exists. Aloha Millworks remained running.

Palworld preservation stopped each server one at a time only after confirming the exact populated world directory. Jared world `DB40338954B844C28CEA21471A392F98` produced a stopped-state manifest with 135 `.sav` checksum entries; Fran world `396F5898378F4E9CAE89461F403653D9` produced 108. Both existing containers returned `running` / `healthy` after preservation; no stack was recreated against an empty directory.

Home Assistant and NPM remained running while the established SQLite online-backup method refreshed `/var/backups/system-info/sqlite/home-assistant_v2.db`, `zigbee.db`, and `npm-database.sqlite`; all three staging files passed `PRAGMA quick_check`. NPM's `data` and `letsencrypt` binds remain the recovery-critical paths, with 44 certificate files observed. Stateful persistence was revalidated for Arcane, Cloudflare DDNS, Homepage/Glances, NPM, qBittorrentVPN, Prowlarr, Gamarr, both Palworld servers, Jellyfin, Radarr, Sonarr, Home Assistant, Homebridge, and Aloha Millworks. Arcane's only named volume remains `arcane_arcane-data`.

During the same ticket, existing Radarr, Sonarr, and Gamarr container objects were recovered after a boot-time NAS mount race; they were started in place and returned HTTP 302, with Gamarr healthy. The host-level `rotom-nas-docker-recovery.service` is now enabled to retry only NAS-backed startup failures after Docker; it does not alter Compose restart policies or globally gate Docker on NAS availability. Full reboot validation of that helper remains deferred.

## 8E. JAR-24 Final Recovery-Point Docker Verification — 2026-09-25

JAR-24 made no Compose, image, network, port, restart-policy, bind-mount, or deployment-state changes. The final host-level backup verification ended with the same 16 expected deployed containers running; Arcane, Homepage, Gamarr, and both Palworld servers reported healthy where Docker healthchecks exist. Jared Wines remained intentionally undeployed with its Compose project retained and no container object. Zero failed systemd units were present at final acceptance.

Final Restic snapshot `fbe1838e` (`fbe1838e769cb740294ec0bf1017088a47ca5b97560083382af7cfcbebb2897f`) at `2026-09-25 18:14:22 PDT` captures the established host-side Docker configuration roots and named-volume data through `/home`, `/etc`, `/usr/local`, `/var/backups/system-info`, and `/var/lib/docker/volumes`. JAR-24 directly verified both current Palworld `Level.sav` files, the JAR-23 preservation audit, and the NAS recovery helper/unit inside that snapshot. NAS-resident bind sources under `/mnt` remain outside host Restic and must not be mistaken for data copied into the final snapshot.


## 8F. JAR-25 Live Bare-Metal Image Runtime Verification — 2026-09-25

JAR-25 changed no Compose file, image, network, port, bind mount, named volume, restart policy, or intended deployment state. Before whole-disk acquisition, the workflow captured the exact set of 16 running containers, paused selected active maintenance scheduling, stopped Arcane first, then stopped the other deployed workloads, and flushed filesystem writes. Jared Wines remained intentionally undeployed and was not added to the running set.

The raw source `/dev/nvme0n1` was read and streamed directly to the UNAS Shared Drive. After the first image write completed, the workflow restarted the exact pre-image container set before performing the long stored-image reread/checksum phase. Final acceptance output reported the original 16-container runtime restored and zero failed systemd units. No container was recreated against a new or empty data path.

This runtime quiesce improves application consistency but does not make the image equivalent to an offline image: the host OS and mounted ext4 root remained active during acquisition. The resulting JAR-25 image is therefore recorded as best-effort live/crash-consistent.

## 9. Evidence and Interpretation Caveats

- **Blank port fields:** Blank `docker ps` port fields do not establish networking mode. The later audit in 03 independently verified host networking for `homebridge`, `home-assistant`, `cloudflare-ddns`, and `nginx-proxy-manager`; this is no longer an open verification item. Continue to inspect current configuration before editing a service rather than inferring network mode or customary ports from a blank field.

## 10. Historical Verification — 2026-09-15

Read-only Docker checks on 2026-09-15 confirmed 15 running containers and one Created container, `jaredwines.com`. That count/status is historical and was superseded by later audits. Other still-valid 2026-09-15 observations remain historical evidence only; use **Current Docker Runtime and Deployment State** in section 1A for the current runtime.

## 11. Reconstruction-Critical Information

For current reconstruction, start with **Current Docker Runtime and Deployment State** (section 1A) and the current guest service-account table in section 7. The dated pre-migration sections remain recovery evidence and must not be replayed blindly where they use historical `downloads`, old network allocations, or pre-VM status text. Preserve each active service's current owning account, Compose path, image, container name, published ports/network mode, persistent binds/volumes, restart behavior, and non-secret environment-variable names.

UID/GID account definitions are canonical in `07-Users-and-Permissions.md`; NAS export/mount semantics are canonical in `04-NAS-and-Storage.md`; full backup policy is canonical in `05-Backup-and-Restore.md`.

## 12. Proposed Services

No additional Docker service is promoted to current state by this restructure. Any future service documented elsewhere remains Proposed until deployment evidence is recorded here and in the change log.

## 13. Outstanding / Needs Verification

The VM-era backup baseline is no longer a Docker gap. The latest observed automatic Rotom Restic Backup snapshot is `1fd623ca` from 2026-10-15 03:00 PDT, approximately 20.282 GiB; the backup service completed successfully. Earlier points including `553e7a39`, `9494979d`, `8efea294`, and `f666d63c` remain dated evidence. Jared confirmed a current full Rotom Restic Backup data-read and restore rehearsal; the selected snapshot and transcript were not retained. `/mnt` remains outside the Rotom Restic Backup source set.

- **Intentional:** Fran is recreated with full sudo on Rotom and PVE. Her historical exact-command backup sudoers rule remains absent because full sudo already authorizes the workflow; SSH-key, legacy documentation/GitHub/Codex, and named documentation access remain deferred.
- **Verified by Jared confirmation — live Libraries-NFS loss:** the planned non-disruptive running-qBittorrent test completed. Detailed runtime observations were not retained; do not infer behavior beyond the confirmed test outcome.
- **Known Arcane automation drift:** live 2026-10-15 Arcane settings have automatic updates and auto-heal enabled, scheduled pruning disabled, and `followProjectSymlinks=true`. A successful non-dry-run automatic update recreated Jared Wines at 10:40 PDT. This supersedes the earlier same-day database reading that reported updates and auto-heal disabled. JAR-85 verified an earlier active Arcane container mount of `/home/downloaders/docker`; the former `/home/infra/docker/downloads` discovery symlink was unreferenced by the current Arcane database and retired. The Downloader bind and its empty source were removed on 2026-10-13. Historical Arcane records that name the old symlink remain historical evidence. Its persisted qBittorrentVPN project status may be stale/`unknown`; Docker runtime and WireGuard, not Arcane metadata, are runtime authority.

## 14. Related Documentation

- `03-Network-and-Domains.md` — canonical Docker network, listener, port, domain, and proxy routing details.
- `04-NAS-and-Storage.md` — canonical NFS/mount/storage contracts and hardlink behavior.
- `07-Users-and-Permissions.md` — canonical Linux account, UID/GID, group, and privilege model.
- `06-Maintenance-and-Automation.md` — update/restart automation and recurring operational behavior.

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate this document's procedures and current-state reference after any container name, desired state, Compose path, image, port, volume, network, dependency, persistent-data boundary, or health-check change. Also update document 01 if a task destination or procedure heading changes and document 03 when a listener or external route is affected.
