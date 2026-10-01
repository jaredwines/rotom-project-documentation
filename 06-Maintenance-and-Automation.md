# 06 - Maintenance and Automation

**Documentation set:** Rotom Project Documentation  
**Document role:** Canonical source for scheduled/routine maintenance, automation, monitoring behavior, and operational administration workflow  
**Hosts:** PVE hypervisor `pve` and Debian VM `rotom`  
**Baseline verified:** historical workload evidence through 2026-09-25; Phase B host/VM foundation plus live automation refresh verified 2026-09-27  
**Documentation updated:** 2026-10-01 — JAR-86 unused Media bindfs view retired and verified
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `07-Users-and-Permissions.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## 1. Purpose and Scope

This document is the canonical detailed owner for what Rotom does automatically or routinely over time, including systemd/cron/application automation and the surrounding operational administration workflow. The RPD maintenance contract is governed exclusively by `00-Rotom-Change-Log.md`; this document does not duplicate it. Full Restic configuration remains canonical in `05-Backup-and-Restore.md`.

### Evidence provenance

Host: Rotom
Evidence date: 2026-09-15, America/Los_Angeles
Scope: Read-only inspection of systemd, cron, Docker, backups, monitoring, and application automation. No Rotom service, timer, container, backup, or configuration was changed while collecting evidence.
Documentation updated: **2026-09-28** through JAR-34's verified non-migrating `/srv/rotom` foundation, the Git-backed RPD checkout, and Jared-global Codex discovery integration; Proxmox host-config Restic commissioning after the backup-control/recovery-point refresh following JAR-67 remains current; JAR-31 application/recovery restoration and reboot acceptance plus the JAR-30 Docker/containerd/logging baseline remain current. JAR-34 adds no scheduler or workload automation: its root-owned local Git repository versions only future declarative `stacks/`/`scripts/` content, while mutable `appdata`, secrets, and backup staging are ignored and remain outside that history. Existing application paths, backups, and timers remain unchanged.

## 2. Current Maintenance and Automation — 2026-09-28 JAR-68 final

JAR-78 retains the Game NFS readiness check in `rotom-nas-docker-recovery`, but retires the unused `rotom-downloads-game-ro.service` and `/mnt/nas-downloads-game-ro` view after confirming no container consumer. JAR-79 then removed qBittorrent's unused Games category and its two empty Downloader directories. JAR-81 retains the Downloader NFS root and sentinel while removing only its obsolete `torrents/` tree. JAR-86 then retired the unused Media bindfs view and removed only its recovery-helper branch, retaining the separate Downloader and Media NFS readiness checks. JAR-83 restores the established `/mnt/nas-game` automount as a minimal marker contract and makes its marker an explicit qBittorrent safety dependency.

JAR-36 maintains three empty Docker network contracts; they are not scheduled automation and have no container attachments. The tracked `/srv/rotom/stacks/NETWORKING.md` contract directs later workload tickets to attach only required services and use Docker DNS.

JAR-37 disables Arcane automatic updates and auto-heal. Arcane remains a manual/observational Docker-administration tool, so no unattended service mutation is part of the current automation baseline. Deliberate container updates follow `/srv/rotom/stacks/RUNTIME-POLICY.md`: verify backup/recovery readiness, inspect the target configuration, pull and recreate only that stack, wait for readiness, verify its application/storage/network/proxy behavior, and roll back only that stack if required.

JAR-38 converted Homepage, Glances, Arcane, and Cloudflare DDNS to independent v2 Compose lifecycles under `/srv/rotom/stacks/infra`; the non-secret definitions are local-Git commit `a17a217`. The migrated modules retain `unless-stopped`. Homepage and Glances recover on `rotom-monitoring`; Arcane on `rotom-proxy`; Cloudflare DDNS on its private bridge. Their post-reboot acceptance passed. Arcane’s automatic update/heal settings remain false. JAR-73 later added alerting as described below.

JAR-40 added independent Media v2 modules at local commit `f5f2da4`. The five migrated services retain `unless-stopped`; Gamarr retains its image healthcheck. NPM remains the established host-networked reverse proxy, so the applications retain host compatibility ports while joining their v2 Docker networks. No update automation, backup schedule, download payload, or VPN behavior changed.

`rotom-qbittorrent-media-guard.timer` is enabled and active every 15 seconds, with `/usr/local/sbin/rotom-qbittorrent-media-guard` and its matching systemd service. JAR-83 verifies it requires the underlying Media payload NFS mount and marker plus the Game NFS mount and `.rotom-qbt-media-ready` marker. qBittorrentVPN has matching read-only Compose binds with `create_host_path: false`; the Media payload remains unchanged. If either marker/mount pair is absent while qBittorrentVPN is running, the guard stops it rather than allowing an accidental local write. A controlled Game-marker absence test stopped the container; restoring the marker allowed clean container and WireGuard recovery.

JAR-72 reorganized Homepage cards without changing monitoring behavior or any application boundary. **Documents**, immediately below **Hosted Websites**, contains Paperless and the private Rotom Docs portal; Docs is intentionally a link/container-status card without an HTTP monitor, so Homepage does not receive a trusted-LAN access-policy exemption. **Media** contains Jellyfin, Radarr, Sonarr, Prowlarr, and RomM; JAR-77 retired the former Gamarr card and force-recreated only Homepage, which returned healthy and `https://rotom.casa` `200`. **Downloader**, immediately above **NAS Storage**, contains qBittorrent. Historical rollback artifacts remain historical evidence.

### PVE host

- Host identity is `pve` / `pve.rotom.casa`; canonical Mac SSH aliases are `pve` / `pve.rotom.casa`.
- `pvedaemon` and `pveproxy` are enabled for the PVE HTTPS UI. NPM serves it at `https://pve.rotom.casa` through HTTPS to `192.168.1.68:8006`, with forced TLS, WebSocket upgrade, and an enabled access list. Homepage links to it from **Rotom Management** without a monitor. This is an administrative access path, not an automation or a PVE control API.
- PVE host-config Restic: `pve-restic-backup.timer` enabled/active at `04:00`, `Persistent=true`, `AccuracySec=1min`; service `pve-restic-backup.service`; worker `/usr/local/sbin/pve-restic-backup`; mount `/mnt/nas-pve-restic-backup`; staging `/var/backups/pve-restic-recovery`; retention `7 daily / 4 weekly / 12 monthly`.
- Whole-VM automatic job: `rotom-vm-daily`, enabled at `05:00`, VMID 100, storage `nas-rotom-vm-backup`, snapshot + zstd, `repeat-missed=0`, retention `7 daily / 4 weekly / 6 monthly`.
- Whole-VM manual launcher/service/worker: `/usr/local/bin/backup-rotom-vm-to-nas`, `rotom-vm-vzdump-manual.service`, `/usr/local/sbin/rotom-vm-vzdump-manual`; lock `/run/lock/rotom-vm-vzdump-manual.lock`. The launcher re-executes through `sudo` for non-root invocation, while the worker and systemd service remain root-owned. Jared can follow the service with `journalctl -fu rotom-vm-vzdump-manual.service` through `systemd-journal` membership. Disconnect-safe ownership and duplicate-run guard were verified.
- PVE temperature bridge: `/usr/local/sbin/pve-cpu-temp-api` + `pve-cpu-temp-api.service`, endpoint `192.168.1.68:8788`; Homepage label `PVE CPU Temperature`.
- PVE backup-status bridge: `/usr/local/sbin/pve-backup-status-api` + `pve-backup-status-api.service`, LAN-only endpoint `192.168.1.68:8789`; `/pve-restic-backup-status` and `/rotom-vm-backup-status` supply Homepage's PVE Restic and Rotom VM Schedule cards. It is read-only and has no backup-control endpoint.
- JAR-74 weekly maintenance: root-owned `0750` `/usr/local/sbin/weekly-host-maintenance`, `pve-weekly-maintenance.service`, and enabled `pve-weekly-maintenance.timer` run Sundays at `00:30` with `Persistent=true`, `AccuracySec=1min`, and no randomized delay. Before work, the locked worker requires no failed unit, at least 10 GiB free on `/` and `/var`, no active `pve-restic-backup.service` or `rotom-vm-vzdump-manual.service`, and all `pvesm` storage active. It waits up to ten minutes for APT locks, runs normal `apt-get upgrade` without autoremove or automatic reboot, then limits cleanup to APT downloads, existing tmpfiles rules, and archived journals above 500 MiB. Post-run it logs host health, free space, and an advisory `REBOOT REQUIRED` state; it never reboots. The 2026-09-29 manual run passed, upgraded eight ordinary packages, retained 57 GiB free, and exited successfully.
- NIC WOL/offload mitigation remains as previously documented. Final physical reboot/post-reboot acceptance returned zero failed PVE units and VM100 auto-started.

