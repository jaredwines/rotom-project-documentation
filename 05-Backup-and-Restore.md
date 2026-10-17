# 05 - Backup and Restore

**Documentation set:** Rotom Runbook
**Document role:** Canonical source for Restic backup architecture, scope, retention, verification, restore evidence, and recovery boundaries
**Recovery scope:** pre-migration Rotom plus current Proxmox/VM foundation  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-16 — Separate the total-loss guide from RR membership
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `04-NAS-and-Storage.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1. Current Recovery Architecture**, **2. Rotom Restic Backup**, and **3. PVE Restic Backup**.
- Use **7. Restore Procedure** for recovery planning, while verifying the selected recovery point and current target paths before action.
- Treat **6. Recovery-Critical Historical Evidence** as evidence; it does not prove that a snapshot, local artifact, mount, or credential path still exists.
- Review **8. Recovery Limitations and Outstanding Work** before relying on a protection domain.
- For complete loss of the PVE computer, use the supplementary repository [PVE and Rotom Total-Loss Recovery guide](guide/PVE-and-Rotom-Total-Loss-Recovery.md#emergency-recovery-path). The guide is not an RR member; this document retains ownership of backup architecture and ordinary restores.
- This file owns backup and restore claims; storage locations, automation schedules, and privilege boundaries remain canonical in documents 04, 06, and 07.

## Start by Task or Symptom

| Task | Procedure |
|---|---|
| Run or verify Rotom Restic Backup | [Run and Verify Rotom Restic Backup](#run-and-verify-rotom-restic-backup) |
| Investigate a failed scheduled backup | Perform the runbook preflight, then inspect the owning timer/service in [document 06](06-Maintenance-and-Automation.md#investigate-a-failed-timer-or-automation) |
| Verify repository structure or full pack readability | Use the assurance-level guidance in this document; do not confuse snapshot creation with `restic check` or `restic check --read-data` |
| Restore data | Select a known recovery point and restore to an isolated restricted staging location before planning live replacement |
| PVE Restic or Rotom VM VZDump work | Use the PVE sections in this document; all PVE actions are **PVE — Jared executes** |
| Rebuild from blank replacement hardware | Use the supplementary [PVE and Rotom Total-Loss Recovery guide](guide/PVE-and-Rotom-Total-Loss-Recovery.md#emergency-recovery-path) |

## Run and Verify Rotom Restic Backup

### 1. Outcome

Create one Rotom Restic host-data snapshot through the installed worker, apply its established `7 daily / 4 weekly / 12 monthly` automatic-snapshot retention and prune policy, and verify that the run completed successfully and produced a current snapshot. Source data and repository internals must not be manually edited.

### 2. Use this runbook when

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

### 3. Impact and authorization

The backup reads configured Rotom paths, refreshes restricted system-information files under `/var/backups/system-info`, creates transactionally consistent copies of the NPM and Home Assistant SQLite databases, writes a tagged Restic snapshot, then runs `forget --prune` under the installed retention policy. It can use significant disk, CPU, NAS I/O, and time.

The repository password file is `/etc/restic/restic-repository-password`. Never print, copy, paste, hash, or record its contents. Backups may contain credentials, certificates, application data, and personal information; do not expose snapshot contents in routine evidence.

### 4. Canonical current-state inputs

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

### 5. Read-only preflight

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

**Read-only**

**Credential-sensitive**

```bash
sudo env \
  RESTIC_REPOSITORY=/mnt/nas-rotom-restic-backup/rotom-restic-backup \
  RESTIC_PASSWORD_FILE=/etc/restic/restic-repository-password \
  restic snapshots --host rotom --tag automatic --latest 3
