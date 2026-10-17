# 04 - Rotom NAS and Storage Architecture

**Documentation set:** Rotom Runbook
**Document role:** Canonical source for NAS exports, NFS mounts, storage layout, automount behavior, and storage contracts
**Hosts:** PVE hypervisor `pve`, Debian VM `rotom`, and UniFi UNAS 2  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-15 — Prepare Rotom Runbook identity and migration gates
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `05-Backup-and-Restore.md`, `07-Users-and-Permissions.md`, `08-Rotom-Directory-Tree.txt`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1. Current Storage and Recovery Policy**, **2. Libraries Storage Contract**, and **3. PVE Backup Storage**.
- Treat recovery-critical historical artifacts as evidence, not as current mount or ownership instructions.
- Use **7. Recovery-Critical Historical Artifacts** only after reconciling it with the current five-mount contract, current identity data in document 07, and read-only live evidence.
- Review **Outstanding / Needs Verification** before changing NAS metadata, exports, automounts, or protection assumptions.
- This file owns NAS/NFS and storage contracts; backup retention and restore proof are canonical in document 05, while account and ACL policy are canonical in document 07.

## Start by Task or Symptom

- For an unavailable configured NFS path, inactive generated mount/automount, exact-source mismatch, or NAS-backed application dependency failure, use the procedure immediately below.
- For timer or retry-service scheduling behavior, follow [document 06](06-Maintenance-and-Automation.md#investigate-a-failed-timer-or-automation).
- For repository or restore work after storage is ready, follow [document 05](05-Backup-and-Restore.md#start-by-task-or-symptom).

## Diagnose and Recover a Configured NAS Mount

### 1. Outcome

Restore one existing Rotom NFS contract to its documented source without editing storage configuration or writing test data. When the Libraries mount caused eligible Docker startup failures, run the installed helper so it recovers only covered containers with a recorded NAS/mount failure signature.

### 2. Use this runbook when

- a current `/mnt/nas-*` path is not an active NFS mount;
- a systemd `.mount` unit failed during boot or transient NAS/network unavailability;
- a covered Libraries-backed container is stopped with a NAS mount startup error;
- an automount exists but the underlying NFS mount is not active when required.

Do **not** use this runbook to:

- add, remove, or rename an export or mount;
- change NAS ownership, permissions, ACLs, snapshots, or share policy;
- repair PVE backup mounts;
- unmount storage used by running containers;
- create files as a write test;
- restart Docker globally.

### 3. Impact and authorization

Read-only commands do not intentionally change configuration. Accessing an automounted path or starting its `.mount` unit activates the existing NFS connection and is a live-state change, although it does not alter the mount contract.

The installed `rotom-nas-docker-recovery.service` validates only `/mnt/nas-libraries`, `library/`, and `paperless/`. It then considers only `qbittorrentvpn`, `radarr`, `sonarr`, `jellyfin`, `romm`, `paperless`, and `nginx-proxy-manager`. It starts a stopped container only when its restart policy is `unless-stopped` or `always` and its recorded error matches a NAS/mount startup failure. It leaves running, intentionally stopped, absent, and unrelated containers unchanged.

### 4. Canonical current-state inputs

| Mount | systemd mount unit | Current role |
|---|---|---|
| `/mnt/nas-game-servers` | `mnt-nas\x2dgame\x2dservers.mount` | Empty Game Servers service boundary; Palworld state is VM-local |
| `/mnt/nas-smart-home` | `mnt-nas\x2dsmart\x2dhome.mount` | Empty Smart Home service boundary; application state is VM-local |
| `/mnt/nas-rotom-restic-backup` | `mnt-nas\x2drotom\x2drestic\x2dbackup.mount` | Rotom Restic repository storage |
| `/mnt/nas-libraries` | `mnt-nas\x2dlibraries.mount` | Libraries, torrents, games, and Paperless media |
| `/mnt/nas-hosted-websites` | `mnt-nas\x2dhosted\x2dwebsites.mount` | Hosted Websites service storage boundary; current Aloha content is VM-local |

All five are NFS systemd automount contracts from NAS `192.168.1.70`. Exact export paths and ownership remain canonical in document 04 and live `findmnt --fstab` output.

At the initial 2026-10-15 validation point, the Libraries NFS filesystem was active and healthy but `mnt-nas\x2dlibraries.automount` was inactive with boot result `mount-start-limit-hit`. A controlled repair restored both unit layers, confirmed the exact source, restarted the seven dependent containers, and verified their services. A same-day full reboot then proved the hardened recovery path: early Docker and first-helper attempts briefly failed before NAS reachability, while the helper's configured retry about 30 seconds later re-armed the automount, mounted the exact source, recovered eligible services, and completed successfully. Both generated Libraries units remained active and no systemd units were failed. An active mount with an inactive or failed automount is still degraded and requires targeted recovery.

### 5. Read-only preflight

Do not begin by listing every mount path; that would activate all configured automounts. Inspect configuration and current activation first.

**Read-only**

```bash
findmnt --fstab -t nfs,nfs4 -o TARGET,SOURCE,FSTYPE,OPTIONS
findmnt -rn -t nfs,nfs4 -o TARGET,SOURCE,FSTYPE,OPTIONS
systemctl list-units --type=automount --all --no-pager
systemctl --failed --no-pager
sudo docker ps -a \
  --format 'table {{.Names}}\t{{.Status}}'
```

For one exact target—Libraries in this example—inspect its units and consumers:

**Read-only**

```bash
systemctl status 'mnt-nas\x2dlibraries.automount' \
  'mnt-nas\x2dlibraries.mount' \
  --no-pager
sudo journalctl -u 'mnt-nas\x2dlibraries.mount' -n 100 --no-pager
findmnt -M /mnt/nas-libraries -o TARGET,SOURCE,FSTYPE,OPTIONS
sudo docker ps -a \
  --filter name='qbittorrentvpn' \
  --filter name='radarr' \
  --filter name='sonarr' \
  --filter name='jellyfin' \
  --filter name='romm' \
  --filter name='paperless' \
  --filter name='nginx-proxy-manager' \
  --format 'table {{.Names}}\t{{.Status}}'
```

If `findmnt -M` reports only an `autofs` layer or no NFS filesystem, the NFS mount is not yet active. Do not treat the visible local mountpoint directory as proof that NAS data is available.

### 6. Stop conditions

Stop before recovery if:

- the target is not one of the five current contracts;
- `findmnt --fstab` shows a different source or options from document 04;
- the target is mounted from an unexpected source or with unexpected read/write semantics;
- NAS address `192.168.1.70` is unreachable because of a broader network incident;
- an unmount would be required while a container, backup, or process uses the filesystem;
- local data may have been written beneath an inactive mountpoint;
- mount recovery requires changing `fstab`, export policy, ownership, permissions, or ACLs;
- a container is stopped without a NAS/mount failure signature;
- Restic or another writer is active against the affected filesystem.

### 7. Procedure

#### Recover one configured mount

If both the Libraries automount and NFS mount are inactive, reset only their failed state, arm the existing automount, and start the exact configured mount.

**Changes Rotom**

```bash
sudo systemctl reset-failed \
  'mnt-nas\x2dlibraries.automount' \
  'mnt-nas\x2dlibraries.mount'
sudo systemctl start 'mnt-nas\x2dlibraries.automount'
sudo systemctl start 'mnt-nas\x2dlibraries.mount'
```

If the NFS mount is already active but its automount is failed or inactive, do not unmount the working filesystem. Reset and start only the automount:

**Changes Rotom**

```bash
sudo systemctl reset-failed 'mnt-nas\x2dlibraries.automount'
sudo systemctl start 'mnt-nas\x2dlibraries.automount'
```

If systemd refuses to arm the automount while the NFS mount is active, stop. Do not unmount a filesystem with running consumers merely to clear the unit state.

**Read-only**

```bash
findmnt -M /mnt/nas-libraries -t nfs,nfs4 \
  -o TARGET,SOURCE,FSTYPE,OPTIONS
```

The target must resolve to the exact Libraries NFS source from `findmnt --fstab`. If it does not, stop.

Verify required directories without creating anything:

**Read-only; may activate an existing automount**

```bash
sudo test -d /mnt/nas-libraries/library
sudo test -d /mnt/nas-libraries/paperless
sudo stat -c '%u:%g %a %n' \
  /mnt/nas-libraries \
  /mnt/nas-libraries/library \
  /mnt/nas-libraries/paperless
```

#### Recover eligible Libraries-backed containers

Run the installed helper only after the Libraries mount and required directories pass. Because the oneshot unit uses `RemainAfterExit=yes`, use `restart` to execute a fresh check.

**Changes Rotom**

```bash
sudo systemctl restart rotom-nas-docker-recovery.service
```

The helper may start only the covered containers that meet its restart-policy and recorded-error gates. It must not start intentionally stopped containers without a qualifying NAS error.

### 8. Verification

**Read-only**

```bash
findmnt -M /mnt/nas-libraries -t nfs,nfs4 \
  -o TARGET,SOURCE,FSTYPE,OPTIONS
systemctl is-active 'mnt-nas\x2dlibraries.automount' \
  'mnt-nas\x2dlibraries.mount' \
  rotom-nas-docker-recovery.service
sudo journalctl -u rotom-nas-docker-recovery.service -n 120 --no-pager
sudo docker ps -a \
  --filter name='qbittorrentvpn' \
  --filter name='radarr' \
  --filter name='sonarr' \
  --filter name='jellyfin' \
  --filter name='romm' \
  --filter name='paperless' \
  --filter name='nginx-proxy-manager' \
  --format 'table {{.Names}}\t{{.Status}}'
systemctl --failed --no-pager
```

Success requires the exact NFS source and its automount to be active, the helper to complete successfully when it was needed, already-running containers to remain running, and only eligible NAS-failed containers to be recovered. Application-specific health and HTTP checks from documents 02/03 must also pass for every container the helper starts. An active mount with an inactive or failed automount is usable but remains degraded and is not complete recovery.

For a non-Libraries mount, success is the exact NFS source plus the service-specific verification from its canonical reference. Do not run the Libraries recovery helper merely because another current mount was activated.

### 9. Rollback or recovery

Starting an existing mount has no configuration rollback. Do not unmount it automatically after recovery, especially when consumers may have resumed.

If the mount is wrong, read-only, unstable, or backed by an unexpected source, stop consumers safely under a separately scoped incident procedure before any unmount. Preserve journal and `findmnt` evidence. Never delete files from the local mountpoint or NAS to “clear” a mount failure.

If the helper starts a container that then fails application verification, leave the mount intact and troubleshoot that container. Do not stop all Docker workloads or change NAS data as a blanket rollback.

### 10. Troubleshooting branches

- **Automount active, NFS inactive:** start the exact `.mount` unit, then verify its source.
- **Mount unit reports `Network is unreachable`:** diagnose NAS/LAN reachability; do not edit the export or `fstab` first.
- **Mount active, required Libraries directory missing:** stop. This is a storage-layout or wrong-source incident.
- **Container stopped with a qualifying mount error:** run the targeted helper after mount verification.
- **Container stopped without a mount error:** leave it stopped and use its service runbook.
- **Backup mount unavailable:** do not run the Libraries helper; use the backup runbook after restoring the exact backup mount.
- **Unexpected local files beneath a mountpoint:** stop and preserve them. Do not mount over, move, merge, or delete them without a separate reconciliation plan.

### 11. Evidence to retain

Record the exact target, configured source, observed source, mount unit state, error text, consumer state before and after, helper result, and application verification. Do not record NAS credentials or protected application inputs.

### 12. Related documentation

- [04-NAS and Storage](04-NAS-and-Storage.md)
- [02-Docker Services](02-Docker-Services.md)
- [06-Maintenance and Automation](06-Maintenance-and-Automation.md)
- [Rotom Restic Backup](05-Backup-and-Restore.md#run-and-verify-rotom-restic-backup)

### 13. Maintenance notes

Revalidate this runbook whenever a mount/export, systemd unit, required directory, recovery-helper target list, container restart policy, or NAS-dependent service changes.


## 1. Current Storage and Recovery Policy

**Evidence date:** 2026-10-15.

Rotom uses five current guest NFSv3 systemd automounts:

| NAS export | Guest mount | Root contract | Use |
|---|---|---|---|
| `Game_Servers/.data` | `/mnt/nas-game-servers` | `988:5001`, setgid service boundary | Empty boundary; Palworld state is VM-local |
| `Smart_Home/.data` | `/mnt/nas-smart-home` | `988:5003`, setgid service boundary | Empty boundary; Smart Home state is VM-local |
| `Libraries/.data` | `/mnt/nas-libraries` | `988:5000`; current descendants use Libraries identity | Authoritative Libraries payload and Paperless media |
| `Hosted_Websites/.data` | `/mnt/nas-hosted-websites` | `988:5004`, setgid service boundary | Empty boundary; website content is VM-local |
| `Rotom_Restic_Backup/.data` | `/mnt/nas-rotom-restic-backup` | root-protected repository boundary | Rotom Restic Backup |

The fstab options are the standard NFS systemd-automount pattern with `_netdev`, `nofail`, `x-systemd.automount`, and `x-systemd.mount-timeout=30s`. Verify the exact source and live options rather than reconstructing an entry from this summary.

All five were observed mounted read/write during the cleanup audit. Accessing an automount can activate it; use the command labels in the runbook above.

## 2. Libraries Storage Contract

`/mnt/nas-libraries` is the sole active Libraries guest contract. Its required current paths include:

- `library/` for Jellyfin, Radarr, Sonarr, RomM, and NPM's controlled read-only use.
- `torrents/` plus the qBittorrent readiness sentinel.
- `paperless/` for authoritative Paperless media.

Compose uses non-creating binds where a missing source would risk local shadow directories. The recovery helper must fail closed when the exact NFS filesystem, automount state, required directories, ownership, or sentinel is wrong.

At boot, Docker may request Libraries before the NAS is reachable. The enabled recovery helper resets failed generated units, arms `mnt-nas\x2dlibraries.automount`, mounts the exact export, verifies it, and conditionally recovers eligible containers. The verified reboot path required the helper's configured retry approximately thirty seconds later.

## 3. PVE Backup Storage

PVE uses two backup-specific NAS contracts:

- `PVE_Restic_Backup/.data` at `/mnt/nas-pve-restic-backup`, containing repository `pve-restic-backup`.
- `Rotom_VM_Backup/.data` as PVE storage `nas-rotom-vm-backup` at `/mnt/pve/nas-rotom-vm-backup`, with content type `backup`.

PVE does not mount Rotom guest application shares. Refer to document 05 for schedules, retention, verification, and restore procedures.

## 4. Numeric Identity and NAS Access

NFSv3 `sec=sys` access depends on numeric UID/GID and the NAS export policy. Interpret ownership numerically; a local username displayed for UID or GID `988` does not establish application ownership.

Current service storage GIDs are:

| Domain | UID:GID |
|---|---:|
| Platform | `997:5002` |
| Libraries | `127:5000` |
| Hosted Websites | `902:5004` |
| Game Servers | `995:5001` |
| Smart Home | `126:5003` |

Ordinary administrators should not be added to service storage groups merely for convenience. Use the documented service identity or authorized root access for service-specific administration.

## 5. Protection Boundaries

Rotom Restic Backup excludes `/mnt`; NAS-resident Libraries data therefore requires NAS-side protection. Jared confirmed Libraries snapshot retention and an isolated restore outcome, but exact NAS-side settings and selected recovery-point evidence were not supplied in the documentation task. Revalidate those details before depending on a specific snapshot.

The Game Servers, Smart Home, and Hosted Websites shares are empty service boundaries, not backup copies of their VM-local application state.

Do not manually reorganize or delete Restic repository internals. Do not change NAS exports, permissions, snapshot policy, or data during a guest-mount recovery unless separately authorized.

## 6. Retired Guest Storage Contracts

The following guest paths are retired and absent: `/mnt/nas-media`, `/mnt/nas-library`, `/mnt/nas-downloaders`, `/mnt/nas-documents`, `/mnt/nas-filesync`, `/mnt/nas-web`, `/mnt/nas-gameserver`, `/mnt/nas-smarthome`, `/mnt/nas-customapps`, `/mnt/nas-apps`, `/mnt/nas-auth`, and `/mnt/nas-shared-drive`.

Guest retirement does not prove NAS-side export or data deletion. The old Media, singular Library, Documents, Filesync, Web, Gameserver, Smarthome, Customapps, and Shared Drive names must not be recreated without current NAS-side evidence and explicit authorization.

The former Downloader read-only bindfs compatibility views are retired. No current container depends on them.

## 7. Recovery-Critical Historical Artifacts

The checksum-verified pre-virtualization raw NVMe image is recorded at:

`/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25/rotom-nvme-live-pre-proxmox-2026-09-25.img`

The guest Shared Drive mount is retired, so that path is a historical NAS location rather than a currently accessible guest path. Any access, retention change, or restore requires a separately authorized NAS access method. Preserve the image's recorded size, checksum, partition inspection, and crash-consistent limitation in document 05 and the protected change log.

Historical service-root, mount, export, and directory captures remain available through Git history and dated change-log evidence. They must be reconciled with the current five-contract table before any reconstruction.

## 8. Outstanding / Needs Verification

- Exact NAS snapshot schedules and retention for current Libraries data.
- NAS-side disposition of guest-retired exports and their data.
- Root-only pre-change recovery-helper copy retention.
- Incident-specific selection and verification of a usable recovery point.

## 9. Related Documentation

- [02 — Docker Services](02-Docker-Services.md)
- [03 — Network and Domains](03-Network-and-Domains.md)
- [05 — Backup and Restore](05-Backup-and-Restore.md)
- [07 — Users and Permissions](07-Users-and-Permissions.md)

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate after an export, fstab entry, mount option, automount unit, ownership, ACL, sentinel, bind source, recovery-helper rule, snapshot policy, repository path, or storage dependency changes.