### Rotom VM

- Docker/containerd remain enabled. Current intended runtime is 16 container objects / 14 running because both Palworld containers are intentionally stopped; their worlds remain preserved.
- `rotom-nas-docker-recovery.service` remains the NAS-backed container recovery helper. JAR-86 healthy-state validation passed with all twelve NFS automounts, the retained Downloader sentinel and Media paths, and qBittorrentVPN `wg0` `10.2.0.2/32`; the unused Media bindfs view and its branch are absent.
- Guest Restic remains `rotom-restic-backup.service` / `.timer` around `03:00`, with manual `backup-restic-to-nas`; guest naming was intentionally not changed by JAR-68.
- JAR-47 repaired the v2 Home Assistant SQLite staging path and verified snapshot `aedd8e57`; its restricted restore passed Palworld archive checksums and Home Assistant SQLite integrity. Media uses daily 12:00 AM UNAS-local snapshots with a 16-snapshot limit for authoritative library rollback.
- JAR-71 Remote Desktop Commander is the native enabled `desktop-commander.service`, running outbound-only as `desktopcmd` with `UMask=0077`, `NoNewPrivileges=true`, and `PrivateTmp=true`. It has no Docker, NAS, proxy, DNS, or inbound-listener dependency; `/home/desktopcmd/workspace` is its only intended writable work area.
- JAR-74 weekly maintenance: root-owned `0750` `/usr/local/sbin/weekly-host-maintenance`, `rotom-weekly-maintenance.service`, and enabled `rotom-weekly-maintenance.timer` run Sundays at `01:15` with `Persistent=true`, `AccuracySec=1min`, and no randomized delay. The same lock, failed-unit, 10 GiB capacity, and APT-lock checks apply; it additionally blocks if `rotom-restic-backup.service` is active or any running Docker container reports `unhealthy`. It uses normal package upgrade only, never autoremove or reboot; cleanup is limited to APT downloads, existing tmpfiles rules, archived journals above 500 MiB, and dangling Docker images. It never prunes Docker containers, volumes, networks, NAS mounts/data, snapshots, Restic state, or application data. Post-run journal output records system/Docker health, free space, and advisory reboot-required state. The 2026-09-29 manual run passed, upgraded three ordinary packages, retained 32 GiB free, found intended running containers healthy, and reported no reboot required.
- Weekly Linux update script remains present but the reboot-capable cron schedule remains absent unless separately recommissioned.

### Current PVE recovery points

- PVE host-config Restic: surviving canonical snapshot `35d4b0c2`; two pre-rename snapshots deliberately forgotten; repository check PASS.
- Whole-VM: exactly one fresh verified archive `vzdump-qemu-100-2026_09_28-00_09_19.vma.zst`, `47,415,540,796` bytes; zstd and full VMA verification PASS; currently unprotected and subject to normal `7/4/6` retention.

The intended staggered cadence remains guest Restic around `03:00`, PVE host-config Restic at `04:00`, and whole-VM PVE backup at `05:00`.

## 2A. Preserved Pre-Migration Automation Overview

| Time or trigger | Automation | Verified behavior |
|---|---|---|
| Boot | Docker | Docker and containerd are enabled. Docker starts after networking and containerd. |
| Post-Docker boot / retry on failure | `rotom-nas-docker-recovery.service` | Verifies the retained Downloader and Media NFS contracts, then recovers only stopped NAS-backed containers with recorded NAS/mount startup errors. Enabled; JAR-86 healthy-state validation passed after retiring the unused Media bindfs unit and branch. |
| Every minute and container start | Cloudflare DDNS | Checks its configured DNS record and updates it if needed. |
| Every 30 seconds | Arcane auto-heal | Enabled; may restart unhealthy managed containers. |
| Midnight | Arcane automatic updates | Enabled with no exclusions. |
| 00:00 daily | Palworld backups | Each Palworld server creates a game-save archive. |
| 02:00 daily | Palworld updates | Each Palworld server checks for and installs game updates. |
| Around 03:00 daily | NAS backup | Restic backup timer with a random delay of up to 10 minutes. |
| Sunday 04:00 | Weekly Linux update | Updates packages and may reboot if the system requires it. |
| System managed | OS maintenance | APT, log rotation, firmware refresh, trim, package database backup, filesystem checks, and cleanup. |

## 3. Preserved Pre-Migration systemd, Mounts, and Startup Ordering

Relevant enabled timers are unas-backup, apt-daily, apt-daily-upgrade, anacron, dpkg-db-backup, logrotate, fwupd-refresh, fstrim, e2scrub_all, man-db, plocate-updatedb, motd-news, and systemd-tmpfiles-clean.

Docker and containerd are enabled and active. Docker wants network-online and starts after network-online and containerd. Docker itself has no explicit NAS mount requirement.

The NAS backup service is a root oneshot service. It requires /mnt/nas-rotom-backup and starts after network-online and the mount. It is inactive between scheduled runs; the latest inspected run completed successfully.

The backup status API service is enabled and active. It restarts on failure and starts after networking, but has no explicit NAS mount dependency.

The current fstab-backed NAS paths keep their existing automount model. JAR-78 retired the stale unavailable `/mnt/nas-gameserver` entry and local mountpoint; `/mnt/nas-game` is the sole active Game contract. The retired `/mnt/nas-downloads` boundary and reserved `/mnt/nas-customapps` boundary do not create a global Docker NAS dependency. JAR-86 retired the unreferenced `rotom-downloads-media-ro.service` and local view; the Downloader and Media NFS readiness checks remain independent.

qBittorrent uses a service-specific Docker/Compose startup guard rather than a global Docker→NAS systemd dependency. The active `/mnt/nas-downloads/torrents` bind and Downloads sentinel bind use `create_host_path: false`; deliberate removal of the Downloads sentinel caused recreation to fail as required. This prevents local-directory fallback on recreation. A live runtime NFS-loss watchdog was not implemented.

No rc.local hook exists. The at scheduler is not installed. No per-user crontabs were found. The only observed user timer was launchpadlib-cache-clean for jared.


### JAR-23 NAS-backed Docker recovery hardening — 2026-09-25

After the September 23 boot race left the Downloads bindfs views and several NAS-backed containers stopped, JAR-23 added root-owned executable `/usr/local/sbin/rotom-nas-docker-recovery` and systemd unit `/etc/systemd/system/rotom-nas-docker-recovery.service`. The unit is enabled under `multi-user.target`, `Requires=docker.service`, runs after `docker.service` and `network-online.target`, and uses `Restart=on-failure` with a 30-second delay so a transiently unavailable NAS can be retried without making all Docker services depend on the NAS.

The helper explicitly starts/verifies `/mnt/nas-downloaders` and `/mnt/nas-media` as NFS, verifies the Downloader `.rotom-qbt-nas-ready` sentinel and Media library path, then inspects qBittorrentVPN, Radarr, Sonarr, and Jellyfin. JAR-77 removed its retired Gamarr branch, JAR-78 retired the unused Game bindfs branch, and JAR-86 retired the unused Media bindfs view and its helper branch. Already-running containers are left untouched. A stopped container is started only when its restart policy is `unless-stopped`/`always` and its recorded Docker error matches a NAS/mount startup-failure signature; other stopped containers are left stopped. This protects intentionally stopped workloads from being indiscriminately started.

Manual validation on the healthy server completed `Result=success`, `ExecMainStatus=0`, and `active (exited)` while reporting all five covered containers already running. Zero failed systemd units remained. The helper SHA-256 was `ea1ac1b6db7da89317b50b0de95a07a4be2cab5a7d530e5c3c1c1ed30bdb9d85`; the unit SHA-256 was `92b3b8d4bce02f574abb49b63031bcdca066a999459a6cebc79a7ac178486ac0`. Duplicate journal messages during the test were cosmetic because the helper logs both to stdout and `logger`. Full reboot-path validation is intentionally deferred until preservation work is complete.

### JAR-24 final backup and repository verification — 2026-09-25

JAR-24 used the existing `/usr/local/bin/backup-to-nas` launcher and did not change the timer, include/exclude roots, repository format, password-file location, or retention policy. A stale Restic lock left by an aborted read-only helper was removed with the supported `restic unlock` command only after verifying no Restic process and no active `unas-backup.service`. The successful canonical run saved final snapshot `fbe1838e` at `2026-09-25 18:14:22 PDT`, applied the existing 7-daily / 4-weekly / 12-monthly policy, removed five redundant same-day snapshots, and completed prune/repack/index cleanup.

