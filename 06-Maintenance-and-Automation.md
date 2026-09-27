# 06 - Maintenance and Automation

**Documentation set:** Rotom Project Documentation  
**Document role:** Canonical source for scheduled/routine maintenance, automation, monitoring behavior, and operational administration workflow  
**Hosts:** Proxmox hypervisor `proxmox` and Debian VM `rotom`  
**Baseline verified:** historical workload evidence through 2026-09-25; Phase B host/VM foundation plus live automation refresh verified 2026-09-27  
**Documentation updated:** 2026-09-27 — RPD maintenance governance moved exclusively to `00-Rotom-Change-Log.md`
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
Documentation updated: **2026-09-27** through the backup-control/recovery-point refresh after JAR-67; JAR-31 application/recovery restoration and reboot acceptance plus the JAR-30 Docker/containerd/logging baseline remain current; JAR-22 and earlier migration evidence below remains preserved. The prior update reconciled the `infra` / `smarthome` identity-path migration and six new service-account skeletons. The earlier Available Sources and operational-history notes remain preserved. The verified Prowlarr ownership/path migration, completed central qBittorrentVPN/Gamarr deployment, verified Homebridge container-name correction, and other operational findings below retain their original evidence dates. Automation findings retain their original evidence dates except where explicitly refreshed. The September 18 backup-access protection work changed the UNAS backup share-root mode, removed Glances' direct backup bind, recreated the Homepage/Glances Compose project, verified both manual backup launch paths, and rechecked the backup timer/service state. Later the same day, the Palworld service account was renamed from `game-server` to `game` with UID/GID `995:985` preserved and its home moved to `/home/game`; both Palworld projects were recreated and verified healthy from the new path. On 2026-09-19 the separate Game storage work changed the current `game` primary GID to `5001`, updated both Palworld `PGID` values to `5001`, and commissioned `/mnt/nas-game` as an active systemd automount. Later on 2026-09-19 Prowlarr moved from `/home/media/docker/prowlarr` to the then-current `/home/rotom/docker/prowlarr`, changed runtime identity to `997:986`, and became a verified Arcane-managed project with auto-update enabled; the 2026-09-21 rename subsequently moved it to `/home/infra/docker/prowlarr`. Central qBittorrentVPN likewise moved in 2026-09-19 to the then-current `/home/rotom/docker/qbittorrentvpn` at `997:5000`, then to `/home/infra/docker/qbittorrentvpn` during the account rename; JAR-21 later moved it to `/home/downloads/docker/qbittorrentvpn` at `901:5005`, centralized torrents on Downloads, and retained verified VPN/kill-switch behavior; Gamarr was added under `/home/game/docker/gamarr` at `995:5001`. Backup scope, Restic repository format, retention policy, and timer schedule were not changed. JAR-6 later migrated Infra-through-Auth primary GIDs to `5002`–`5009`, commissioned eight per-service systemd automounts, and verified them plus qBittorrent/backup behavior through a normal Rotom reboot; those changes did not alter backup scope or Docker's global startup dependency model. Later on 2026-09-21, all ten operational service homes gained verified `nas` symlinks to their corresponding `/mnt/nas-*` mount. JAR-21 then added the two persistent read-only bindfs view services, Downloads-specific qBittorrent startup guard, Arcane symlink discovery, and final post-cleanup backup; these later facts are authoritative for the **final pre-migration** download stack. JAR-31 current `downloaders` state is authoritative in section 2.

## 2. Current Maintenance and Automation — 2026-09-27 refresh

The current guest has production workloads restored. Guest Restic automation is now recommissioned under the normalized `rotom-restic-backup` naming, while the Sunday reboot-capable updater remains deliberately unscheduled. Proxmox whole-VM automation remains native to PVE.

### Proxmox host

