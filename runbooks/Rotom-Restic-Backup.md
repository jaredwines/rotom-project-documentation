# Runbook: Run and Verify Rotom Restic Backup

**Document role:** Operational pilot runbook  
**Status:** Verified against the current Rotom timer, service, launcher, worker, mount, repository path, and retention implementation  
**Applies to:** Rotom VM host-data Restic backup only  
**Risk:** Read-only inspection; moderate backup/repository write and policy-driven prune  
**Authorization:** Read-only checks are diagnostic. Starting the established backup is an ordinary backup operation, but it writes a snapshot and applies the installed retention/prune policy. Repository unlock, repair, deletion, credential changes, restores over live data, PVE backup, and whole-VM backup require their own scope.  
**Last live verification:** 2026-10-15 — timer active, latest scheduled service successful, exact unit/launcher/worker inspected, and backup NFS mount active  
**Canonical references:** [05-Backup-and-Restore.md](../05-Backup-and-Restore.md), [04-NAS-and-Storage.md](../04-NAS-and-Storage.md), [06-Maintenance-and-Automation.md](../06-Maintenance-and-Automation.md), [07-Users-and-Permissions.md](../07-Users-and-Permissions.md)  
**Maintenance contract:** [00-Rotom-Change-Log.md](../00-Rotom-Change-Log.md)

## 1. Outcome

Create one Rotom Restic host-data snapshot through the installed worker, apply its established `7 daily / 4 weekly / 12 monthly` automatic-snapshot retention and prune policy, and verify that the run completed successfully and produced a current snapshot. Source data and repository internals must not be manually edited.

## 2. Use this runbook when

- an authorized manual Rotom host backup is needed;
- the scheduled Rotom Restic Backup must be verified;
- the backup timer or service reports failure;
- a backup must be checked before maintenance.

Do **not** use this runbook for:

- PVE Restic Backup;
- Rotom VM VZDump backup;
- NAS-side snapshots;
- restoring over live application data;
- changing include/exclude files, credentials, retention, or repository location;
- deleting, moving, or reorganizing Restic repository internals.

## 3. Impact and authorization

The backup reads configured Rotom paths, refreshes restricted system-information files under `/var/backups/system-info`, creates transactionally consistent copies of the NPM and Home Assistant SQLite databases, writes a tagged Restic snapshot, then runs `forget --prune` under the installed retention policy. It can use significant disk, CPU, NAS I/O, and time.

The repository password file is `/etc/restic/restic-repository-password`. Never print, copy, paste, hash, or record its contents. Backups may contain credentials, certificates, application data, and personal information; do not expose snapshot contents in routine evidence.

## 4. Canonical current-state inputs

| Item | Current contract |
|---|---|
| Manual launcher | `/usr/local/bin/backup-restic-to-nas` |
| systemd service | `rotom-restic-backup.service` |
| timer | `rotom-restic-backup.timer`, daily around `03:00` with up to 10 minutes randomized delay |
| worker | `/usr/local/sbin/rotom-restic-backup` |
| mount | `/mnt/nas-rotom-restic-backup` |
| repository | `/mnt/nas-rotom-restic-backup/rotom-restic-backup` |
| retention | automatic snapshots: 7 daily, 4 weekly, 12 monthly, followed by prune |

Rotom Restic Backup protects configured VM-local host/application state. `/mnt` is excluded; NAS-resident Libraries/Paperless media and the repository itself are separate protection domains.

## 5. Read-only preflight

**Read-only**

```bash
systemctl is-enabled rotom-restic-backup.timer
systemctl is-active rotom-restic-backup.timer \
  rotom-restic-backup.service
systemctl list-timers rotom-restic-backup.timer --all --no-pager
systemctl status rotom-restic-backup.service --no-pager
sudo journalctl -u rotom-restic-backup.service -n 120 --no-pager
findmnt -M /mnt/nas-rotom-restic-backup -t nfs,nfs4 \
  -o TARGET,SOURCE,FSTYPE,OPTIONS
df -h / /var /mnt/nas-rotom-restic-backup
pgrep -af '(^|/)(restic|rotom-restic-backup)( |$)' || true
sudo stat -c '%U:%G %a %n' \
  /usr/local/bin/backup-restic-to-nas \
  /usr/local/sbin/rotom-restic-backup \
  /etc/restic/include.txt \
  /etc/restic/exclude.txt \
  /etc/restic/restic-repository-password
```

List snapshot metadata without printing secret contents:

**Read-only; credential-sensitive path usage**

```bash
sudo env \
  RESTIC_REPOSITORY=/mnt/nas-rotom-restic-backup/rotom-restic-backup \
  RESTIC_PASSWORD_FILE=/etc/restic/restic-repository-password \
  restic snapshots --host rotom --tag automatic --latest 3
```