Post-backup verification ran both `restic check` and full `restic check --read-data`; all 971 packs were read with no errors. A representative isolated restore recovered both JAR audit artifacts, all three staged SQLite databases, and both Palworld world saves; restored databases passed `PRAGMA quick_check`, and the restore scratch directory was deleted. The JAR-24 evidence set is `/home/jared/audits/jar-24-2026-09-25`. This verification does not change the standing schedule: `unas-backup.timer` remains the automatic daily path, and `/mnt` remains excluded from host Restic.


### JAR-25 live-imaging maintenance workflow — 2026-09-25

JAR-25 performed the full-disk acquisition as a controlled maintenance window rather than a normal recurring job. The workflow first captured the exact 16-container running set and the active subset of selected maintenance units, then stopped those maintenance units and all Docker workloads, flushed journald/filesystem writes, and streamed `/dev/nvme0n1` read-only directly to the UNAS Shared Drive. After the first full image write, the workflow restored the captured runtime set before beginning the long complete NAS-image reread/checksum pass. Final acceptance reported the original 16-container set restored and zero failed systemd units.

This procedure is not installed as recurring automation. Its consistency classification remains best-effort live/crash-consistent because the root filesystem stayed mounted. Do not treat the existence of the image as proof that a restore has been rehearsed.

A session-management issue was also isolated before the real run: a normal `tmux new -s jar25` session exited immediately, while a configuration-bypassed detached session using `/bin/bash --noprofile --norc` remained stable. The root cause of the default tmux/shell exit was not investigated. For long preservation commands, verify the tmux session actually persists before starting work, and verify the acquisition process/image file exists before assuming a pasted command is running. The 64 MiB JAR-25 dry run successfully tested the exact NVMe-to-NAS streaming and complete reread/hash path before the full image.

## 4. Preserved Pre-Migration Cron and Operating-System Updates

At the preserved pre-migration baseline, a custom root cron job was defined in `/etc/cron.d/weekly-linux-update` as `0 4 * * 0 root /usr/local/sbin/weekly-linux-update`, so it ran every Sunday at 04:00 as root. A fresh direct read of the executable verifies it logs to `/var/log/weekly-linux-update.log`, uses `set -e`, runs `apt-get update`, `apt-get -y upgrade`, `apt-get -y autoremove`, and `apt-get -y autoclean`, and invokes `/usr/sbin/reboot` only when `/var/run/reboot-required` exists. The executable is `root:root` mode `0755`.

Regular daily and weekly cron scripts contained no additional Docker action, reboot, or package-update command.

APT daily timers are enabled. The unattended-upgrades package and standard unattended-upgrades configuration files were absent. Mint automatic-upgrade and autoremove timers are installed but disabled.

## 5. Preserved Pre-Migration Docker Restart, Health, Logs, and Update Automation

At the pre-migration configuration baseline, all 17 documented service definitions used the `unless-stopped` restart policy. JAR-22 later had 16 deployed container objects because Jared Wines was intentionally absent; current JAR-31 runtime is likewise 16 deployed containers.

JAR-22 verified 16 running containers and 16 total container objects. Aloha remains running from `/home/web/docker/alohamillworks.com`. Jared Wines retains `/home/web/docker/jaredwines.com/compose.yaml`, but no current container object exists; do not recreate one solely to match older inventory wording.

Arcane, Homepage, both Palworld servers, and Gamarr currently report Docker health checks. Gamarr's healthcheck is supplied by the image (`/etc/s6-overlay/s6-rc.d/svc-gamarr/data/check`); its Compose file has no explicit `healthcheck:` key. qBittorrentVPN has no Docker healthcheck and remains verified through functional/VPN checks instead.

At the pre-migration baseline, all containers used Docker `json-file` logging and Homebridge was the only container with explicit per-container limits (10 MB, one retained file); no daemon-wide Docker log configuration was found then. **Current VM state differs:** JAR-30 configured daemon-wide `json-file` limits of `10m` × `3` in `/etc/docker/daemon.json`.

### Arcane

**Current live verification — 2026-09-27:** Arcane automatic updates are enabled at midnight using seconds-first cron syntax `0 0 0 * * *`, with no excluded containers. Auto-heal is enabled every 30 seconds (`*/30 * * * * *`), with no excluded containers, maximum five restarts, and a 30-minute restart window. `followProjectSymlinks=true`. Scheduled Docker pruning is disabled (`scheduledPruneEnabled=false`); configured prune modes therefore remain dormant scheduled-prune defaults rather than an active recurring prune job.


Arcane has no notification providers configured.

Arcane's active container mounts the Downloaders-owned canonical project root `/home/downloaders/docker` directly. `followProjectSymlinks=true` remains configured, but JAR-85 verified that Arcane's current database does not reference the former `/home/infra/docker/downloads` discovery path; that inactive symlink was retired. Historical project-state records that name it remain historical evidence. Docker independently verifies qBittorrentVPN running with `wg0` up; any stale Arcane project metadata is not runtime authority. Arcane does not manage Palworld under `/home/game/docker`, so Arcane updates do not overlap Palworld's updater.

Read-only/final JAR-21 inspection of `/var/lib/docker/volumes/arcane_arcane-data/_data/arcane.db` verifies the Prowlarr/qBittorrentVPN rows are reconciled to the Downloads symlink paths. Unrelated stale historical Smart Hub/Smarthome records remain technical debt. Arcane also previously recorded registry pull retry and rate-limit failures.


### JAR-22 maintenance snapshot — 2026-09-22

The pre-migration audit found **zero failed systemd units** and 13 active/listed system timers. `unas-backup.timer` remained enabled with its next scheduled run around 03:00, Docker and containerd remained enabled, and `rotom-downloads-game-ro.service` plus `rotom-downloads-media-ro.service` remained enabled. Root had no crontab. `/etc/cron.d/weekly-linux-update` still schedules `/usr/local/sbin/weekly-linux-update` at `0 4 * * 0` (Sunday 04:00). No maintenance job, timer, unit enablement, or container state was changed by JAR-22.

## 6. Preserved Pre-Migration Palworld Automation

Both Palworld containers generate Supercronic jobs at startup:

- 00:00 daily: backup
- 02:00 daily: update
- automatic reboot: disabled
- backup retention: 30 days

The 02:00 job ran successfully on 2026-09-15. Each server created a pre-update backup, shut down cleanly, installed the game update, and restarted. Docker itself did not restart.

The newest archive from each server passed gzip validation and contains world `Level.sav` plus player-save files. During the account/home migration both servers were stopped, the complete `.sav` sets were SHA-256 compared before and after the move with no differences, and both containers were recreated healthy from `/home/game/docker/...`. This verifies archive readability, migration integrity, and current startup, not a full restore.

## 7. Preserved Pre-Migration Backup Scheduling Reference

Restic stores the host backup repository at /mnt/nas-rotom-backup/rotom-restic-backup.

The automated backup retains seven daily, four weekly, and 12 monthly snapshots tagged automatic, then prunes expired snapshots.

Jared and Fran can start the same full backup manually with the shared `/usr/local/bin/backup-to-nas` launcher. The launcher is `root:root` mode `0755` and executes `/usr/local/sbin/backup-to-unas` through `sudo`; the protected script remains `root:root` mode `0700`. After the backup share changed to mode `0700`, both human launch paths were verified. During the later Palworld migration, a stopped-state run produced `53c989ec` at 23:00:33 PDT and the post-migration run produced `0d64ab6d` at 23:08:43 PDT; the latter run then removed `53c989ec` under the normal retention policy and completed prune. This confirms that ordinary-user denial does not block the privileged backup path and that same-day manual snapshots can be expired immediately by retention.

This command runs the normal whole-server backup, including its retention and prune phase. It is not a Fran-only backup, and Fran does not need direct access to `/mnt/nas-rotom-backup`; `/home/fran` is covered by the existing `/home` include. Treat `/usr/local/bin/backup-to-nas` as the canonical manual command. Earlier per-user launcher copies may still exist because their cleanup was not shown. **The latest documented recovery point is JAR-24 snapshot `fbe1838e` at `2026-09-25 18:14:22 PDT`.** Its canonical run completed normal retention/prune, and JAR-24 then completed full repository `--read-data` verification and an isolated restore. Earlier snapshots such as `e886c55d` and `f0d51eaa` remain historical evidence for the states and tests documented at those times.

Backup scope includes system configuration, user homes, local application data, system information, and Docker volumes. It excludes temporary and cache data, logs, Docker overlay storage, Docker container logs, and mounted external filesystems.