- Proxmox VE `9.2.20` runs kernel `7.0.14-19-pve`; the prior fallback kernel remains installed.
- PVE/Ceph use no-subscription repositories; BIOS/microcode and `i915.enable_dc=0` stability baseline remain as previously documented.
- ConBee II remains intentionally disconnected. No application Docker/Podman automation or application NFS mounts exist on the hypervisor. Dedicated PVE-only NFS storage `nas-rotom-proxmox-backup` remains at `/mnt/pve/nas-rotom-proxmox-backup`, `content backup` only, NFSv3, managed by Proxmox rather than `/etc/fstab`.
- Native PVE job `rotom-daily` remains enabled in `/etc/pve/jobs.cfg`: node `proxmox`, VMID 100, `05:00` daily, snapshot mode, zstd, `repeat-missed=0`, notes template `Scheduled Rotom VM backup - {{guestname}} VM {{vmid}} on {{node}}`, retention `keep-daily=7,keep-weekly=4,keep-monthly=6`.
- `/usr/local/bin/backup-proxmox-to-nas` is the manual human-facing helper. It runs `vzdump` for VMID 100 against `nas-rotom-proxmox-backup` with snapshot mode + zstd. It is **independent of** `rotom-daily`; the native PVE scheduler does not call this helper, and no Proxmox systemd backup timer/service was added.
- At the administrator's request, all older VMID 100 backup artifacts were deleted while leaving storage and `rotom-daily` intact. The sole current archive is `vzdump-qemu-100-2026_09_27-14_08_51.vma.zst` (`45,424,272,161` bytes), which passed full VMA verification with exit code `0`; VM 100 remained running. Final listing showed only that archive plus its `.log`, with no protection marker. The first unattended scheduler-triggered `05:00` run still has not been observed.

### Rotom VM

- Debian 13.7, qemu-guest-agent, NTP, weekly `fstrim.timer`, guest NFS automounts, Docker Engine `29.8.1`, containerd `2.3.6`, local `/var/lib/docker`, and daemon `json-file` limits remain current.
- The exact 16-container production set is restored; Jared Wines remains intentionally absent. Docker/containerd and the complete application set survived the JAR-31 acceptance reboot.
- `rotom-nas-docker-recovery.service` remains enabled and verified through the documented boot-time NFS race/retry.
- `rotom-downloads-media-ro.service` and `rotom-downloads-game-ro.service` remain active read-only compatibility views sourced from `/mnt/nas-downloaders`.
- Restic active names are now `/usr/local/bin/backup-restic-to-nas`, `/usr/local/sbin/rotom-restic-backup`, `/etc/sudoers.d/rotom-restic-backup`, `rotom-restic-backup.service`, `rotom-restic-backup.timer`, `/usr/local/sbin/rotom-restic-backup-status-api`, `rotom-restic-backup-status-api.service`, and `/etc/restic/nas-password`. Old active `unas-*`, `backup-to-unas`, `update-to-nas`, and `unas-password` names were verified absent; historical rollback copies retain their original names.
- `rotom-restic-backup-status-api.service` is enabled/active on TCP `8787`. Homepage continues to use `http://192.168.1.69:8787/backup-status`.
- `rotom-restic-backup.timer` is enabled/active. Its schedule remains `OnCalendar=*-*-* 03:00:00`, `Persistent=true`, `RandomizedDelaySec=10m`. Immediately after recommissioning the status API reported `schedule=Active`, `enabled=enabled`, `next_run=9/28/2026 3:07:23 AM`, `last_run=Unknown`, `last_result=success`.
- The normal privileged implementation performs `restic backup` then `restic forget --tag automatic --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune`. A verified manual `backup-restic-to-nas` run created `f666d63c` at `2026-09-27 13:30:28 PDT`, processed `17.410 GiB`, removed superseded same-day snapshot `74d3b0ca`, and pruned about `120.462 MiB`. Subsequent `restic check` passed `21/21` snapshots with no errors.
- `/usr/local/sbin/weekly-linux-update` remains restored and syntax-valid; `/etc/cron.d/weekly-linux-update` is deliberately absent. Fran's historical backup sudoers rule remains deliberately absent.
- Arcane remains healthy with the current automatic update/auto-heal settings documented below.

The preserved pre-migration automation inventory below remains the source for historical schedules/settings. Historical references to `unas-backup*` and `backup-to-nas` describe the retired/pre-normalization state and are not current operational names.

### Current guest Restic execution state — 2026-09-27