```

Expected result is an enabled/active timer, inactive oneshot service between runs, exact active NFS mount, no concurrent Restic process, protected control files, and readable snapshot metadata.

### 6. Stop conditions

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

### 7. Procedure

The canonical interactive launcher executes the protected worker through `sudo`.

**Changes Rotom and protected storage/repository state**

```bash
/usr/local/bin/backup-restic-to-nas
```

Keep the session attached until the command exits. A successful run must complete both snapshot creation and the installed retention/prune phase with exit status zero.

For a disconnect-safe manually initiated run, use the existing systemd service instead of wrapping the worker in an ad hoc background process:

**Changes Rotom and protected storage/repository state**

```bash
sudo systemctl start rotom-restic-backup.service
```

Use only one launch method for a run. Never start the service while the launcher or another Restic process is active.

### 8. Verification

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

### 9. Rollback or recovery

A completed backup is not rolled back. Restic retention governs automatic snapshot cleanup. Do not delete a new snapshot merely because the run was manually initiated.

If a run fails before committing a snapshot, source data should remain unchanged. Preserve the error and inspect the repository using supported Restic commands. Do not manually delete files from `data`, `index`, `keys`, `snapshots`, `locks`, or `config`.

If Restic reports a stale lock, first prove that no Restic process and no backup service is active. `restic unlock` changes repository metadata and requires a separately reviewed, explicitly authorized recovery step. Never remove a lock file manually.

### 10. Troubleshooting branches

- **Backup mount absent:** follow the NAS mount runbook for `/mnt/nas-rotom-restic-backup`; do not run the Libraries container helper.
- **Service already active:** monitor it; do not start a second run.
- **Required SQLite database missing or invalid:** stop and repair the application/state discrepancy before backup.
- **Repository locked:** identify the holder. If none exists, prepare a reviewed supported unlock; do not delete lock files.
- **Snapshot succeeds but prune fails:** treat the run as incomplete, preserve logs, and inspect repository health/capacity before any retry.
- **No new snapshot:** confirm the exact exit status and full error. Do not infer success from an inactive oneshot service.
- **Repository check requested:** use document 05; `restic check --read-data` is a long, NAS-intensive operation and is not routine post-run verification.
- **Restore requested:** select and inspect a recovery point, restore into an isolated restricted staging location, verify it, and only then plan any live replacement.

### 11. Evidence to retain

Record the run time in `America/Los_Angeles`, launch method, mount source, service/command result, newest snapshot short ID and timestamp, retention/prune result, and outstanding errors. Do not record passwords, environment values containing secrets, private application data, or restored file contents.

### 12. Related documentation

- [05-Backup and Restore](05-Backup-and-Restore.md)
- [04-NAS and Storage](04-NAS-and-Storage.md)
- [06-Maintenance and Automation](06-Maintenance-and-Automation.md)
- [NAS mount recovery](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount)

### 13. Maintenance notes

Revalidate this runbook whenever the service/timer, launcher, worker, repository, mount, password-file path, include/exclude policy, SQLite staging set, retention, or backup scope changes.


## 1. Current Recovery Architecture

Rotom recovery is layered. No single layer replaces another:

1. Rotom Restic Backup protects selected guest-local configuration and application state.
2. PVE Restic Backup protects PVE host configuration and recovery material.
3. Rotom VM Backup creates whole-VM Proxmox VZDump archives.
4. NAS snapshot/restore protects authoritative NAS payload excluded from guest Restic.
5. Historical application-aware artifacts and the pre-virtualization raw image provide additional dated recovery evidence.

Select and verify an incident-appropriate recovery point before use. A successful scheduled job, repository check, archive decompression, or file listing is not by itself a complete restore rehearsal.

## 2. Rotom Restic Backup

| Control | Current value |
|---|---|
| Repository | `/mnt/nas-rotom-restic-backup/rotom-restic-backup` |
| NAS export | `Rotom_Restic_Backup/.data` |
| Manual launcher | `/usr/local/bin/backup-restic-to-nas` |
| Worker | `/usr/local/sbin/rotom-restic-backup` |
| Service / timer | `rotom-restic-backup.service` / `rotom-restic-backup.timer` |
| Schedule | Around `03:00`, persistent, randomized delay up to ten minutes |
| Retention | 7 daily / 4 weekly / 12 monthly |
| Credential path | `/etc/restic/restic-repository-password`; contents never document |

The current source scope includes `/etc`, `/home`, `/root`, `/opt`, `/srv`, `/usr/local`, `/var/lib/docker/volumes`, and `/var/backups/system-info`, subject to the installed include/exclude files. `/mnt` remains excluded.

The latest documented automatic observation is snapshot `1fd623ca` from 2026-10-15 around 03:00 PDT, approximately 20.282 GiB, with a successful service result. Manual post-credential-path snapshot `333c87b3` also completed successfully. Jared confirmed current full data-read verification and a restore rehearsal, but the selected snapshot and detailed transcript were not supplied here.

The VM-local `/srv/rotom/backup-staging` path is absent. Historical references to local fast-rollback archives below that root do not establish current availability.

## 3. PVE Restic Backup

| Control | Current value |
|---|---|
| Repository | `/mnt/nas-pve-restic-backup/pve-restic-backup` |
| NAS export | `PVE_Restic_Backup/.data` |
| Worker | `/usr/local/sbin/pve-restic-backup` |
| Service / timer | `pve-restic-backup.service` / `pve-restic-backup.timer` |
| Schedule | Daily `04:00` |
| Staging | `/var/backups/pve-restic-recovery` |
| Credential path | `/etc/restic/restic-repository-password` on PVE |

The 2026-10-14 audit found ten snapshots and a successful `restic check`. Manual post-credential-path snapshot `b5af666b` completed successfully, and the timer was enabled and active.

PVE work remains Jared-executed where the RR labels it **PVE — Jared executes**.

## 4. Rotom VM Backup

Proxmox job `rotom-vm-daily` protects VMID `100` on storage `nas-rotom-vm-backup`:

- Daily at `05:00`.
- Snapshot mode with zstd compression.
- `repeat-missed=0`.
- Retention: 7 daily / 4 weekly / 6 monthly.

Manual execution uses `/usr/local/bin/backup-rotom-vm-to-nas`, `rotom-vm-vzdump-manual.service`, worker `/usr/local/sbin/rotom-vm-vzdump-manual`, and lock `/run/lock/rotom-vm-vzdump-manual.lock`. Systemd owns the long-running job so it survives SSH disconnect.

The 2026-10-14 audit found eight retained VMID 100 archives. The newest listed archive passed full zstd decompression. Jared later confirmed current VMA verification or a disposable restore; the exact archive and transcript were not supplied. Archive availability changes under normal retention.

An earlier disposable restore proved that a selected VZDump archive could boot as isolated VMID `101` with no host attachments and its network link down. The disposable VM and temporary storage were removed after the test.

## 5. NAS-Resident Application Data

Rotom Restic excludes `/mnt`. Authoritative Libraries data, including Paperless media, therefore depends on NAS-side protection.

Paperless VM-local database, broker, configuration, search data, consume/export paths, and protected inputs are under `/srv/rotom/{appdata,secrets}/libraries/paperless`. Authoritative media is `/mnt/nas-libraries/paperless`. The restored five-file media set matched its retained manifest. Jared confirmed Libraries snapshot retention and an isolated restore outcome; exact policy values and the selected recovery point were not supplied.

Game Servers, Smart Home, and Hosted Websites NAS shares are empty service boundaries. Their application state remains VM-local and depends on Rotom Restic and, at a broader layer, whole-VM backup.

## 6. Recovery-Critical Historical Evidence

### Final pre-virtualization Restic baseline

Snapshot `fbe1838e` at 2026-09-25 18:14:22 PDT is the final documented pre-virtualization guest recovery point. Standard `restic check` and full `restic check --read-data` passed, reading all 971 packs without error. A representative isolated restore recovered both Palworld saves, the preserved audit artifacts, and staged Home Assistant, Zigbee, and NPM databases; restored SQLite quick checks passed.

Related audit paths are:

- `/home/jared/audits/jar-22-2026-09-22`
- `/home/jared/audits/jar-23-2026-09-23`
- `/home/jared/audits/jar-24-2026-09-25`

Verify that the selected snapshot still exists before relying on these paths.

### Raw NVMe rollback image

The historical raw image is:

`/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25/rotom-nvme-live-pre-proxmox-2026-09-25.img`

Recorded logical size: `250059350016` bytes. Recorded SHA-256:

`b62b2c8210bb2a6446abc29c5767c7d8531f547dfd2ec3f0259dbee8d74c5e90`

The acquisition-stream and complete stored-image reread hashes matched, and partition inspection found EFI and Linux filesystem partitions. The image is **best-effort live / crash-consistent** because the source root filesystem remained mounted. An actual bare-metal restore has not been rehearsed.

The guest Shared Drive mount is retired. Accessing, retaining, or restoring this NAS-resident image requires a separately authorized NAS path and exact target verification. The image and Restic repository are on different shares of the same physical UNAS appliance and therefore share that failure domain.

## 7. Restore Procedure

1. Identify the failure domain and the authoritative source before changing anything.
2. Stop writers only when the selected application or restore method requires it.
3. Verify the selected Restic snapshot, VZDump archive, NAS snapshot, or raw image exists and matches the intended system and date.
4. Restore into an isolated target whenever practical. Never overwrite current production data merely to test recoverability.
5. Preserve numeric ownership, modes, ACLs, service identity, Compose paths, and non-creating NAS-bind protections.
6. Validate files and databases before promotion. Use application-native integrity checks where available.
7. Promote only the reviewed target, then verify the application, storage, network, proxy, and backup state.
8. Retain the command results, selected recovery identifier, checksums, and validation evidence.

Stop if the repository is locked by active work, credentials are unavailable, the target contains current data, the expected source is not the documented filesystem, or the requested action would change retention, prune data, delete a recovery point, or overwrite protected state without explicit authorization.

## 8. Recovery Limitations and Outstanding Work

- No off-site layer is documented; Rotom Restic, PVE Restic, VM Backup, and NAS snapshots share the local site and substantial hardware failure domains.
- The raw NVMe image has not been restored to bare metal.
- Current detailed transcripts for the owner-confirmed Restic, VMA/disposable-restore, and Libraries NAS restore exercises were not supplied.
- NAS snapshot schedules and exact retention must be revalidated before incident reliance.
- Root-only historical rollback artifacts require exact-path inspection before any retention decision.
- Credentials, repository internals, NAS snapshot policy, and retention/prune behavior require separate authorization to change.

## 9. Related Documentation

- [02 — Docker Services](02-Docker-Services.md)
- [04 — NAS and Storage](04-NAS-and-Storage.md)
- [06 — Maintenance and Automation](06-Maintenance-and-Automation.md)
- [07 — Users and Permissions](07-Users-and-Permissions.md)

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate this document after a source path, exclusion, repository, credential path, timer, schedule, retention rule, archive destination, NAS snapshot policy, restore result, application persistence path, or recovery procedure changes. Revalidate the supplementary total-loss guide when one of its RR-owned dependencies changes.