Protection acceptance began on 2026-09-18 and later verification through 2026-09-19 established:

- share root `988:988`, mode `0700`;
- the original pre-rename ordinary direct-access matrix denied `jared`, `fran`, `rotom`, `media`, `game-server` (then UID/GID `995:985`), `smart-home`, and `web-host`;
- a fresh 2026-09-19 matrix after the Game GID migration also denied current `game` (`995:5001`) and the other six ordinary named accounts;
- root retained read/write/traverse on both the mount root and Restic repository;
- latest path-verified snapshot in this documentation, `f0d51eaa` at 06:39:30 PDT on 2026-09-19, contains the then-current Prowlarr/qBittorrentVPN/Gamarr/Palworld/Home Assistant/website paths; later Homebridge changes remain pending post-change snapshot verification;
- retention/prune completed successfully, including removal of redundant same-day stopped-state snapshot `53c989ec`;
- `unas-backup.timer` remains enabled and active, and a later inspected timer-triggered `unas-backup.service` run completed with exit status `0/SUCCESS`; the service is normally inactive/dead between runs.

Earlier repository metadata, partial data-pack verification, protected-size, and SQLite `quick_check` results remain valid historical evidence.

A representative isolated Restic restore has now been tested successfully: both current Palworld `Level.sav` files plus selected Home Assistant and service configuration files were restored under `/tmp`, inspected, and the scratch tree was removed. This does not constitute a full bare-metal/system disaster-recovery rehearsal or a full end-to-end Palworld server restore; those remain untested.

## 8. Preserved Pre-Migration Monitoring, Notifications, Certificates, and DDNS

Homepage and Glances remain in the same Compose project. Glances has a read-only host-root bind, read-only Docker socket, and read-only `glances.conf` bind, but **no longer mounts `/mnt/nas-rotom-backup`**. The NAS-backup storage metric previously exposed through Glances/Homepage is intentionally removed; no replacement backup-capacity monitoring was added in this change.

No Glances warning or critical threshold was found. A read-only Arcane database query found no notification-provider records, so Arcane currently has no configured alert destination.

No active Home Assistant YAML configuration for email, Telegram, Discord, Slack, mobile app, webhook, or alert delivery was found. Built-in notification blueprints exist but do not prove active alert delivery.

### JAR-48 baseline dashboard monitoring — 2026-09-28

Homepage's `Rotom Monitoring` section uses Glances v4 through Docker DNS on `rotom-monitoring` for Rotom system information, CPU, memory, container activity, local disk I/O and capacity, and host network traffic. It now includes **Media NAS Storage**, the authoritative Glances filesystem metric `fs:/mnt/host-root/mnt/nas-media`, and **UNAS Reachability**, an IPv4 ICMP check to `192.168.1.70`. The active Media and Downloader mounts are NFSv3 runtime dependencies; the reserved `nas-customapps` share intentionally has no check. The former Auth account/mount and stale Gameserver guest mount were retired.

Homepage retains active HTTPS/internal site monitors for Nginx Proxy Manager, Arcane, Home Assistant, and Homebridge; backup-status and PVE temperature/backup-status cards remain read-only. JAR-73 adds an Uptime Kuma link under Rotom Management and a deliberately small alerting layer: `/usr/local/sbin/rotom-alerting` runs each minute via `rotom-alerting.timer`, sources protected Infra secrets, sends Rotom/Pushover alerts only on state transitions, checks Restic/PVE backup-status APIs, Media NFS with a three-minute grace, root-disk thresholds, and the qBittorrent guard. The PVE counterpart runs each minute as `pve-alerting.timer` for an independent heartbeat and PVE root-disk thresholds. Healthchecks monitors external Rotom, PVE, and monitoring heartbeats at 3-minute period plus 2-minute grace, with Pushover integration. For planned work, run `sudo rotom-alerting maintenance {rotom|pve|monitoring|all} MINUTES`; it pauses selected Healthchecks checks with manual-resume protection, suppresses local notifications, and automatically resumes at expiry.

Nginx Proxy Manager checks certificate renewal hourly for certificates expiring within 30 days. A direct TLS inspection verified the currently served `*.rotom.casa`/`rotom.casa` Let's Encrypt certificate is valid from 2026-09-11 through 2026-12-10. No recent renewal error was found; a real future renewal event has not yet been observed.

Cloudflare DDNS checks every minute and at container startup. Its inspected record was already current. It has no Docker health check.

## 8A. JAR-31 Homepage Monitoring Reconciliation — 2026-09-26 checkpoint

The final compatibility restore left Homepage healthy but exposed monitoring drift inherited from the pre-virtualization host. The remediation is now part of the current Phase B monitoring baseline:

- Homepage internal URL checks were failing with repeated `EAI_AGAIN` for `nginx.rotom.casa`, `arcane.rotom.casa`, `home-assistant.rotom.casa`, and `homebridge.rotom.casa`. Targeted `extra_hosts` entries in the Homepage service now resolve those names, plus `rotom.casa`, to the Rotom VM at `192.168.1.69`. All four affected HTTPS checks returned HTTP 200 afterward and no new `EAI_AGAIN` appeared during verification.
- The Rotom Backup Schedule widget was moved to `http://192.168.1.69:8787/backup-status`. At this JAR-31 checkpoint the restored timer was intentionally held and the API returned `schedule=INACTIVE`, `enabled=disabled`, `next_run=Unknown`, `last_run=Unknown`, and `last_result=success`. **This timer state is historical:** later on 2026-09-27 the controls were renamed and `rotom-restic-backup.timer` was deliberately recommissioned enabled/active; the current status API then reported `schedule=Active`, `enabled=enabled`, and `next_run=9/28/2026 3:07:23 AM`.
- Disk Usage changed from the obsolete bare-metal `disk:nvme0n1` metric to `disk:sda` after Glances `diskio` verified the VM devices and `/dev/sda2` root parent.
- The Debian VM exposes no physical CPU package-temperature sensor. PVE exposes `coretemp` `Package id 0`. The canonical read-only helper is `/usr/local/sbin/pve-cpu-temp-api` with `/etc/systemd/system/pve-cpu-temp-api.service`, binding `192.168.1.68:8788`. Its `/api/4/sensors` response is intentionally Glances-compatible so Homepage can retain the original graph behavior rather than falling back to a non-charting custom API tile.
- Homepage CPU Temperature uses the Proxmox bridge as a Glances v4 widget with `metric: sensor:Package id 0`, `chart: true`, `refreshInterval: 1000`, and `pointsLimit: 15`. The separate `/temperature` endpoint remains available for a simple JSON health/readout check.

The temperature helper reads `/sys/class/hwmon` dynamically to locate `coretemp` / `Package id 0`; it does not expose credentials, does not change Proxmox application boundaries, and is separate from the Rotom VM's Glances container.

## 8B. Current Proxmox Wake-on-LAN and NIC stability workaround — 2026-09-27

Proxmox physical `nic0` is the Intel I219-V at PCI `0000:00:1f.6` using the `e1000e` driver (MAC `1c:69:7a:0e:f2:f7`). It reports `Supports Wake-on: pumbg`, `Wake-on: g`, and PCI wake `enabled`. Historical WOL and offload changes were initially captured as `/etc/network/interfaces.pre-wol` and `/etc/network/interfaces.pre-e1000e-offload-20260927`; JAR-51-era cleanup retired those inactive copies after reference checks. The current `iface nic0 inet manual` stanza runs both `post-up /usr/sbin/ethtool -s nic0 wol g` and `post-up /usr/sbin/ethtool -K nic0 tso off gso off`; no separate WOL or NIC-offload service/timer is installed. `ifquery --check nic0` and `ifquery --check vmbr0` pass.

The TSO/GSO change was added after the 2026-09-27 evening outage, where the previous Proxmox boot repeatedly logged `e1000e 0000:00:1f.6 nic0: Detected Hardware Unit Hang` from approximately `20:56:04` through `20:59:46`. During the same failure window Proxmox reported `nas-rotom-proxmox-backup` offline and the Rotom guest reported NFS server `192.168.1.70` not responding. The NIC transmit hang therefore explains the observed network/NFS outage. The journal then ends abruptly before a new Proxmox boot; there is no recorded orderly shutdown, kernel panic, MCE, OOM, thermal shutdown, or persistent pstore crash record tying the NIC hang directly to the whole-host reboot. That causal link remains **Needs Verification**.