Current automation points only to `/mnt/nas-rotom-restic-backup`. `/usr/local/sbin/rotom-restic-backup` exports the repository path and checks the mountpoint; `rotom-restic-backup.service` declares `RequiresMountsFor=/mnt/nas-rotom-restic-backup`. The old `/mnt/nas-rotom-backup` NFS mount is unmounted and absent from current backup configuration; old `Rotom_Home_Server_Backup` is rollback-only.

The human command `/usr/local/bin/backup-restic-to-nas` contains only the sudo handoff to `/usr/local/sbin/rotom-restic-backup`. Jared's `/etc/sudoers.d/rotom-restic-backup` grants NOPASSWD only for that exact privileged implementation. The current protected credential file is `/etc/restic/nas-password`; its contents are never documented.

The timer is now commissioned: enabled/active with daily `03:00`, persistent catch-up, and up to ten minutes randomized delay. A manual execution is still a real backup + retention/prune operation; it is not a dry-run health check.

### Current Proxmox VZDump automation — 2026-09-27

PVE job `rotom-daily` remains the automatic scheduler for VMID `100`. It targets `nas-rotom-proxmox-backup`, runs daily at `05:00`, uses snapshot mode and zstd compression, has `repeat-missed=0`, and applies `keep-daily=7,keep-weekly=4,keep-monthly=6`.

Manual command `/usr/local/bin/backup-proxmox-to-nas` is intentionally separate. An operational lesson from commissioning: invoke the installed helper command itself; pasting its script body into an interactive shell executes its `exec vzdump` line in that shell and replaces the shell process. The interrupted test at `13:35:08` left no partial archive or VM lock; Proxmox retained only a log, later removed during the intentional VMID 100 cleanup. A subsequent helper run and the final clean-slate run both completed successfully; the final sole archive is the verified `14_08_51` file documented above.

## 2A. Preserved Pre-Migration Automation Overview

| Time or trigger | Automation | Verified behavior |
|---|---|---|
| Boot | Docker | Docker and containerd are enabled. Docker starts after networking and containerd. |
| Post-Docker boot / retry on failure | `rotom-nas-docker-recovery.service` | Verifies real Downloads/Media/Game NFS readiness, restores the two Downloads bindfs views, and recovers only stopped NAS-backed containers with recorded NAS/mount startup errors. Enabled; manual healthy-state validation passed; reboot validation deferred. |
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

All twelve fstab-backed NAS paths keep their existing automount model. JAR-21 additionally created two persistent bindfs units: `rotom-downloads-media-ro.service` and `rotom-downloads-game-ro.service`. They are enabled and active and expose `/mnt/nas-downloads` read-only at `/mnt/nas-downloads-media-ro` and `/mnt/nas-downloads-game-ro` with forced Media/Game identities for the Arr applications.

qBittorrent uses a service-specific Docker/Compose startup guard rather than a global Docker→NAS systemd dependency. The active `/mnt/nas-downloads/torrents` bind and Downloads sentinel bind use `create_host_path: false`; deliberate removal of the Downloads sentinel caused recreation to fail as required. This prevents local-directory fallback on recreation. A live runtime NFS-loss watchdog was not implemented.

No rc.local hook exists. The at scheduler is not installed. No per-user crontabs were found. The only observed user timer was launchpadlib-cache-clean for jared.


### JAR-23 NAS-backed Docker recovery hardening — 2026-09-25

After the September 23 boot race left the Downloads bindfs views and several NAS-backed containers stopped, JAR-23 added root-owned executable `/usr/local/sbin/rotom-nas-docker-recovery` and systemd unit `/etc/systemd/system/rotom-nas-docker-recovery.service`. The unit is enabled under `multi-user.target`, `Requires=docker.service`, runs after `docker.service` and `network-online.target`, and uses `Restart=on-failure` with a 30-second delay so a transiently unavailable NAS can be retried without making all Docker services depend on the NAS.

The helper explicitly starts/verifies `/mnt/nas-downloads`, `/mnt/nas-media`, and `/mnt/nas-game` as NFS, verifies the Downloads torrent tree and `.rotom-qbt-nas-ready` sentinel, starts/verifies `rotom-downloads-media-ro.service` and `rotom-downloads-game-ro.service`, tests traversal as `media` and `game`, then inspects qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin. Already-running containers are left untouched. A stopped container is started only when its restart policy is `unless-stopped`/`always` and its recorded Docker error matches a NAS/mount startup-failure signature; other stopped containers are left stopped. This protects intentionally stopped workloads from being indiscriminately started.

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