Expected result is an enabled/active timer, inactive oneshot service between runs, exact active NFS mount, no concurrent Restic process, protected control files, and readable snapshot metadata.

## 6. Stop conditions

Stop before starting a backup if:

- another Restic process or `rotom-restic-backup.service` is active;
- the mount is absent, read-only, or from an unexpected source;
- the repository path or credential-file path differs from the current contract;
- `/`, `/var`, the NAS, or repository lacks safe working capacity;
- required SQLite source databases are missing or report integrity problems;
- include/exclude configuration or worker content differs unexpectedly;
- the repository reports corruption, an unexplained lock, missing packs, or another writer;
- the requested task is actually PVE, VZDump, NAS snapshot, or restore work.

Do not use `restic unlock`, `repair`, `forget`, `prune`, or repository-file deletion as an automatic preflight fix.

## 7. Procedure

The canonical interactive launcher executes the protected worker through `sudo`.

**Changes Rotom and the Restic repository**

```bash
/usr/local/bin/backup-restic-to-nas
```

Keep the session attached until the command exits. A successful run must complete both snapshot creation and the installed retention/prune phase with exit status zero.

For a disconnect-safe manually initiated run, use the existing systemd service instead of wrapping the worker in an ad hoc background process:

**Changes Rotom and the Restic repository**

```bash
sudo systemctl start rotom-restic-backup.service
```

Use only one launch method for a run. Never start the service while the launcher or another Restic process is active.

## 8. Verification

**Read-only**

```bash
systemctl status rotom-restic-backup.service --no-pager
sudo journalctl -u rotom-restic-backup.service -n 160 --no-pager
findmnt -M /mnt/nas-rotom-restic-backup -t nfs,nfs4 \
  -o TARGET,SOURCE,FSTYPE,OPTIONS
sudo env \
  RESTIC_REPOSITORY=/mnt/nas-rotom-restic-backup/rotom-restic-backup \
  RESTIC_PASSWORD_FILE=/etc/restic/restic-repository-password \
  restic snapshots --host rotom --tag automatic --latest 3
systemctl --failed --no-pager
```

Success requires:

- the command or service exited successfully;
- the backup output contains a completed snapshot rather than only a scan;
- the retention/prune phase completed without error;
- the newest automatic snapshot timestamp corresponds to the run;
- the repository remains reachable through the exact NFS mount;
- no new failed systemd unit or application failure appears.

Snapshot existence proves that Restic committed a snapshot; it does not by itself prove full-pack readability or end-to-end restoration. Use the separately scoped repository-check and isolated-restore procedures in document 05 for those assurance levels.

## 9. Rollback or recovery

A completed backup is not rolled back. Restic retention governs automatic snapshot cleanup. Do not delete a new snapshot merely because the run was manually initiated.

If a run fails before committing a snapshot, source data should remain unchanged. Preserve the error and inspect the repository using supported Restic commands. Do not manually delete files from `data`, `index`, `keys`, `snapshots`, `locks`, or `config`.

If Restic reports a stale lock, first prove that no Restic process and no backup service is active. `restic unlock` changes repository metadata and requires a separately reviewed, explicitly authorized recovery step. Never remove a lock file manually.

## 10. Troubleshooting branches

- **Backup mount absent:** follow the NAS mount runbook for `/mnt/nas-rotom-restic-backup`; do not run the Libraries container helper.
- **Service already active:** monitor it; do not start a second run.
- **Required SQLite database missing or invalid:** stop and repair the application/state discrepancy before backup.
- **Repository locked:** identify the holder. If none exists, prepare a reviewed supported unlock; do not delete lock files.
- **Snapshot succeeds but prune fails:** treat the run as incomplete, preserve logs, and inspect repository health/capacity before any retry.
- **No new snapshot:** confirm the exact exit status and full error. Do not infer success from an inactive oneshot service.
- **Repository check requested:** use document 05; `restic check --read-data` is a long, NAS-intensive operation and is not routine post-run verification.
- **Restore requested:** select and inspect a recovery point, restore into an isolated restricted staging location, verify it, and only then plan any live replacement.

## 11. Evidence to retain

Record the run time in `America/Los_Angeles`, launch method, mount source, service/command result, newest snapshot short ID and timestamp, retention/prune result, and outstanding errors. Do not record passwords, environment values containing secrets, private application data, or restored file contents.

## 12. Related documentation

- [05-Backup and Restore](../05-Backup-and-Restore.md)
- [04-NAS and Storage](../04-NAS-and-Storage.md)
- [06-Maintenance and Automation](../06-Maintenance-and-Automation.md)
- [NAS Mount Recovery](NAS-Mount-Recovery.md)

## 13. Maintenance notes

Revalidate this runbook whenever the service/timer, launcher, worker, repository, mount, password-file path, include/exclude policy, SQLite staging set, retention, or backup scope changes.