The mitigation was first applied live with `ethtool -K nic0 tso off gso off` and then persisted in `/etc/network/interfaces`. Verification showed `tcp-segmentation-offload: off`, `generic-segmentation-offload: off`, `generic-receive-offload: on`, `Speed: 1000Mb/s`, `Duplex: Full`, `Wake-on: g`, and `Link detected: yes`. GRO, checksum offloads, EEE, bridge settings, and other NIC features were intentionally left unchanged so the workaround remains narrow and reversible. Status is **Implemented / Monitoring** rather than a proven permanent fix. After a future normal reboot or power cycle, verify TSO/GSO remain disabled and review `journalctl -k -b` for new `Hardware Unit Hang`, `NETDEV WATCHDOG`, or `e1000e` reset events.

A controlled physical `systemctl poweroff` performed earlier the same day was followed by a successful host return. Afterward `ethtool nic0` still reported `Wake-on: g`; VMID 100 was `running` with `onboot: 1`. The Rotom guest initially hit transient failed Downloader/Media/Game NFS mounts and exposed only the 11 non-NAS-dependent containers, then `rotom-nas-docker-recovery.service` recovered qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin and exited `0/SUCCESS`. Final guest state was zero failed units, all 16 expected containers, real NFSv3 mounts, and qBittorrentVPN `wg0` up at `10.2.0.2/32`.

This verifies persistent host-side WOL configuration and the complete controlled post-power-cycle Proxmox/VM/application recovery path. The transcript does not include a magic-packet sender invocation, so it does not establish that the NUC's power-on was actually triggered by WOL; that end-to-end trigger remains **Needs Verification**.

## 9. Preserved Pre-Migration Wake-on-LAN

This section refers to the retired Linux Mint bare-metal host and must not be used as current Proxmox WOL configuration. The wired NIC `eno1` reports `Supports Wake-on: pumbg` and `Wake-on: g`, verifying magic-packet Wake-on-LAN is enabled at the host/NIC level. No scheduled Wake-on-LAN sender or custom WOL script was found. End-to-end delivery while Rotom is off/suspended remains untested.

## 9A. Current NAS/Docker Maintenance State — 2026-09-27 refresh

- Twelve guest fstab entries continue to generate `.automount`/`.mount` units with `_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s`.
- Current `/mnt/nas-downloaders` uses NAS `Downloader/.data`; the earlier JAR-29/JAR-30 `Downloads/.data` current-state claim is superseded by live fstab/generated-unit/findmnt/showmount evidence.
- Docker and containerd are enabled/active at boot. Docker runtime remains local; NAS-backed application recovery is handled by the targeted helper rather than a global Docker→NAS dependency.
- `rotom-nas-docker-recovery.service` is enabled under `multi-user.target`, requires Docker, starts after Docker/network-online, and retries on failure. The JAR-31 reboot proved the retry path: first Media mount attempt failed, the next invocation succeeded and recovered exactly the affected NAS-backed containers.
- The helper leaves already-running workloads untouched and recovered qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin after matching their prior NAS mount startup-failure state.
- The historical two-view state is superseded: the Game view is retired, while the enabled Media view exposes `/mnt/nas-downloaders` read-only but has no current container consumer and is proposed for retirement.
- `rotom-restic-backup-status-api.service` is enabled/active on `0.0.0.0:8787`; UFW is not installed in the current guest.
- Restic automatic execution is commissioned: `rotom-restic-backup.timer` is enabled/active on the daily `03:00` schedule with persistent catch-up and up to ten minutes randomized delay. Weekly update cron remains absent; Fran backup sudoers remains absent.
- **zsh scripting lesson, reconfirmed during JAR-31 diagnostics:** never use `path` as an interactive zsh loop/scalar variable. It mutates the special array tied to `$PATH` and produced `command not found` for `findmnt`, `systemctl`, `sudo`, and `sed`; setting `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`, exporting it, and running `rehash` restored command lookup without any host failure.

## 10. Preserved Pre-Migration Service-Identity and NAS Maintenance

At the final pre-migration baseline, Media was **UID `127`, primary GID `5000`**; Jellyfin, Radarr, and Sonarr used `127:5000`, while Gamarr used `995:5001`. JAR-21 moved Prowlarr and qBittorrentVPN to the then-named `downloads` identity `901:5005` under `/home/downloads/docker`, with torrent data at `/mnt/nas-downloads/torrents`. **Current VM naming is `downloaders` with `/home/downloaders` and `/mnt/nas-downloaders`; numeric identity `901:5005` is unchanged.**

The pre-migration NAS Media root was `988:5000` mode `2770`, and the final library tree was the authoritative Media application data. JAR-21 moved active torrent data to Downloads GID `5005`. Those numeric ownership rules remain relevant to reconstruction, but current host paths use the VM-era `downloaders` naming. Use numeric ownership when diagnosing NFS; Rotom's displayed `fwupd-refresh` name for numeric `988` does not identify the real NAS account.

The adopted access model keeps ordinary `jared` access separate from Media: administer Media on the **Rotom VM** by switching with `sudo -iu media`. A 2026-09-19 pre-migration check verified Jared was not a member of Media GID `5000`; no current evidence contradicts that policy. Root, sudo, and privileged Docker access still allow administrators to cross these ordinary account boundaries.

In the observed UNAS isolated NFS setup, `rpc.mountd --manage-gids` rebuilds supplementary groups from the NAS's UID lookup and retains the request's primary GID. UID `1000` (`jared` on Rotom) maps to UNAS `jwines760`, primary GID `988`, supplementary GID `987`. Client supplementary GID `5000` alone did not permit media access; primary GID `5000` did. Treat `jared` being denied media access as intentional isolation, and test maintenance access using the media account rather than broadening permissions.

At the pre-migration backup-protection checkpoint, `/mnt/nas-rotom-backup` presented numeric `988:988` with share-root mode `0700`. The 2026-09-19 denial matrix verified all seven ordinary named accounts denied read/write/traverse while root retained repository access. That old export/path is now retired. **Current backup storage is `/mnt/nas-rotom-restic-backup` on `Rotom_Restic_Backup`, also presenting `988:988` mode `0700`; a fresh named-user denial matrix was not rerun after the JAR-32 cutover.**

For a **current read-only** check, run **on the Rotom VM as `jared`**:

```bash
id jared
id media
findmnt -T /mnt/nas-media
findmnt -T /mnt/nas-rotom-restic-backup
stat -c '%A %a %u:%g %n' /mnt/nas-media /mnt/nas-rotom-restic-backup
sudo -iu media
```

Then run **on the Rotom VM as `media`** to inspect the effective service identity and current Media paths without changing files:

```bash
id
ls -ldn /mnt/nas-media /mnt/nas-media/library /mnt/nas-media/torrents
exit
```

The final `exit` returns to the original `jared` shell. These are suggested checks, not fresh results from this documentation update. Inspect an existing service's `compose.yaml` and mounts before changing it; avoid recursive ownership resets or permission broadening.

### Service mounts — preserved pre-migration state

| Service account | Pre-migration UID:primary GID | Pre-migration NAS mount | Historical operational state |
|---|---|---|---|
| `media` | `127:5000` | `/mnt/nas-media` | Active application share |
| `game` | `995:5001` | `/mnt/nas-game` | Active application share |
| `infra` | `997:5002` | `/mnt/nas-infra` | Commissioned automount; no workload bound |
| `smarthome` | `126:5003` | `/mnt/nas-smarthome` | Commissioned automount; HA/Homebridge local |
| `documents` | `900:5004` | `/mnt/nas-documents` | Commissioned automount; empty |
| `downloads` | `901:5005` | `/mnt/nas-downloads` | Active torrent storage and Prowlarr/qBittorrentVPN domain |
| `web` | `902:5006` | `/mnt/nas-web` | Commissioned automount; websites local |
| `filesync` | `903:5007` | `/mnt/nas-filesync` | Commissioned automount; empty |
| `apps` | `904:5008` | `/mnt/nas-apps` | Commissioned automount; empty |
| `auth` | `905:5009` | `/mnt/nas-auth` | Commissioned automount; empty |

All eight JAR-6 mounts use the standard fstab automount pattern. Downloads now has a qBittorrent-specific sentinel plus the two persistent read-only bindfs view services; there is still no global Docker NAS dependency or runtime-loss watchdog. The other seven new shares remain boundary-only. Host Restic excludes `/mnt`.

### Service-home `~/nas-<service>` administration convention — pre-migration 2026-09-21

All ten operational service accounts now have a verified service-specific symbolic link following `~/nas-<service> -> /mnt/nas-<service>`. For example, `media` uses `~/nas-media`, `downloads` uses `~/nas-downloads`, and `infra` uses `~/nas-infra`. The links were initially created as generic `~/nas` names, then renamed in place. Final verification confirmed every new link target and intended-account traversal and ended with `ALL NAS SHORTCUT RENAMES: PASS`.