Arcane's host project root remains `/home/infra/docker`. Current discovery uses `/home/infra/docker/downloads -> /home/downloaders/docker`; `followProjectSymlinks=true` is freshly verified. Arcane persists Prowlarr and qBittorrentVPN at `/home/infra/docker/downloads/prowlarr` and `/home/infra/docker/downloads/qbittorrentvpn`, while those paths resolve to the Downloaders-owned canonical trees. Fresh project-state inspection reports Prowlarr `running`/`1 of 1` and qBittorrentVPN stale as `unknown`/`0 of 1`, last updated 2026-09-22. Docker independently verifies qBittorrentVPN running with `wg0` up, so Arcane's qBittorrent project status is stale metadata rather than runtime authority. Arcane does not manage Palworld under `/home/game/docker`, so Arcane updates do not overlap Palworld's updater.

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

Nginx Proxy Manager checks certificate renewal hourly for certificates expiring within 30 days. A direct TLS inspection verified the currently served `*.rotom.casa`/`rotom.casa` Let's Encrypt certificate is valid from 2026-09-11 through 2026-12-10. No recent renewal error was found; a real future renewal event has not yet been observed.

Cloudflare DDNS checks every minute and at container startup. Its inspected record was already current. It has no Docker health check.

## 8A. JAR-31 Homepage Monitoring Reconciliation — 2026-09-26 checkpoint

The final compatibility restore left Homepage healthy but exposed monitoring drift inherited from the pre-virtualization host. The remediation is now part of the current Phase B monitoring baseline:

- Homepage internal URL checks were failing with repeated `EAI_AGAIN` for `nginx.rotom.casa`, `arcane.rotom.casa`, `home-assistant.rotom.casa`, and `homebridge.rotom.casa`. Targeted `extra_hosts` entries in the Homepage service now resolve those names, plus `rotom.casa`, to the Rotom VM at `192.168.1.69`. All four affected HTTPS checks returned HTTP 200 afterward and no new `EAI_AGAIN` appeared during verification.
- The Rotom Backup Schedule widget was moved to `http://192.168.1.69:8787/backup-status`. At this JAR-31 checkpoint the restored timer was intentionally held and the API returned `schedule=INACTIVE`, `enabled=disabled`, `next_run=Unknown`, `last_run=Unknown`, and `last_result=success`. **This timer state is historical:** later on 2026-09-27 the controls were renamed and `rotom-restic-backup.timer` was deliberately recommissioned enabled/active; the current status API then reported `schedule=Active`, `enabled=enabled`, and `next_run=9/28/2026 3:07:23 AM`.
- Disk Usage changed from the obsolete bare-metal `disk:nvme0n1` metric to `disk:sda` after Glances `diskio` verified the VM devices and `/dev/sda2` root parent.
- The Debian VM exposes no physical CPU package-temperature sensor. Proxmox exposes `coretemp` `Package id 0`; it read 56.0 C during the verification. A minimal read-only `/usr/local/sbin/rotom-temp-api` with `/etc/systemd/system/rotom-temp-api.service` now runs on the Proxmox host and binds `192.168.1.68:8788`. Its `/api/4/sensors` response is intentionally Glances-compatible so Homepage can retain the original graph behavior rather than falling back to a non-charting custom API tile.
- Homepage CPU Temperature uses the Proxmox bridge as a Glances v4 widget with `metric: sensor:Package id 0`, `chart: true`, `refreshInterval: 1000`, and `pointsLimit: 15`. The separate `/temperature` endpoint remains available for a simple JSON health/readout check.

The temperature helper reads `/sys/class/hwmon` dynamically to locate `coretemp` / `Package id 0`; it does not expose credentials, does not change Proxmox application boundaries, and is separate from the Rotom VM's Glances container.

## 9. Preserved Pre-Migration Wake-on-LAN

The wired NIC `eno1` reports `Supports Wake-on: pumbg` and `Wake-on: g`, verifying magic-packet Wake-on-LAN is enabled at the host/NIC level. No scheduled Wake-on-LAN sender or custom WOL script was found. End-to-end delivery while Rotom is off/suspended remains untested.

