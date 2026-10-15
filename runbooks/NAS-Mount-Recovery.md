# Runbook: Diagnose and Recover a Rotom NAS Mount

**Document role:** Operational pilot runbook  
**Status:** Verified for diagnosis, controlled automount repair, and current recovery-helper behavior
**Applies to:** The five current NFS systemd-automount contracts on the Rotom VM  
**Risk:** Read-only diagnosis; moderate targeted mount and eligible-container recovery  
**Authorization:** Diagnosis is read-only. Starting an existing configured mount and running the installed targeted recovery helper change live Rotom state. Editing `fstab`, changing NAS exports/permissions, unmounting live storage, or modifying data requires separate scope.  
**Last live verification:** 2026-10-15 — current fstab contracts, active NFSv3 mounts, systemd unit, and installed helper inspected  
**Canonical references:** [04-NAS-and-Storage.md](../04-NAS-and-Storage.md), [02-Docker-Services.md](../02-Docker-Services.md), [06-Maintenance-and-Automation.md](../06-Maintenance-and-Automation.md), [07-Users-and-Permissions.md](../07-Users-and-Permissions.md)  
**Maintenance contract:** [00-Rotom-Change-Log.md](../00-Rotom-Change-Log.md)

## 1. Outcome

Restore one existing Rotom NFS contract to its documented source without editing storage configuration or writing test data. When the Libraries mount caused eligible Docker startup failures, run the installed helper so it recovers only covered containers with a recorded NAS/mount failure signature.

## 2. Use this runbook when

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

## 3. Impact and authorization

Read-only commands do not intentionally change configuration. Accessing an automounted path or starting its `.mount` unit activates the existing NFS connection and is a live-state change, although it does not alter the mount contract.

The installed `rotom-nas-docker-recovery.service` validates only `/mnt/nas-libraries`, `library/`, and `paperless/`. It then considers only `qbittorrentvpn`, `radarr`, `sonarr`, `jellyfin`, `romm`, `paperless`, and `nginx-proxy-manager`. It starts a stopped container only when its restart policy is `unless-stopped` or `always` and its recorded error matches a NAS/mount startup failure. It leaves running, intentionally stopped, absent, and unrelated containers unchanged.

## 4. Canonical current-state inputs

| Mount | systemd mount unit | Current role |
|---|---|---|
| `/mnt/nas-game-servers` | `mnt-nas\x2dgame\x2dservers.mount` | Empty Game Servers service boundary; Palworld state is VM-local |
| `/mnt/nas-smart-home` | `mnt-nas\x2dsmart\x2dhome.mount` | Empty Smart Home service boundary; application state is VM-local |
| `/mnt/nas-rotom-restic-backup` | `mnt-nas\x2drotom\x2drestic\x2dbackup.mount` | Rotom Restic repository storage |
| `/mnt/nas-libraries` | `mnt-nas\x2dlibraries.mount` | Libraries, torrents, games, and Paperless media |
| `/mnt/nas-hosted-websites` | `mnt-nas\x2dhosted\x2dwebsites.mount` | Hosted Websites service storage boundary; current Aloha content is VM-local |

All five are NFS systemd automount contracts from NAS `192.168.1.70`. Exact export paths and ownership remain canonical in document 04 and live `findmnt --fstab` output.

At the initial 2026-10-15 validation point, the Libraries NFS filesystem was active and healthy but `mnt-nas\x2dlibraries.automount` was inactive with boot result `mount-start-limit-hit`. A controlled repair then restored both unit layers, confirmed the exact source, restarted the seven dependent containers, and verified their services. Treat an active mount with a failed automount as working storage with degraded future recovery posture, not as full success.

## 5. Read-only preflight

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

## 6. Stop conditions

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

## 7. Procedure

### Recover one configured mount

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

**Read-only; accesses the active mount**

```bash
sudo test -d /mnt/nas-libraries/library
sudo test -d /mnt/nas-libraries/paperless
sudo stat -c '%u:%g %a %n' \
  /mnt/nas-libraries \
  /mnt/nas-libraries/library \
  /mnt/nas-libraries/paperless
```

### Recover eligible Libraries-backed containers

Run the installed helper only after the Libraries mount and required directories pass. Because the oneshot unit uses `RemainAfterExit=yes`, use `restart` to execute a fresh check.

**Changes Rotom**

```bash
sudo systemctl restart rotom-nas-docker-recovery.service
```

The helper may start only the covered containers that meet its restart-policy and recorded-error gates. It must not start intentionally stopped containers without a qualifying NAS error.

## 8. Verification

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

## 9. Rollback or recovery

Starting an existing mount has no configuration rollback. Do not unmount it automatically after recovery, especially when consumers may have resumed.

If the mount is wrong, read-only, unstable, or backed by an unexpected source, stop consumers safely under a separately scoped incident procedure before any unmount. Preserve journal and `findmnt` evidence. Never delete files from the local mountpoint or NAS to “clear” a mount failure.

If the helper starts a container that then fails application verification, leave the mount intact and troubleshoot that container. Do not stop all Docker workloads or change NAS data as a blanket rollback.

## 10. Troubleshooting branches

- **Automount active, NFS inactive:** start the exact `.mount` unit, then verify its source.
- **Mount unit reports `Network is unreachable`:** diagnose NAS/LAN reachability; do not edit the export or `fstab` first.
- **Mount active, required Libraries directory missing:** stop. This is a storage-layout or wrong-source incident.
- **Container stopped with a qualifying mount error:** run the targeted helper after mount verification.
- **Container stopped without a mount error:** leave it stopped and use its service runbook.
- **Backup mount unavailable:** do not run the Libraries helper; use the backup runbook after restoring the exact backup mount.
- **Unexpected local files beneath a mountpoint:** stop and preserve them. Do not mount over, move, merge, or delete them without a separate reconciliation plan.

## 11. Evidence to retain

Record the exact target, configured source, observed source, mount unit state, error text, consumer state before and after, helper result, and application verification. Do not record NAS credentials or protected application inputs.

## 12. Related documentation

- [04-NAS and Storage](../04-NAS-and-Storage.md)
- [02-Docker Services](../02-Docker-Services.md)
- [06-Maintenance and Automation](../06-Maintenance-and-Automation.md)
- [Rotom Restic Backup](Rotom-Restic-Backup.md)

## 13. Maintenance notes

Revalidate this runbook whenever a mount/export, systemd unit, required directory, recovery-helper target list, container restart policy, or NAS-dependent service changes.