Treat `~/nas-<service>` as an administrator convenience, not an operational dependency. Compose files, systemd units, backup/recovery scripts, qBittorrent sentinel/bind paths, storage checks, and troubleshooting commands should continue to use the canonical `/mnt/nas-*` path unless there is a specific reason to test the shortcut itself. No automation, mount, permission, container, or NAS configuration changed when the links were created or renamed.

### JAR-6 automount verification lessons — 2026-09-21

- An active systemd automount initially exposes an `autofs` layer. `findmnt -T <mountpoint>` can therefore report `systemd-1 autofs` instead of the underlying NFS mount. Trigger the path first, wait for the `.mount` unit, then verify with `findmnt -M <mountpoint> -t nfs,nfs4` or equivalent.
- Private `2770` service roots intentionally prevent ordinary Jared traversal. Run functional metadata checks as the intended service account (or with suitable privileged access), not as Jared and then misread `Permission denied` as a share failure.
- When using `sudo -u <service>` for diagnostics, start from a neutral traversable working directory such as `/` if the child command may restore/inspect its initial working directory.
- JAR-6's first automount commissioning attempt rolled back cleanly after an autofs-verification script error; the successful retry retained `/root/fstab.jar6-retry-20260921-163659.bak`.

### JAR-21 Downloads maintenance state — 2026-09-21

- Active Prowlarr: `/home/downloads/docker/prowlarr`.
- Active qBittorrentVPN: `/home/downloads/docker/qbittorrentvpn`; runtime `901:5005`; WireGuard `wg0` verified.
- Active torrents: `/mnt/nas-downloads/torrents`; fail-closed sentinel `/mnt/nas-downloads/.rotom-qbt-nas-ready`.
- `rotom-downloads-media-ro.service` and `rotom-downloads-game-ro.service` are enabled and active and must remain available before Arr imports can see completed Downloads data.
- Old Media/Game torrent trees and old Infra Prowlarr/qBittorrent project trees are removed.
- Arcane follows `/home/infra/docker/downloads -> /home/downloads/docker` with a read-only same-path Downloads bind and `followProjectSymlinks=true`.
- Final post-cleanup backup snapshot `a6d86b20` succeeded; final `systemctl --failed` reported none and `systemctl is-system-running` returned `running`.

### JAR-9 rollback maintenance state — 2026-09-21

`/home/jared/rotom-manual-verify.sh` was restored from its pre-JAR-9 backup and passes shell syntax validation. Current service/NAS checks again use `game`, `/home/game`, `/mnt/nas-game`, and both qBittorrent Media/Game sentinels. The JAR-9 `/mnt/nas-game-servers` automount is retired from `fstab`. For NAS shell administration from the Mac, `ssh nas` is now a verified working alias.

## 11. Preserved Pre-Migration Absence or Disabled Automation

- No Docker prune schedule was found.
- No at scheduler or queued at jobs exists.
- No rc.local startup hook exists.
- No per-user crontabs exist.
- Mint automatic upgrade and autoremove timers are disabled.
- Palworld automatic reboot is disabled.
- No custom systemd service or timer references a missing executable.
- No duplicate host-level Docker update or reboot schedule was found.

## 12. Command-Line Safety, Administration, and Manual Synchronization Workflow

### Large command block safety

For Rotom administration, **large or multi-step command blocks should run inside an isolated Bash process by default**, for example:

```bash
bash <<'EOF'
set -Eeuo pipefail

# commands

EOF
```

This keeps `set -e`, `exit`, shell-option changes, traps, and command failures inside the child Bash process instead of applying them to or terminating the administrator's interactive SSH login shell. A failure may end the child Bash block with a nonzero status, but the parent SSH session should remain open. Small single-purpose commands do not require this wrapper. Commands that intentionally need to modify the current interactive shell state are exceptions and must be identified explicitly. This is a project-wide Rotom command-safety convention for setup, troubleshooting, migrations, storage work, Docker work, recovery, audits, and maintenance.

### Documentation and synchronization workflow

The current workflow is:

- Use the native **macOS Terminal** app. The current verified administration entry points are `ssh rotom.casa` (or alias `ssh rotom`) for Debian user `jared`, and `ssh pve` (or `ssh pve.rotom.casa`) for PVE user `jared`. The Mac SSH config uses dedicated identity files `~/.ssh/id_ed25519_rotom` and `~/.ssh/id_ed25519_pve` respectively, with `IdentitiesOnly yes`, `AddKeysToAgent yes`, and `UseKeychain yes`. The Rotom public key is authorized in `/home/jared/.ssh/authorized_keys`; the PVE public key is authorized in PVE Jared's account and the matching root-authorized key was removed. Jared's PVE `sudo` and `jared@pam` Administrator access were verified on 2026-09-29. Password-authentication policy remains unchanged; retain PVE root for emergency recovery. Ghostty has been removed from the Mac.
- **Codex execution lanes — adopted 2026-09-29:** Codex performs routine live **Rotom** inspection and separately authorized implementation through the concrete `rotom` SSH alias as non-root `jared`, invoking `sudo` only for the specific privileged operation. Before a task relies on this agent-operated lane, Codex performs a noninteractive, read-only SSH access preflight. It then reads the relevant RPD, inspects the live state, compares live evidence with the RPD, makes the smallest authorized change, and verifies the final live state. A failed Rotom SSH preflight is an access blocker to report; it is not permission to change SSH configuration, use credentials, or silently perform the work through another host.
- **PVE is Jared-executed:** Jared manually runs **every** PVE check and change in a PVE SSH session. Codex must provide concise, ticket-specific copy/paste blocks in sequence: (1) read-only preflight identifying the exact target, (2) one targeted action, and (3) post-action verification with the expected outcome. Codex does not initiate PVE actions itself and must wait for the returned output before advancing when that output establishes safety or success. Jared's supplied output is the PVE evidence source for ticket, Linear, and RPD records. There is no universal PVE mutation command; commands remain specific to the verified ticket scope.
- **Shared safeguards:** Do not include credentials or key material in command blocks. Stop for destructive actions, required secrets or credentials, protected backup/recovery behavior or data, or genuine unresolved ambiguity. On a failure, investigate the supplied evidence and use the next safe, scoped approach rather than issuing blind retries. Keep Linear current for active ticket milestones and update the RPD only from final supported evidence.
- ChatGPT-controlled browser work is not part of the Rotom administration or documentation workflow. Do not open or control a local browser for Rotom administration or Available Sources synchronization.
- Apple Passwords/iCloud Passwords is the credential source of truth. Never record credential values or authentication material in these documents.
- RPD maintenance is governed exclusively by `00-Rotom-Change-Log.md`. Use the canonical command **`Update the RPD`** and follow the maintenance contract there; do not duplicate that contract in this document.
- For Rotom work tied to an active **Linear** ticket, keep the ticket current during execution: record meaningful verified implementation milestones, material design or scope decisions, blockers, rollbacks, and intentional deferrals as they occur. Do not use Linear as a raw command transcript or duplicate routine diagnostics. When ChatGPT is actively assisting with the ticket and Linear access is available, make these meaningful updates during the work without requiring a separate request for each update. **The RPD remains the canonical verified as-built source of truth**; update it from the final verified state at ticket closeout.
- Generated packages and separate saved artifacts belong in the project's `Downloads/` folder. The project-managed `sources/` directory is read-only and is not a mechanism for updating Available Sources.
- The current operational RPD checkouts are verified at `/Users/jared/Documents/ChatGPT/rotom-project-documentation` on Mac and `/home/infra/documentation/rotom-project-documentation` on Rotom. Git operations use Jared's general GitHub SSH identity and run as `jared`; do not copy that private key into service accounts. Each checkout is an operational mirror/reference for Codex and Git use, not a replacement for the Available Sources membership boundary.
- The adopted routine Git interface is the universal `rpd` helper with subcommands `path`, `status`, `check`, `pull`, `diff`, `log [N]`, `commit "message"`, `push`, and `help`. The canonical source is repository support file `tooling/rpd`; the identical script is installed at `/usr/local/bin/rpd` on Mac and Rotom. It uses `git --no-pager`, does not provide reset/clean/force-push/automatic merge/rebase operations, and retains the guarded fast-forward/clean-tree/remote/branch/upstream checks. `commit` stages changed, deleted, and new non-ignored repository files and creates the supplied local commit; `push` publishes existing commits after safety checks.
- Repository resolution is portable rather than path-hard-coded in the helper: when invoked inside the expected RPD Git repository, `rpd` uses that checkout; otherwise it reads a single checkout path from `${XDG_CONFIG_HOME:-$HOME/.config}/rpd/repository`. `rpd path` prints the validated resolved checkout. The current fallback file contains `/Users/jared/Documents/ChatGPT/rotom-project-documentation` on Mac and `/home/infra/documentation/rotom-project-documentation` on Rotom. The configuration directory/file were installed mode `0700`/`0600`.
- RPD execution remains environment-specific under the canonical contract in document 00, but shared workflow instructions resolve the checkout with `rpd path`. ChatGPT on Mac delivers changed replacement files plus the complete RPD ZIP, tells Jared to copy the changed files into the Mac checkout returned by `rpd path` and manually replace the matching Available Sources copies, then tells Jared to run a relevant `rpd commit`, `rpd check`, and `rpd push`; ChatGPT performs no Git action itself. Codex on either Mac or Rotom resolves the local checkout with `rpd path`, edits it directly only when RPD maintenance is authorized, then completes the final state with a relevant `rpd commit`, `rpd check`, `rpd push`, and success verification. Available Sources replacement remains manual by Jared.
- The repository-root `AGENTS.md` is shared Git/Codex support metadata for RPD work and points to document 00 rather than duplicating the canonical maintenance contract. Mac global `~/.codex/AGENTS.md` contains only the command-line editor preference. Rotom `/home/jared/.codex/AGENTS.md` remains mode `0600` under the hardened `.codex` directory, contains Rotom machine/live-state and safety guidance, resolves RPD through `rpd path`, and delegates shared RPD behavior to repository `AGENTS.md` plus document 00. If the helper is unavailable or any guard fails, Codex must stop and report the problem rather than bypassing it with destructive or ad-hoc Git operations.