## 9A. Current NAS/Docker Maintenance State — 2026-09-27 refresh

- Twelve guest fstab entries continue to generate `.automount`/`.mount` units with `_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s`.
- Current `/mnt/nas-downloaders` uses NAS `Downloader/.data`; the earlier JAR-29/JAR-30 `Downloads/.data` current-state claim is superseded by live fstab/generated-unit/findmnt/showmount evidence.
- Docker and containerd are enabled/active at boot. Docker runtime remains local; NAS-backed application recovery is handled by the targeted helper rather than a global Docker→NAS dependency.
- `rotom-nas-docker-recovery.service` is enabled under `multi-user.target`, requires Docker, starts after Docker/network-online, and retries on failure. The JAR-31 reboot proved the retry path: first Media mount attempt failed, the next invocation succeeded and recovered exactly the affected NAS-backed containers.
- The helper leaves already-running workloads untouched and recovered qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin after matching their prior NAS mount startup-failure state.
- Both bindfs compatibility services are active and expose `/mnt/nas-downloaders` read-only through `/mnt/nas-downloads-media-ro` and `/mnt/nas-downloads-game-ro`.
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
findmnt -T /mnt/nas-downloads-media-ro
stat -c '%A %a %u:%g %n' /mnt/nas-media /mnt/nas-rotom-restic-backup
sudo -iu media
```

Then run **on the Rotom VM as `media`** to inspect the effective service identity and current read-only Downloader compatibility view without changing files:

```bash
id
ls -ldn /mnt/nas-media /mnt/nas-media/library /mnt/nas-downloads-media-ro /mnt/nas-downloads-media-ro/torrents
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

- Use the native **macOS Terminal** app. The current verified administration entry points are `ssh rotom.casa` (or alias `ssh rotom`) for Debian user `jared`, and `ssh proxmox` (or `ssh proxmox.rotom.casa`) for Proxmox `root`. The Mac SSH config uses dedicated identity files `~/.ssh/id_ed25519_rotom` and `~/.ssh/id_ed25519_proxmox` respectively, with `IdentitiesOnly yes`, `AddKeysToAgent yes`, and `UseKeychain yes`. Key-only authentication has been forced and verified successfully for both targets. The Rotom public key is authorized in `/home/jared/.ssh/authorized_keys`; the Proxmox public key is authorized in `/root/.ssh/authorized_keys` alongside the preserved pre-existing RSA key. Password-authentication policy remains unchanged. Ghostty has been removed from the Mac.
- ChatGPT-controlled browser work is not part of the Rotom administration or documentation workflow. Do not open or control a local browser for Rotom administration or Available Sources synchronization.
- Apple Passwords/iCloud Passwords is the credential source of truth. Never record credential values or authentication material in these documents.
- RPD maintenance is governed exclusively by `00-Rotom-Change-Log.md`. Use the canonical command **`Update the RPD`** and follow the maintenance contract there; do not duplicate that contract in this document.
- For Rotom work tied to an active **Linear** ticket, keep the ticket current during execution: record meaningful verified implementation milestones, material design or scope decisions, blockers, rollbacks, and intentional deferrals as they occur. Do not use Linear as a raw command transcript or duplicate routine diagnostics. When ChatGPT is actively assisting with the ticket and Linear access is available, make these meaningful updates during the work without requiring a separate request for each update. **The RPD remains the canonical verified as-built source of truth**; update it from the final verified state at ticket closeout.
- Generated packages and separate saved artifacts belong in the project's `Downloads/` folder. The project-managed `sources/` directory is read-only and is not a mechanism for updating Available Sources.

This policy supersedes the earlier browser-workflow and `00`–`08`-only documentation-scope decisions recorded in the change log. It does not update `/home/infra/documentation/AGENTS.md`; that file was not inspected or modified by this documentation-only change. If its operating instructions still contain conflicting documentation guidance, update them only in a separate reviewed synchronization task. Preparing Rotom Project Documentation does not by itself deploy files to Rotom or change its services.

## 13. Preserved Pre-Migration Codex Documentation Discovery

