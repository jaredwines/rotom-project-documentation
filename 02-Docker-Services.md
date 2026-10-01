# 02 - Rotom Docker Services Inventory

**Documentation set:** Rotom Project Documentation  
**Document role:** Canonical source for Docker/Compose service inventory and deployment details  
**Scope:** Rotom workload layer; current supplied 2026-10-01 Docker inventory is 25 container objects / 24 running, with deployment details separately evidenced per service
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-01 — boot-time Game NAS recovery limitation verified
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `03-Network-and-Domains.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## 1. Purpose and Scope

This inventory is based on the uploaded `rotom-docker-containers.txt` (container names, images, statuses, and ports) and `rotom-compose-locations.txt` (Compose file paths), with verified Docker inspection and status updates from the conversation incorporated on 2026-09-14. The original table statuses are recorded observations rather than continuously live values; later dated verification sections incorporate fresh Docker state and health checks where they were explicitly captured. The original source files do not specify a capture timestamp.

### Evidence provenance

Last updated: 2026-09-28 (JAR-34 declarative filesystem foundation added; JAR-31 current VM application restoration/reboot acceptance remains the Docker runtime baseline)
Project: **Rotom-Home-Server**

The 2026-09-16 identity update below uses supplied Rotom/UNAS command output from the historical “Torrent Seeding Explanation” conversation. The 2026-09-18 backup-access work additionally verified the live Homepage/Glances Compose project, removed Glances' backup-share bind, recreated that two-service project, and confirmed both services healthy/running afterward. Later 2026-09-18 supplied output verified the Palworld service-account rename to `game`, home move to `/home/game`, Compose recreation from the new paths, live bind mounts, runtime numeric identity, health, and unchanged world identifiers. Supplied 2026-09-19 output then verified the separate `game` GID migration from `985` to `5001`, both Palworld Compose identities at `995:5001`, healthy recreated containers against the existing data trees, and the dedicated `/mnt/nas-game` NFS commissioning. Later supplied 2026-09-19 migration output verified Prowlarr copied in a stopped state from `/home/media/docker/prowlarr` to the then-current `/home/rotom/docker/prowlarr`, recreated with `PUID=997` / `PGID=986`, running as the then-named `rotom:rotom`; the 2026-09-21 service-account rename subsequently moved that tree to `/home/infra/docker/prowlarr` without changing numeric IDs, preserving port `9696`, `/config` data, indexers, history, Sonarr/Radarr integrations, and reverse-proxy behavior. The same day qBittorrentVPN was migrated from `/home/media/docker/qbittorrentvpn` to the then-current `/home/rotom/docker/qbittorrentvpn`, changed to `PUID=997` / `PGID=5000`; the 2026-09-21 rename subsequently moved that project to `/home/infra/docker/qbittorrentvpn`, given Media/Game torrent-only binds and NAS fail-closed guards, and reverified for VPN/kill-switch behavior. Sonarr/Radarr were recreated with `/mnt/nas-media -> /media`. Gamarr was installed under `/home/game/docker/gamarr` at `995:5001`, port `6767`, with `/mnt/nas-game -> /game`, and the end-to-end Game download/import/hardlink/seeding workflow was accepted by the user.

Services are grouped by the Linux account directory containing their active Compose file. These directory associations do not by themselves establish the user inside each container. The original inventory had complete Compose working-directory metadata for only a subset of services, but the later 2026-09-19 full verification reconciled all 17 documented container records against the current Compose paths and Docker metadata. Application process identity remains a separate claim and is documented only where it was independently verified.

**Port notation:** Entries containing `->` are published host-to-container mappings. Bare ports are listed separately as container-only ports with no host mapping shown. “None shown” means the source PORTS field was blank; it does not establish network mode or lack of connectivity.

## 1A. Current Phase B Docker State — final JAR-68 acceptance, 2026-09-28

The Proxmox VE host `pve` remains hypervisor-only; the application Docker runtime is inside the Debian `rotom` VM. Current runtime foundation remains Docker Engine `29.8.1`, containerd `2.3.6`, Docker Compose `v5.5.1`, local Docker root `/var/lib/docker`, and package-created Docker group GID `989` with no members.

JAR-34 created the non-migrating v2 declarative namespace at `/srv/rotom`. It contains empty domain reservations under `stacks/` and service-owned empty domain roots under `appdata/`; it contains no placeholder Compose files, containers, databases, or production bind mounts. Existing production projects under `/home/<service>/docker` remain current until their individual Phase D workload tickets migrate and verify them. `/srv/rotom` is a local Git repository for only `stacks/`, `scripts/`, and support files; `.gitignore` excludes `appdata/`, `secrets/`, `backup-staging/`, databases, logs, caches, generated state, and environment files.

JAR-78 supersedes the Gameserver naming for the active Palworld domain: locked `game` is `995:5001` with home `/home/game`; `downloader` remains `901:5005` and `customapps` remains `904:5008`. The NAS-recovery helper retains the Game NFS readiness check; the unused Game Downloads bindfs view is retired. The Palworld modules were recreated one at a time after their paths moved to the Game domain. Existing numeric PUID/PGID values remain unchanged.

JAR-36 created unused external bridge networks `rotom-proxy` (`172.29.0.0/16`), `rotom-arr` (`172.30.0.0/16`), and `rotom-monitoring` (`172.31.0.0/16`). Future migrated stacks attach only services needing cross-stack connectivity, use Docker DNS/service names, and may retain private default networks. No current container is attached; current ports and bridge networks remain authoritative until owning workload migration tickets.

JAR-37 is the accepted Phase C runtime gate. Docker's daemon-wide `json-file` cap is `max-size=10m`, `max-file=3`; the current long-running containers use `restart: unless-stopped`. Healthchecks remain image-specific rather than synthetic: Arcane, Homepage, and Gamarr currently report healthy, while a missing healthcheck does not by itself indicate a fault. The reusable local policy is `/srv/rotom/stacks/RUNTIME-POLICY.md`: it requires backup/recovery readiness, configuration review, target-stack-only pull/recreation, readiness and application/storage/network/proxy verification, and target-stack-only rollback. Future mutable bind state belongs under `/srv/rotom/appdata/<domain>/...` where image support permits; Compose files must not contain secret values.

### JAR-75 Filesync Syncthing — 2026-09-29

### JAR-77 Gamarr retirement — 2026-09-30

Gamarr is retired. JAR-77 verified its live v2 Compose labels before removing `/srv/rotom/stacks/media/gamarr`, `/srv/rotom/appdata/media/gamarr`, and the legacy `/home/game/docker/gamarr`/standalone Gamarr config. Its container, private default network, port `6767`, Homepage card, NPM route, and recovery-worker branch are absent. At JAR-77 closeout, RomM retained its read-only `/mnt/nas-media/library/games` bind; JAR-76 later superseded that bind with the current read-only Game library path. Prowlarr retains only Radarr/Sonarr applications. At JAR-77 closeout, qBittorrentVPN's `Games` category was retained because its active/seeding state was not authenticated; JAR-79 later removed it after live zero-torrent verification. The targeted root-protected restore archive is `/srv/rotom/backup-staging/media/jar77-20260930/gamarr-config-and-compose.tar.gz`.

### JAR-79 Game torrent-directory retirement — 2026-10-01

qBittorrentVPN's local API reported zero torrents; its unused `Games` category was removed after the two associated `Downloader/.data` directories were confirmed empty, then removed with non-recursive `rmdir`. The active category set is Movies/Shows only. Live Compose inspection also established that qBittorrentVPN currently binds `/mnt/nas-media/torrents` at both `/media/torrents` and `/game/torrents`; JAR-79 made no change to that payload topology. This is a current-state correction to the conflicting older Downloader-path claims, which remain historical unless separately reconciled.

### JAR-83 minimal Game qBittorrent marker — 2026-10-01

The existing `Game/.data` export is again automounted at `/mnt/nas-game`, but it contains only `.rotom-qbt-media-ready/` (`901:5000`, mode `2750`) below its `988:5001` mode-`2770` root. qBittorrentVPN now requires that marker as a read-only `/run/rotom-nas-game` long bind with `create_host_path: false`, alongside the existing Media marker. Its active payload remains `/mnt/nas-media/torrents`; no Game torrent or library bind was restored. The 15-second guard requires both NFS mounts and marker directories. A controlled Game-marker absence test stopped qBittorrentVPN, and restoring the marker allowed clean container, guard, and WireGuard (`10.2.0.2/32`) recovery.

### Boot-time Game NAS recovery finding — 2026-10-01

During the 12:57 PDT guest boot, the initial `/mnt/nas-game` NFS mount failed while the NAS network route was unavailable. Docker consequently left JDownloader and RomM stopped with Game-bind startup errors; qBittorrentVPN, Radarr, Sonarr, and Jellyfin also remained stopped after their NAS bind startup failures. After the NAS returned, targeted remounts restored Game and Filesync, the expected Game inbox/library and Game/Media marker paths were verified, and the six affected containers were started successfully.

This does not change the payload topology or legacy-tree decision: current torrents remain exclusively at `/mnt/nas-media/torrents`, and the removed documented legacy `/home/*/docker` trees were not consumers. The Game `.rotom-qbt-media-ready` directory is a qBittorrentVPN sentinel only; `rotom-qbittorrent-media-guard` checks it with the Media mount/marker every 15 seconds and stops qBittorrentVPN when either safety pair is invalid, but does not restart it. JDownloader and RomM require their Game bind paths to start, but have no Game-specific sentinel, guard, or automatic recovery path.

### JAR-84 legacy service-home Docker-tree retirement — 2026-10-01

Fresh root Docker inspection found every active application Compose label under `/srv/rotom/stacks`, except the retained active NPM project under `/home/infra/docker/nginx-proxy-manager`. The inactive legacy `/home/downloaders/docker/{prowlarr,qbittorrentvpn}` and `/home/media/docker/{radarr,sonarr}` trees were archived, checksum-verified, covered by, and byte-read back from Restic snapshot `3c85d5be` before removal. `/home/game/docker` was verified empty and removed. `/home/downloaders/docker` remains because Arcane read-only mounts its parent; `/home/media/docker/jellyfin` and held `/home/web/docker` were not changed. In particular, JAR-84 supersedes the prior JAR-49 current-location conclusion: active Jellyfin is `/srv/rotom/stacks/media/jellyfin/compose.yaml`, while its former service-home tree remains retained legacy material.

### JAR-82 Nginx Proxy Manager Infra v2 migration — 2026-10-01

NPM now runs from `/srv/rotom/stacks/infra/nginx-proxy-manager/compose.yaml` with authoritative persistent state at `/srv/rotom/appdata/infra/nginx-proxy-manager/{data,letsencrypt}`. It retains host networking, its established image/container identity, DNS, restart policy, listeners, proxy-host database, access-list configuration, TLS material, and compatibility upstreams. The former `/home/infra/docker/nginx-proxy-manager` Compose/data tree is retained rollback material only and was not modified after the final state copy.

### JAR-73 Uptime Kuma — 2026-09-30

### PVE web UI Homepage entry — 2026-09-30

NPM serves the PVE HTTPS UI at `https://pve.rotom.casa`, forwarding over HTTPS to `192.168.1.68:8006`; the route has forced TLS, WebSocket upgrade, and an enabled access list. Homepage's **Rotom Management** section includes a **PVE** link to that URL without a `siteMonitor`, so the dashboard does not create a separate monitor request to the privileged PVE UI. Local-SNI HTTPS returned the Proxmox login page; Homepage stayed healthy and returned local HTTP `200`. The route did not require a Compose, container, listener, DNS, certificate, PVE-service, or network-policy change.

`uptime-kuma` uses `louislam/uptime-kuma:2` from `/srv/rotom/stacks/infra/uptime-kuma/compose.yaml`. Its SQLite data is VM-local at `/srv/rotom/appdata/infra/uptime-kuma`; it joins `rotom-proxy` and publishes only `127.0.0.1:3002:3001` for retained host-networked NPM compatibility. It has no Docker socket, NAS mount, or broad host listener. NPM routes `uptime.rotom.casa` with TLS and WebSockets; the administrative UI is authenticated and no public status page is enabled. Homepage's targeted `extra_hosts` mapping resolves `uptime.rotom.casa` to `192.168.1.69`, so its Uptime Kuma `siteMonitor` does not depend on container DNS availability.

Syncthing `2.0.0` runs from `/srv/rotom/stacks/filesync/syncthing/compose.yaml` as numeric `903:5007`, with persistent configuration/database below `/srv/rotom/appdata/filesync/syncthing`. It uses `restart: unless-stopped` and its image-provided healthcheck. Its only payload bind is `/mnt/nas-filesync -> /filesync`; the long bind has `create_host_path: false`, preventing Docker from substituting an unmounted local directory. An isolated nonexistent-source Compose probe verified this fails before fallback-directory creation. It has no Docker socket, privilege, added capability, device mapping, or unrelated NAS bind. It retains a private default network and attaches to `rotom-proxy`; its GUI is published only as `127.0.0.1:8384` for host-networked NPM compatibility. GUI authentication is configured and NPM forces TLS for `syncthing.rotom.casa`. The disposable test peer and acceptance payload were removed after verification, so no production folder or peer is preconfigured.

JAR-78 establishes the current Game domain at local stack roots `/srv/rotom/stacks/game/palworld-jared` and `/srv/rotom/stacks/game/palworld-fran`. Their authoritative world/runtime binds are `/srv/rotom/appdata/game/palworld-jared -> /palworld` and `/srv/rotom/appdata/game/palworld-fran -> /palworld`; root-only environment files remain under `/srv/rotom/secrets/game` and are not documented here. Each container uses its own private default network, `restart: unless-stopped`, image-provided healthcheck, and retained UDP mapping: Jared `8211/udp` plus `27015/udp`; Fran `8212/udp` plus `27016/udp`. Both are currently healthy after controlled one-at-a-time recreation. Native Palworld backup directories are under their local appdata roots. Neither stack mounts `/mnt/nas-game`; the retired `/home/game/docker/palworld-server-*` trees are historical rollback evidence only.

Docker socket access is exceptional: Arcane has the raw read-write socket for deliberate Docker administration only, with automatic updates and auto-heal disabled; Homepage has the read-only socket for dashboard metadata; Glances has the read-only socket plus its read-only host-root monitoring bind. A read-only socket mount does not prove a read-only Docker API. All other current application containers have no socket. Current deliberate runtime exceptions are Home Assistant (host networking, privileged, read-only D-Bus), Homebridge/Nginx Proxy Manager/Cloudflare DDNS (host networking), qBittorrentVPN (`CAP_NET_ADMIN` for WireGuard), and the stated Arcane/Glances mounts. Phase D workload tickets must rejustify rather than inherit any exception.

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

### Current Docker inventory — supplied 2026-10-01 output

The supplied root `docker ps -a` output records **25 container objects: 24 running and one exited**. `syncthing` is the sole exited object (`Exited (0)`); both Palworld containers are running and healthy; Gamarr is absent, consistent with JAR-77 retirement. The running set is `jdownloader`, `romm`, `nginx-proxy-manager`, `qbittorrentvpn`, both Palworld servers, `jellyfin`, `homepage`, `prowlarr`, `uptime-kuma`, `rotom-docs`, `romm-valkey`, `romm-db`, `paperless`, `paperless-db`, `paperless-broker`, `alohamillworks.com`, `homebridge`, `home-assistant`, `sonarr`, `radarr`, `arcane`, `cloudflare-ddns`, and `glances`.

`jdownloader` is the JAR-76 Game-domain direct-download service. Its authoritative Compose file is `/srv/rotom/stacks/game/jdownloader/compose.yaml`; it runs as `995:5001` with `unless-stopped`, no host-published ports, no added capabilities, and `no-new-privileges`. Its only writable NAS bind is `/mnt/nas-game/downloads/romm-inbox -> /opt/JDownloader/Downloads`; its VM-local configuration is `/srv/rotom/appdata/game/jdownloader/config`. MyJDownloader inputs are root-administered files below `/srv/rotom/secrets/game/jdownloader`, mounted read-only to the service and never recorded by value. Live verification completed a MyJDownloader-submitted direct download to the inbox.

### JAR-38 Infra v2 convergence — 2026-09-28

The four migrated Infra modules each have an independent root-owned Compose definition: `/srv/rotom/stacks/infra/homepage/compose.yaml`, `glances/compose.yaml`, `arcane/compose.yaml`, and `cloudflare-ddns/compose.yaml`. These definitions were validated and recreated without an image pull; the verified cached images are Homepage, Glances, Arcane, and Cloudflare DDNS.

Homepage configuration is now `/srv/rotom/appdata/infra/homepage/config`; Glances configuration is `/srv/rotom/appdata/infra/glances/glances.conf`; Arcane data is `/srv/rotom/appdata/infra/arcane`, owned `997:5002`. Protected inputs are in `/srv/rotom/secrets/infra` and are Git-ignored. Homepage uses the supported `HOMEPAGE_VAR_*` substitution for its protected provider credential; its configuration contains no credential value. Homepage and Glances attach to `rotom-monitoring` and resolve `glances` through Docker DNS. Homepage additionally attaches `rotom-proxy`; Arcane attaches `rotom-proxy`. Glances exposes no host port. Homepage retains host `3001` and Arcane `3552` only as compatibility upstreams for the still-host-networked NPM; those paths returned local/proxied HTTP 200. NPM remains intact on 80/81/443 for JAR-39.

Cloudflare DDNS now uses its ordinary private `cloudflare-ddns_default` bridge, preserves its read-only token mount/capability drop/no-new-privileges contract, and immediately reported `rotom.casa` current. Arcane retains only its documented raw Docker socket and Infra-stack write bind; its original Downloader Compose compatibility bind remains read-only. `autoUpdate=false` and `autoHealEnabled=false` were reverified. To avoid expanding privileged secret access, Arcane has no v2 secret-directory bind; protected-env stack discovery warnings are intentional and do not authorize it to read secrets. The legacy `/home/infra/docker` module directories and `arcane_arcane-data` volume remain untouched rollback artifacts.

A controlled guest reboot recovered all four modules; Homepage/Arcane health, Homepage→Glances DNS/API, local and HTTPS Homepage, Arcane HTTPS, DDNS reconciliation, NPM compatibility, qBittorrent `wg0`, and zero failed systemd units passed. JAR-48 later adds baseline dashboard coverage using these existing components only; notification delivery is intentionally not configured.

## 2. Core Infrastructure — `infra`

### JAR-5 private Rotom documentation portal — 2026-09-29

`rotom-docs` runs the pinned Nginx static image from `/srv/rotom/stacks/infra/rotom-docs/compose.yaml`; its pinned `mkdocs.yml` lives beside that Compose definition. The portal binds only `127.0.0.1:8082` for retained host-networked NPM compatibility and joins `rotom-proxy`; it has no Docker socket, privileges, or NAS mount. Its root filesystem and `/srv/rotom/appdata/infra/rotom-docs/site` bind are read-only. A one-shot pinned MkDocs Material builder reads `/srv/rotom/appdata/infra/rotom-docs/source` read-only and writes the generated site only; it runs with all capabilities dropped and no network.

The root-owned `scripts/publish-source` is the controlled publication mechanism. It copies only the nine approved RPD members `00`–`08` from the canonical checkout into a derived local source tree, produces the portal landing page, and wraps the canonical `08` text tree in rendered Markdown without modifying the canonical source. Run it deliberately followed by the build profile after future verified RPD changes; the live portal is not a write path into the RPD.

### JAR-40 Media v2 convergence — 2026-09-28

### JAR-10 RomM deployment — 2026-09-29

RomM 5.3.1 with MariaDB 11 and Valkey 9 runs from `/srv/rotom/stacks/media/romm/compose.yaml`. Its state, database, config, assets, resources, cache, and verified logical export are VM-local under `/srv/rotom/appdata/media/romm`; protected database/application inputs are root-only under `/srv/rotom/secrets/media/romm`. JAR-76 changes only its library bind to `/mnt/nas-game/library/games:/romm/library:ro` and enables scheduled quick scans at minutes 7, 22, 37, and 52. It has no Docker socket or privileged mode and exposes only loopback `127.0.0.1:8081` for retained host-networked NPM.

Jellyfin, Radarr, Sonarr, and Prowlarr are active v2 modules under `/srv/rotom/stacks/media/<app>/compose.yaml`; JAR-84 directly verified all four Compose labels and active bind sources. Jellyfin retains its read-only `/mnt/nas-media/library` bind, while Radarr/Sonarr mount `/mnt/nas-media/torrents` at `/media/torrents`. qBittorrentVPN retains its v2 downloader stack/appdata paths and Media payload binds. The former service-home Radarr/Sonarr/Prowlarr/qBittorrentVPN trees are retired recovery artifacts; the Jellyfin tree is deliberately retained. All proxyable services attach `rotom-proxy`; Radarr/Sonarr/Prowlarr attach `rotom-arr`; qBittorrentVPN also attaches `rotom-arr`. Existing ports remain NPM compatibility upstreams.

Core infrastructure; Compose files under `/home/infra/docker/`.

| Current container name | Observed image | Observed status | Published ports | Container-only ports shown | Compose file |
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

Homepage's top-level **Backup Schedule** section appears immediately above **Rotom Monitoring** and contains exactly these custom-API cards in order: **Rotom VM Backup Schedule**, **Rotom Restic Backup Schedule**, and **PVE Restic Backup Schedule**. JAR-70 renamed the former **Rotom Backup Schedule** card only; its guest Restic endpoint remains `http://192.168.1.69:8787/backup-status`. The PVE cards use the LAN-only, read-only endpoints **Rotom VM Backup Schedule** at `http://192.168.1.68:8789/rotom-vm-backup-status` and **PVE Restic Backup Schedule** at `http://192.168.1.68:8789/pve-restic-backup-status`. Each preserves `mdi-backup-restore`, a 60-second refresh, and mappings for Schedule, Last Result, Next Run, and Last Run. Homepage's `settings.yaml` layout map lists **Backup Schedule** first with `style: row` and `columns: 3`, which renders the section at the top as one row of three equal cards. JAR-70's configuration diffs changed only the section placement/order, guest-card label, and that layout entry; Rotom Monitoring's non-backup cards, all endpoints, custom API behavior, Compose project, Glances, backup controls/schedules/retention/repositories/storage, VZDump configuration, and recovery points remained unchanged. Homepage hot reload, Compose validation, API/status checks, and the apex HTTPS check passed.


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

## 2A. Downloads — final pre-migration JAR-21 state (historical name/path)

JAR-21 moved the active download stack to the dedicated `downloads` account (`901:5005`). The old Infra project trees were retained during acceptance and then removed after checksum/active-path guards and a successful Restic backup.

| Current container name | Image | Current verification | Published ports | Compose file |
| --- | --- | --- | --- | --- |
| `prowlarr` | `lscr.io/linuxserver/prowlarr:latest` | Active; HTTP 302; Downloads-owned | `9696:9696` | `/home/downloads/docker/prowlarr/compose.yaml` |
| `qbittorrentvpn` | `binhex/arch-qbittorrentvpn:latest` | Active; HTTP 200; WireGuard `wg0` present; Downloads-owned | `8080:8080` | `/home/downloads/docker/qbittorrentvpn/compose.yaml` |

Prowlarr uses Downloads identity `PUID=901` / `PGID=5005` and keeps its persistent `/config` beneath `/home/downloads/docker/prowlarr`. qBittorrentVPN runs as numeric `901:5005`; its persistent `/config` is `/home/downloads/docker/qbittorrentvpn/config`, and its project `.env` remains secret material that must never be printed or documented.

qBittorrent's active storage contract is one host tree, `/mnt/nas-downloads/torrents`, mounted read/write at both `/media/torrents` and `/game/torrents` to preserve existing category paths. Its startup guard is the read-only `/mnt/nas-downloads/.rotom-qbt-nas-ready -> /run/rotom-nas-downloads` bind with `create_host_path: false`. Default and category paths remain `/media/torrents`, `/media/torrents/incomplete`, Movies/Shows under `/media/torrents/...`, and Games under `/game/torrents/...`. VPN egress and the kill switch were reverified during JAR-21.

Radarr/Sonarr/Gamarr do not receive write access to the Downloads share. Persistent bindfs units expose `/mnt/nas-downloads` through identity-mapped read-only views, and Compose mounts only the torrent subtree from those views at the existing `/media/torrents` or `/game/torrents` container paths. Imports from Downloads to Media/Game are therefore cross-filesystem copies, not hardlinks.

Arcane remains owned by Infra. For project discovery it has `/home/downloads/docker:/home/downloads/docker:ro`, follows `/home/infra/docker/downloads -> /home/downloads/docker`, and has `followProjectSymlinks=true`. Final Arcane registry verification showed only the correct Prowlarr and qBittorrentVPN records at the symlinked Downloads paths; JAR-21 duplicate Infra/Rotom records are gone.

## 3. Media — `media`

Media services; Compose files under `/home/media/docker/`.

| Current container name | Observed image | Observed status | Published ports | Container-only ports shown | Compose file |
| --- | --- | --- | --- | --- | --- |
| `jellyfin` | `lscr.io/linuxserver/jellyfin:latest` | Running at JAR-84 verification | `0.0.0.0:8096->8096/tcp`, `[::]:8096->8096/tcp` | `8920/tcp` | `/srv/rotom/stacks/media/jellyfin/compose.yaml` |
| `radarr` | `lscr.io/linuxserver/radarr:latest` | Running at JAR-84 verification | `0.0.0.0:7878->7878/tcp`, `[::]:7878->7878/tcp` | None shown | `/srv/rotom/stacks/media/radarr/compose.yaml` |
| `sonarr` | `lscr.io/linuxserver/sonarr:latest` | Running at JAR-84 verification | `0.0.0.0:8989->8989/tcp`, `[::]:8989->8989/tcp` | None shown | `/srv/rotom/stacks/media/sonarr/compose.yaml` |

### Current media identity and NAS access

The Rotom `media` account remains UID **127** and primary GID **5000** (formerly **129**). The three current media-owned services—Jellyfin, Radarr, and Sonarr—retain the established media identity pattern:

```text
PUID=127
PGID=5000
UMASK=002
```

Prowlarr and qBittorrentVPN are Downloads-owned services at `901:5005` after JAR-21. Radarr, Sonarr, and Jellyfin remain media-owned at numeric `127:5000`; Jellyfin additionally has GPU-related supplementary groups needed for `/dev/dri` access.

`/mnt/nas-media` has root ownership `988:5000`, mode `2770`. Existing mixed owner UIDs in the library are preserved. The former Media torrent tree was removed after JAR-21; active torrent objects are now created in Downloads storage under the Downloads identity/GID contract.

Sonarr and Radarr mount the complete Media NAS at `/media` and bind `/mnt/nas-media/torrents` read-only at `/media/torrents`; Jellyfin's library mount remains read-only. qBittorrentVPN binds `/mnt/nas-media/torrents` read/write. JAR-86 retired the unused `/mnt/nas-downloads-media-ro` bindfs view after confirming it had no container or Compose consumer. Only qBittorrent uses the VPN and its kill switch was directly verified. **JAR-49 current state:** PVE attaches the NUC's Iris Plus Graphics 655 as optional `hostpci0`; Rotom maps its Intel `01:00.0` GPU to `/dev/dri/card1` and `/dev/dri/renderD128`. Active Jellyfin maps `/dev/dri`, adds guest GIDs `44` (video) and `992` (render), and runs under its existing `127:5000` service identity. Jellyfin is deliberately configured for QSV at `/dev/dri/renderD128`, with H.264/HEVC hardware processing enabled. A forced live H.264 transcode initialized Intel `iHD`/VA-API and used `h264_qsv`, then exited `0`; this attachment, device mapping, and Jellyfin runtime survived a complete PVE reboot. QSV is optional: remove `hostpci0` and the Compose device/group mappings to return Jellyfin to software transcoding on portable hardware. Coffee Lake Gen9.5 supports H.264 plus HEVC 8/10-bit acceleration but not AV1; H.264 High 10 and HEVC RExt remain software-only limitations. Low-Power encoder and VPP tone-mapping remain disabled because HuC firmware was not commissioned.

Ordinary `jared` access to the media NAS is intentionally excluded. Administration uses `sudo -iu media` on Rotom after connecting as Jared. Jared's final account listing confirms he is not a member of media GID `5000`. UNAS's observed `--manage-gids` behavior means adding a client-side supplementary group alone does not grant the tested NFS access; using primary GID 5000 succeeded. See 04 and 07 for identity details and verification commands.

## 4. Game — `game`

Palworld services use the locked `game` identity at UID/GID `995:5001` and local state under `/srv/rotom/appdata/game/`. Palworld stays on local storage. Gamarr is retired; the old Game library is retained only for JAR-52 rollback.

| Current container name | Observed image | Observed status | Published ports | Container-only ports shown | Compose file |
| --- | --- | --- | --- | --- | --- |
| `palworld-server-fran` | `thijsvanloef/palworld-server-docker:latest` | **Intentionally stopped** at final JAR-68 acceptance; preserved, not dead/restarting/OOM; `unless-stopped` | `0.0.0.0:8212->8212/udp`, `[::]:8212->8212/udp`, `0.0.0.0:27016->27016/udp`, `[::]:27016->27016/udp` | `25575/tcp` | `/home/game/docker/palworld-server-fran/compose.yaml` |
| `palworld-server-jared` | `thijsvanloef/palworld-server-docker:latest` | **Intentionally stopped** at final JAR-68 acceptance; preserved, not dead/restarting/OOM; `unless-stopped` | `0.0.0.0:8211->8211/udp`, `[::]:8211->8211/udp`, `0.0.0.0:27015->27015/udp`, `[::]:27015->27015/udp` | `25575/tcp` | `/home/game/docker/palworld-server-jared/compose.yaml` |
| `gamarr` | `ghcr.io/gamarr-app/gamarr:latest` | Running; healthy in fresh 2026-09-19 audit (image-provided healthcheck) | `0.0.0.0:6767->6767/tcp`, `[::]:6767->6767/tcp` | None shown | `/home/game/docker/gamarr/compose.yaml` |

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

## 4A. Documents — `documents`

### JAR-8 Paperless-ngx deployment — 2026-09-29

Paperless-ngx `v3.2.1`, PostgreSQL `18`, and Valkey `9` run from `/srv/rotom/stacks/documents/paperless/compose.yaml` (local commit `86a8432`). Mutable data, media, exports, consume, database, and broker state are VM-local below `/srv/rotom/appdata/documents/paperless`; credentials remain only under `/srv/rotom/secrets/documents/paperless`. The web service has loopback-only `8000` access and joins `rotom-proxy`; database/broker have no host ports. NPM route `paperless.rotom.casa -> 127.0.0.1:8000` uses the existing wildcard certificate.

A harmless local PDF import completed OCR, title search, original retrieval, and survived web-service recreation. Its supported export (manifest, metadata, PDF, thumbnail) is protected local backup staging. Generic guest Restic includes `/srv`, but no verified snapshot includes this deployment. Rotom's public-IP hairpin TLS cannot test the route locally, but an independent public TLS assessment verified the expected login redirect and HTTP `200` login page through `paperless.rotom.casa`.

## 5. Smart Home — `smarthome`

### JAR-43 Smart Home v2 convergence — 2026-09-29

Home Assistant and Homebridge now use independent root-administered Compose modules at `/srv/rotom/stacks/smarthome/home-assistant/compose.yaml` and `/srv/rotom/stacks/smarthome/homebridge/compose.yaml`. Their authoritative mutable state is explicitly bound from `/srv/rotom/appdata/smarthome/home-assistant:/config` and `/srv/rotom/appdata/smarthome/homebridge:/homebridge`, respectively. The state was copied with preserved numeric metadata while each service was stopped. JAR-51 later retired the inactive legacy `/home/smarthome/docker/{home-assistant,homebridge}` rollback trees after no active configuration reference was found and the current Compose/services were verified.

Both services retain `network_mode: host` and `restart: unless-stopped`; this preserves Home Assistant discovery and Homebridge/Avahi mDNS behavior. No Docker bridge/network, port, NPM proxy-host, certificate, NAS mount, service UID/GID, or backup-policy change was made. Home Assistant's `/etc/localtime` and `/run/dbus` read-only binds remain intact. Homebridge retains its explicit JSON-file limit of 10 MB with one retained file.

Post-reboot verification confirmed both containers running from the v2 mounts; local `8123`/`8581` and `https://home-assistant.rotom.casa` / `https://homebridge.rotom.casa` each returned HTTP 200. Both Compose modules parsed, Home Assistant and Homebridge SQLite `quick_check` results were `ok`, mDNS/SSDP listeners were present, the guest Restic timer was active, and zero failed systemd units were reported. The existing guest Restic `/srv` source scope covers the v2 state; no backup or restore was run in JAR-43.

| Current container name | Observed image | Observed status | Published ports | Container-only ports shown | Compose file |
| --- | --- | --- | --- | --- | --- |
| `home-assistant` | `ghcr.io/home-assistant/home-assistant:stable` | Running; JAR-43 post-reboot verification passed | None shown (host network) | None shown | `/srv/rotom/stacks/smarthome/home-assistant/compose.yaml` |
| `homebridge` | `homebridge/homebridge:latest` | Running; JAR-43 post-reboot verification passed | None shown (host network) | None shown | `/srv/rotom/stacks/smarthome/homebridge/compose.yaml` |

The Homebridge container name was corrected on 2026-09-19 from the historical misspelling `homebrige` to `homebridge`. The Compose directory and image name were already correct. Homebridge started successfully after the change and restored its cached accessories; the whole-home scan reported no remaining `homebrige` references.

## 6. Web Hosting — `web`

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

- The current Compose inventory follows `/home/<account>/docker/<service>/compose.yaml` across six active workload-owner directories: `infra`, `media`, `game`, `smarthome`, `downloaders`, and `web`. The additional service-account homes (`documents`, `filesync`, and `customapps` at compatibility path `/home/apps`) currently have no deployed application workload. The former `auth` account/home and NAS boundary were retired on 2026-09-29.
- JAR-22 records 16 running containers and 16 total container objects. `jaredwines.com` has a retained Compose project at `/home/web/docker/jaredwines.com/compose.yaml` but no current container object. `glances` shares the Homepage Compose stack at `/home/infra/docker/homepage/compose.yaml`.
- Published mappings include both IPv4 and IPv6 bindings for most mapped services. Glances shows an IPv4 loopback binding only. These listings do not establish firewall rules, router forwarding, or external reachability.
- The original container/path inventories alone do not establish volumes, networks, environment variables, dependencies, VPN behavior, or reverse-proxy routing. Later evidence adds the media identity settings and mounts above; use 03 and 04 for separately audited network and storage details. Full current Compose contents still need inspection before editing any service.

### Service groups and NAS mounts — current guest state through JAR-31

Production workloads are restored under the current service-account layout. The table below describes current guest identity/storage relationships; historical Docker-group membership is not implied.

| Service account | Current guest UID:GID | Current guest NAS mount | Current Docker/application use |
|---|---:|---|---|
| media | `127:5000` | `/mnt/nas-media` | Jellyfin/Radarr/Sonarr; active torrent path is `/mnt/nas-media/torrents` |
| game | `995:5001` | `/mnt/nas-game` | Both Palworld servers plus private JDownloader; Game ROM inbox/review/library are current JAR-76 paths |
| infra | `997:5002` | `/mnt/nas-infra` | Arcane/DDNS/Homepage/Glances/NPM configs remain local under `/home/infra/docker` |
| smarthome | `126:5003` | `/mnt/nas-smarthome` | Home Assistant/Homebridge configs remain local under `/home/smarthome/docker` |
| documents | `900:5004` | `/mnt/nas-documents` | Storage boundary only |
| downloaders | `901:5005` | `/mnt/nas-downloaders` | Active Prowlarr/qBittorrentVPN owner; current NFS source `Downloader/.data`; qBittorrent `wg0` verified |
| web | `902:5006` | `/mnt/nas-web` | Aloha active; Jared Wines retained but intentionally undeployed |
| filesync | `903:5007` | `/mnt/nas-filesync` | Storage boundary only |
| customapps | `904:5008` | `/mnt/nas-customapps` | Reserved storage boundary only; compatibility home `/home/apps` |

Current application restores use `/home/<service>/docker/<project>`. JAR-86 retired the unused `/mnt/nas-downloads-media-ro` Media read-only compatibility view and its recovery-helper branch; its source `/mnt/nas-downloaders`, root, and active qBittorrent sentinel `/mnt/nas-downloaders/.rotom-qbt-nas-ready` remain. JAR-81 removed only the obsolete Downloader `torrents/` tree. The unused Game view was already retired.

The current `docker` group should be re-read before future privilege changes. JAR-30 GID `989` with no members is the last explicit verification; JAR-31 application restoration does not by itself demonstrate group membership changes.

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

Read-only Docker checks on 2026-09-15 confirmed 15 running containers and one Created container, `jaredwines.com`. That count/status is historical and was superseded first by the 2026-09-19 consolidated audit and later by JAR-22/JAR-31. Other still-valid 2026-09-15 observations remain historical evidence only; use **Current Phase B Docker State** in section 1A for the current 16-container runtime.

## 11. Reconstruction-Critical Information

For current reconstruction, start with **Current Phase B Docker State** (section 1A) and the current guest service-account table in section 7. The dated pre-migration sections remain recovery evidence and must not be replayed blindly where they use historical `downloads`, old network allocations, or pre-VM status text. Preserve each active service's current owning account, Compose path, image, container name, published ports/network mode, persistent binds/volumes, restart behavior, and non-secret environment-variable names.

UID/GID account definitions are canonical in `07-Users-and-Permissions.md`; NAS export/mount semantics are canonical in `04-NAS-and-Storage.md`; full backup policy is canonical in `05-Backup-and-Restore.md`.

## 12. Proposed Services

No additional Docker service is promoted to current state by this restructure. Any future service documented elsewhere remains Proposed until deployment evidence is recorded here and in the change log.

## 13. Outstanding / Needs Verification

The VM-era backup baseline is no longer a Docker gap. The current verified guest recovery snapshot is `f666d63c` (`2026-09-27 13:30:28 PDT`, 17.410 GiB), and post-run `restic check` passed 21/21 snapshots with no errors. Guest Restic automation is now recommissioned under enabled/active `rotom-restic-backup.timer`; this does not change Docker backup scope, and `/mnt` remains outside the host Restic source set.

- **Intentional:** Fran is not recreated, and her historical backup sudoers rule is not installed.
- **Needs Verification — live Downloader-NFS loss:** startup/recreation fail-closed behavior and boot-time recovery are verified, but behavior after sudden NFS loss while qBittorrent is already running is not. Read-only inspection of the current Compose/systemd definitions can confirm that no runtime watchdog is documented; actual failure behavior would require a separately planned non-destructive maintenance test and must not be inferred.
- **Known Arcane metadata drift:** current auto-update/auto-heal settings were freshly verified on 2026-09-27 (document 06). JAR-85 verified the active Arcane container mounts `/home/downloaders/docker` directly; the former `/home/infra/docker/downloads` discovery symlink was unreferenced by the current Arcane database and retired. Historical Arcane records that name the old symlink remain historical evidence. Its persisted qBittorrentVPN project status remains stale/`unknown` even though Docker runtime and WireGuard are current and healthy enough to be running.

## 14. Related Documentation

- `03-Network-and-Domains.md` — canonical Docker network, listener, port, domain, and proxy routing details.
- `04-NAS-and-Storage.md` — canonical NFS/mount/storage contracts and hardlink behavior.
- `07-Users-and-Permissions.md` — canonical Linux account, UID/GID, group, and privilege model.
- `06-Maintenance-and-Automation.md` — update/restart automation and recurring operational behavior.