This policy supersedes the earlier browser-workflow and `00`–`08`-only documentation-scope decisions recorded in the change log. Fresh read-only Codex sessions on both Mac and Rotom verified the current layered discovery model: global machine instructions, repository `AGENTS.md`, and document 00 as the canonical RPD maintenance authority. The older top-level `/home/infra/documentation/AGENTS.md` and `README.md` were not re-inspected or modified during this task; their older state is preserved as historical evidence below. Preparing Rotom Project Documentation does not by itself deploy files to Rotom or change its services.

## 13. Codex Documentation Discovery and RPD Git Integration

### Current state — 2026-09-28

The RPD Git integration is now verified on both supported Codex machines. The Mac checkout is `/Users/jared/Documents/ChatGPT/rotom-project-documentation`; the Rotom checkout is `/home/infra/documentation/rotom-project-documentation`. Both use origin `git@github.com:jaredwines/rotom-project-documentation.git`, branch `main`, and upstream `origin/main`. The prior Mac `/Users/jared/Documents/ChatGPT/Rotom-Home-Server` and Rotom `/home/infra/documentation/rpd` paths remain historical evidence only.

Repository support metadata includes `.gitignore`, root `AGENTS.md`, and `tooling/rpd`. These support Git/Codex operation but are not Rotom Project Documentation members unless intentionally added to Available Sources. Available Sources remains the membership boundary for RPD itself.

Jared's general GitHub SSH identity is `~/.ssh/d_ed25519_github`; key contents and passphrases are never documented. GitHub operations for this workflow run as `jared`, not as `root` or a service account.

The instruction model is layered:

- **Mac global:** `~/.codex/AGENTS.md` contains only the command-line editor preference.
- **Rotom global:** `/home/jared/.codex/AGENTS.md` is the Jared-global Rotom machine entry point. The `.codex` directory remains hardened and `AGENTS.md` is mode `0600`; the file contains Rotom live-state/safety rules and requires `rpd path`, repository `AGENTS.md`, relevant RPD reading, live-state inspection/comparison, the smallest targeted change, and verification before changing Rotom.
- **Repository:** root `AGENTS.md` is shared by Git across Mac and Rotom. It identifies document 00 as the canonical RPD maintenance contract, document 01 as the starting index, requires `rpd` guardrails, keeps shared instructions independent of physical checkout paths, and states that repository support files do not become RPD members merely by being tracked.
- **Canonical policy:** `00-Rotom-Change-Log.md` remains the sole RPD maintenance contract.

Fresh read-only Codex sessions on both Mac and Rotom correctly identified the local machine, resolved the checkout with the supported helper, identified the applicable instruction files and document 00, and described the required pre-change live-state workflow without modifying files or system state.

### Universal `rpd` helper — verified 2026-09-28

The canonical helper source is tracked at `tooling/rpd` and the identical script is installed as `/usr/local/bin/rpd` on both Mac and Rotom. All verified copies use SHA-256:

`42866ab6dbb5c47f996289d751cb26767dc79c756638cbeb9851c9e45fbdbe7b`

The supported command namespace is:

- `rpd path` — print the validated resolved local RPD checkout.
- `rpd status` — local repository summary without a network check.
- `rpd check` — repository/integrity validation plus a fresh GitHub fetch and ahead/behind/divergence report.
- `rpd pull` — clean-tree, fast-forward-only GitHub-to-local update.
- `rpd diff` — non-paged unstaged/staged local differences.
- `rpd log [N]` — non-paged recent commit history, default 10.
- `rpd commit "message"` — stage all changed, deleted, and new non-ignored repository files and create a local commit only.
- `rpd push` — push existing clean local commits only after refusing behind/diverged state.
- `rpd help` — usage and safety summary.

Repository resolution is:

1. If the current directory is inside a Git working tree whose `origin` is the expected RPD remote, use that repository root.
2. Otherwise read the configured checkout path from `${XDG_CONFIG_HOME:-$HOME/.config}/rpd/repository`.
3. Validate the expected user, repository, origin, branch, upstream, access, and core RPD files before guarded operations.

The current fallback file contains `/Users/jared/Documents/ChatGPT/rotom-project-documentation` on Mac and `/home/infra/documentation/rotom-project-documentation` on Rotom. The config directory/file are mode `0700`/`0600`. This path file contains no credential material.

Verification evidence includes Mac syntax/path/status/check tests, current-directory and config-fallback resolution, rejection of an unrelated temporary Git repository in favor of the configured RPD checkout, commit `152fd74` adding shared `AGENTS.md` and `tooling/rpd`, successful push, and a clean `main == origin/main` check. Rotom then fast-forwarded to the same commit, installed the repository copy, passed `bash -n`, `rpd path`, `rpd status`, and `rpd check`, resolved correctly from `/srv`, and matched the Mac/repository SHA-256. The prior machine-specific helpers and their path-revalidation gaps are therefore superseded for current state, while the older dated records remain historical evidence.

The Mac emitted an earlier Git warning that a commit identity had been derived automatically from the local username/hostname. Explicit global `user.name` and `user.email` configuration was recommended before routine use of `rpd commit`; no confirming output for that separate Git-identity setting is recorded here. It does not invalidate the verified helper architecture.

When authoritative documents change, follow the canonical RPD maintenance contract in `00-Rotom-Change-Log.md`. Shared instructions resolve the checkout with `rpd path`; ChatGPT on Mac remains a replacement-file/ZIP producer rather than a Git publisher, while Codex on Mac or Rotom may edit the resolved checkout directly only when RPD maintenance is authorized. Available Sources replacement remains manual by Jared.

### Preserved pre-migration Codex discovery history

At the 2026-09-19 pre-migration checkpoint, `/home/infra/documentation/AGENTS.md` was recorded as the authoritative Rotom instruction file whose directory scope applied when Codex started in `/home/infra/documentation` or a descendant. The same checkpoint recorded `/home/infra/documentation/README.md` beside it and no numbered RPD files at that top level. Those statements remain historical evidence; the current verified Jared-global discovery path is the one documented above, and the legacy top-level files were not re-inspected in this task.

For the `game` service account, the active `/home/game/.codex/config.toml` project entry was updated from the historical `/home/game-server/docker/palworld-server-jared` path to `/home/game/docker/palworld-server-jared`. A temporary pre-edit config copy and migration-only `/tmp` manifests were removed after verification. Historical shell/Codex history, session, snapshot, and cache records were intentionally not rewritten merely to erase the old name.

## 14. Preserved Pre-Migration Verification Summary

JAR-6 completed the eight-service storage-GID migration and per-service UNAS mount rollout. A normal Rotom reboot verified all eight service GIDs, generated automounts, exact NFSv3 `sec=sys` sources, root GIDs/modes, intended-account writes, unrelated-account denial, Backup/Shared Drive, and qBittorrent's existing Media/Game fail-closed binds. Backup closeout also verified `/mnt` remains excluded from Restic and the existing backup timer/service contract is unchanged. UNAS-side persistence of the newly set share-root GID/mode metadata across a future UNAS/UniFi Drive restart/update remains deferred.