`/home/infra/documentation/AGENTS.md` is the authoritative Rotom instruction file. Its directory scope applies automatically when Codex starts in `/home/infra/documentation` or a descendant. Starting Codex elsewhere does not make that directory-level file automatically applicable.

For `jared`, the supplied output confirms `/home/jared/.codex/AGENTS.md` exists as `jared:jared` mode `0600`. That user-level file provides the entry point for loading the authoritative Rotom instructions when Jared launches Codex outside `/home/infra/documentation`. `/home/jared/.codex` was also restricted to mode `0700`. Keep this user-level file short and refer to the authoritative files rather than maintaining a second full copy that can drift.

Codex configuration is per Linux account. A `jared` configuration does not configure `fran`, `rotom`, or another user. If Fran will run Codex, create and verify Fran's user-level instruction file in a separate task. Never copy Codex login caches, API keys, device codes, tokens, or authentication-file contents into Rotom documentation.

When authoritative documents change, follow the canonical RPD maintenance contract in `00-Rotom-Change-Log.md`; Jared performs any manual replacement of the Mac-folder and Available Sources copies. A fresh 2026-09-19 top-level listing of `/home/infra/documentation` confirms only `AGENTS.md` and `README.md` are deployed there; none of the current nine Rotom Project Documentation files are present at that location. Those server-side control files are not part of the current Available Sources set unless intentionally added there in the future.

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

Phase B virtualization is accepted, guest Restic automation has been recommissioned under the normalized names, and JAR-67 native Proxmox scheduling remains active. Remaining maintenance/automation items are deliberately separated by state:

- **Verified / commissioned:** guest `rotom-restic-backup.timer` is enabled/active on the daily `03:00` schedule; current verified snapshot is `f666d63c`, and post-run `restic check` passed 21/21 snapshots.
- **Needs Verification — first post-rename Restic scheduled run:** the timer has a populated next trigger, but the first timer-triggered run under `rotom-restic-backup.timer` has not yet been observed. Manual execution of the same privileged implementation is verified.
- **Needs Verification — first scheduler-triggered Proxmox run:** `rotom-daily` is enabled for `05:00`; manual/scheduled-equivalent VZDump paths are verified, but the first unattended scheduler-triggered run has not yet been observed.
- **Retention decision:** the sole current Proxmox archive `vzdump-qemu-100-2026_09_27-14_08_51.vma.zst` is verified but was not protected in the final observed listing, so it can eventually age out under routine retention.
- **Intentional / commissioning decision:** `/usr/local/sbin/weekly-linux-update` exists, but `/etc/cron.d/weekly-linux-update` is absent. Decide separately whether to restore the historical Sunday 04:00 schedule.
- **Intentional:** Fran is not recreated; do not install her historical backup sudoers rule until that account is deliberately restored.
- **Needs Verification — live Downloader-NFS loss:** no runtime watchdog is documented for an already-running qBittorrent session. Read-only inspection of the current Compose/systemd definitions can confirm that static fact; actual behavior during sudden mount loss requires a separately planned non-destructive maintenance test.
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
6. **Alerting:** disk/health monitoring exists, but no verified general alert threshold/delivery destination is recorded.
7. **`casper-md5check.service`:** the failed-unit observation belongs to the retired Linux Mint host. Current JAR-31/JAR-33 VM acceptance ended with zero failed systemd units; do not treat `casper-md5check` as a current fault.
8. **Recovery testing:** Restic full-pack readability and representative restores are verified; a full-system disaster-recovery rehearsal and full end-to-end Palworld server restore remain untested.

## 19. Historical Maintenance Verification — 2026-09-15

The scheduled backup started at 03:01:19 PDT and completed successfully. The read-only repository listing then showed 17 snapshots; the latest was afc91edf from 03:01:20 PDT. Docker, containerd, the backup timer, and the backup-status API remained enabled and active. The only failed unit observed remained casper-md5check.service.

## 20. Related Documentation

- `05-Backup-and-Restore.md` — canonical Restic repository, scope, retention, verification, and restore detail.
- `02-Docker-Services.md` — canonical Docker service deployment inventory.
- `07-Users-and-Permissions.md` — canonical account-switching, privilege, and access model.
- `00-Rotom-Change-Log.md` — documentation and server change history.