The backup protection deployment, Palworld account/home migration, Game GID migration, Prowlarr migration, central qBittorrentVPN migration, and Gamarr deployment all completed successfully. The current Palworld identity is `game` at UID/GID `995:5001`, home `/home/game`; both Palworld containers are healthy, use `/home/game/docker/...` as their Compose working/config paths and bind sources, and retain the verified Jared and Fran world identifiers. The 2026-09-19 stopped-state checksum comparison covered 234 `.sav` files and showed no differences after recreation. The active game-account Codex project path was updated, and migration-only temporary rollback/check files were removed after verification.

JAR-24 final snapshot `fbe1838e` at `2026-09-25 18:14:22 PDT` is the authoritative pre-migration recovery point. Standard `restic check` and `restic check --read-data` both passed; the full-data run read all 971 packs across 20 retained snapshots with no errors. A representative isolated restore recovered both Palworld `Level.sav` files, JAR-22/JAR-23 audit artifacts, and the staged Home Assistant/Zigbee/NPM databases; restored SQLite quick checks passed and the scratch tree was removed. At that pre-migration checkpoint the timer was enabled. The early JAR-31 VM state temporarily held the restored timer disabled/inactive; later on 2026-09-27 it was renamed to `rotom-restic-backup.timer` and deliberately recommissioned enabled/active. Glances no longer mounts the backup share. Persistence of backup-root mode `0700` across a future UNAS/Drive restart remains deferred. JAR-25 now provides a checksum-verified raw NVMe rollback image, but an actual bare-metal restore from that image and full end-to-end Palworld recovery rehearsals remain deferred.

The Homebridge container-name typo was corrected on 2026-09-19 from `homebrige` to `homebridge`. `/home/jared/rotom-manual-verify.sh` and an associated verification record were updated to the corrected name. Homepage required no edit because its Homebridge service configuration was already correct and it discovers Docker through the socket. A whole-home scan found no remaining `homebrige` references, and Homebridge started successfully with its cached accessories restored. The corrected Compose file and `/home/jared/rotom-manual-verify.sh` are within the normal `/home` Restic include scope. Snapshot `e886c55d` is the dated post-change recovery point for that 2026-09-21 stage; JAR-24 `fbe1838e` later superseded it as the final pre-migration Restic baseline.

## 15. Historical Consolidated Verification — 2026-09-19

- `unas-backup.timer` remains enabled/active; the latest inspected scheduled service exited `0/SUCCESS` and completed normal prune/index work.
- `restic check --read-data` completed successfully across the entire current repository, and a representative restore was performed and cleaned up.
- Arcane Prowlarr/qBittorrent project records are reconciled by JAR-21; unrelated stale historical records and absence of notification providers remain separately documented.
- Wake-on-LAN host capability/configuration is verified; end-to-end delivery is not.
- The historical same-day audit captured Docker/NFS/backup/Arcane/WOL state. JAR-21 later disabled the stale `casper-md5check.service`; final closeout reported no failed systemd units.
- Gamarr now has verified named HTTPS access through NPM at `gamarr.rotom.casa -> 192.168.1.69:6767`; its container health is image-provided rather than a Compose healthcheck.
- VPN credential rotation after prior chat exposure remains unverified. Do not print or compare credential values; confirmation must come from administrator/provider-side rotation history.

## 16. Outstanding / Needs Verification

Phase B virtualization is accepted, guest Restic automation has been recommissioned under the normalized names, Proxmox host-config Restic is commissioned at `04:00`, and JAR-67 native Proxmox VZDump scheduling remains active. Remaining maintenance/automation items are deliberately separated by state:

- **Verified / commissioned:** guest `rotom-restic-backup.timer` is enabled/active on the daily `03:00` schedule; JAR-47 snapshot `aedd8e57` completed after v2 application-aware staging, and `restic check` passed 21/21 snapshots.
- **Needs Verification — first post-rename Restic scheduled run:** the timer has a populated next trigger, but the first timer-triggered run under `rotom-restic-backup.timer` has not yet been observed. Manual execution of the same privileged implementation is verified.
- **Verified / commissioned:** PVE host-config Restic manual/service execution, rename, snapshot cleanup, and repository integrity are verified; `pve-restic-backup.timer` is enabled/active for daily `04:00`, and current surviving snapshot is `35d4b0c2`.
- **Needs Verification — future unattended PVE Restic run:** a future timer-triggered execution under final `pve-restic-backup.timer` may be observed for operational evidence.
- **Needs Verification — future scheduler-triggered VZDump run:** `rotom-vm-daily` is enabled for `05:00`; manual VZDump and the fresh verified archive are complete, while a future unattended scheduler-triggered run under the final name may be observed.
- **Retention decision:** current PVE storage contains exactly one fresh verified archive, `vzdump-qemu-100-2026_09_28-00_09_19.vma.zst`. It is unprotected and subject to normal `rotom-vm-daily` retention; protect it only if a permanent baseline is desired.
- **Intentional / commissioning decision:** `/usr/local/sbin/weekly-linux-update` exists, but `/etc/cron.d/weekly-linux-update` is absent. Decide separately whether to restore the historical Sunday 04:00 schedule.
- **Intentional:** Fran is not recreated; do not install her historical backup sudoers rule until that account is deliberately restored.
- **Needs Verification — live Downloader-NFS loss:** no runtime watchdog is documented for an already-running qBittorrent session. Read-only inspection of the current Compose/systemd definitions can confirm that static fact; actual behavior during sudden mount loss requires a separately planned non-destructive maintenance test.
- **Needs Verification — end-to-end WOL trigger:** persistent host-side WOL configuration and post-power-cycle recovery are verified, but the transcript does not capture a magic-packet sender invocation; one controlled off-state wake with the sender command/output captured is still required to prove the NUC actually powers on from WOL.
- **Needs Verification / separate stability track:** continue the Proxmox/ConBee/iGPU follow-up. Do not infer a single earlier-reset cause from the current RPD.

## 17. Proposed Improvements

No proposed automation becomes current through this restructure. Future monitoring, runtime storage-loss watchdogs, smarthome NAS automation, any new `web` NAS design, or other recurring tasks must remain explicitly Proposed until deployed and verified.

## 18. Historical / Stale / Disruptive Automation

These items are retained for risk context; each line states whether it is still current.

1. **Arcane registry history:** a pre-JAR-21 audit found stale Smart Hub/Smarthome records and incorrect Prowlarr/qBittorrent project paths. JAR-21 reconciled the active project paths; unrelated stale historical records remain technical debt. The current 2026-09-27 database reread verifies the active automation settings separately.
2. **Arcane update scope:** the restored database now freshly confirms automatic updates at midnight with no exclusions and auto-heal every 30 seconds with no exclusions, five-restart limit, and 30-minute window. These settings are current as of the 2026-09-27 read-only audit.
3. **Jared Wines:** intentionally undeployed; its Compose file remains `/home/web/docker/jaredwines.com/compose.yaml`. Starting it later is a separate administrative change.
4. **Sunday updater:** the reboot-capable script is restored, but the current VM has no `/etc/cron.d/weekly-linux-update`; the old Sunday 04:00 schedule is historical and intentionally not recommissioned.
5. **Docker logging:** the pre-migration host lacked daemon-wide limits; the current VM has daemon-wide `json-file` limits of `10m` × `3`, so the old “most logs unlimited” warning is retired for current state.
6. **Alerting:** JAR-73 high-signal delivery is verified; it intentionally excludes broad per-container alerting and public status pages.
7. **`casper-md5check.service`:** the failed-unit observation belongs to the retired Linux Mint host. Current JAR-31/JAR-33 VM acceptance ended with zero failed systemd units; do not treat `casper-md5check` as a current fault.
8. **Recovery testing:** Restic full-pack readability and representative restores are verified; a full-system disaster-recovery rehearsal and full end-to-end Palworld server restore remain untested.

## 19. Historical Maintenance Verification — 2026-09-15

The scheduled backup started at 03:01:19 PDT and completed successfully. The read-only repository listing then showed 17 snapshots; the latest was afc91edf from 03:01:20 PDT. Docker, containerd, the backup timer, and the backup-status API remained enabled and active. The only failed unit observed remained casper-md5check.service.

## 20. Related Documentation

- `05-Backup-and-Restore.md` — canonical Restic repository, scope, retention, verification, and restore detail.
- `02-Docker-Services.md` — canonical Docker service deployment inventory.
- `07-Users-and-Permissions.md` — canonical account-switching, privilege, and access model.
- `00-Rotom-Change-Log.md` — documentation and server change history.
