# 05 - Backup and Restore

**Documentation set:** Rotom Project Documentation  
**Document role:** Canonical source for Restic backup architecture, scope, retention, verification, restore evidence, and recovery boundaries  
**Recovery scope:** pre-migration Rotom plus current Proxmox/VM foundation  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-09-27 — Proxmox host-configuration Restic layer deployed, restored, and scheduled
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `04-NAS-and-Storage.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## 1. Purpose and Scope

This document is the canonical detailed owner for Rotom recovery state plus the preserved pre-migration backup/restore design. It distinguishes guest Restic snapshots, Proxmox host-configuration Restic snapshots, NAS data protection, representative restore evidence, the JAR-25 whole-disk rollback image, and current Proxmox whole-VM VZDump protection; those claims must not be treated as interchangeable.

## 2. Current Recovery State — post-change backup refresh, 2026-09-27

JAR-24 and JAR-25 remain the authoritative **pre-migration** recovery artifacts. JAR-32 established the first verified VM-era Restic write, JAR-66 established the separate whole-VM Proxmox backup path, JAR-33 accepted the Phase B architecture as **Rotom Virtualization Baseline**, and JAR-67 established the recurring Proxmox whole-VM schedule/retention layer. Later on 2026-09-27, the active guest Restic controls were normalized to `rotom-restic-backup` naming and the daily timer was recommissioned; the current guest Restic snapshot was refreshed; all older VMID 100 VZDump artifacts were intentionally removed and replaced by one newly verified whole-VM archive; and a separate Proxmox host-configuration Restic repository/service/timer was commissioned and restore-tested. The original JAR-33/JAR-67 named artifacts remain historical evidence, not current files.

- **VM restore/migration profile:** VMID `100` remains q35 + OVMF, CPU `x86-64-v2-AES`, 8 vCPU, 12 GiB RAM, 100 GiB VirtIO SCSI on `local-lvm`, VirtIO MAC `BC:24:11:97:10:47` on `vmbr0`, `onboot: 1`, with no PCI/USB passthrough dependency.
- **Current whole-VM recovery artifact:** dedicated PVE storage `nas-rotom-proxmox-backup` remains mounted at `/mnt/pve/nas-rotom-proxmox-backup`, backed by NAS `Rotom_Proxmox_Backup/.data` exported only to Proxmox `192.168.1.68`. The sole current VMID 100 archive is `vzdump-qemu-100-2026_09_27-14_08_51.vma.zst`, Proxmox-listed at `45,424,272,161` bytes (about 43 GB). The manual backup completed successfully using snapshot mode + zstd and qemu-guest-agent `fs-freeze`/`fs-thaw`; full `zstd -dc | vma verify -` returned exit code `0`, and `qm status 100` remained `running`. Final `pvesm list`/filesystem inspection showed exactly this archive plus its normal `.log`. No `.protected` marker was present, so the archive is currently eligible for normal `rotom-daily` retention unless protection is deliberately added later.
- **Superseded/deleted whole-VM artifacts:** at the administrator's request the previous protected JAR-33 baseline `vzdump-qemu-100-2026_09_27-01_19_56.vma.zst`, JAR-67 commissioning archive `vzdump-qemu-100-2026_09_27-12_21_27.vma.zst`, later verified manual archive `vzdump-qemu-100-2026_09_27-13_44_25.vma.zst`, their relevant sidecars/protection marker, and the orphaned interrupted-run `13_35_08` log were intentionally removed. PVE storage and `rotom-daily` were left intact. An interrupted 13:35 manual test had already been cleaned by Proxmox to a log only; no partial archive or VM lock remained.
- **Current guest Restic recovery artifact:** `/usr/local/bin/backup-restic-to-nas` created snapshot **`f666d63c` at `2026-09-27 13:30:28 PDT`**, host `rotom`, tag `automatic`, size `17.410 GiB`. The run applied the unchanged policy `7 daily / 4 weekly / 12 monthly`, removed superseded same-day snapshot `74d3b0ca`, and pruned about `120.462 MiB`. A subsequent repository `restic check` opened repository `799babb2`, checked all `21/21` snapshots, and returned `no errors were found`. Snapshot `5b4ed601` remains the first verified VM-era write; `74d3b0ca` is historical only because normal retention removed it.
- **Current Proxmox host-configuration Restic recovery artifacts:** repository `/mnt/nas-proxmox-restic-backup/proxmox-restic-backup` is Restic repository **`8ca0218c64`** on NAS `Proxmox_Restic_Backup/.data`, exported only to Proxmox `192.168.1.68`. First snapshot `0cf1c62a` was created at `2026-09-27 15:38:29 PDT`; it then passed an isolated full-snapshot restore to `/var/tmp/proxmox-restic-restore-test`, restoring 284 files/directories (205.589 KiB), with required PVE/network/scripts/inventory paths present and representative restored PVE files byte-identical to the staging copies. The temporary restore was deleted after verification. Subsequent systemd/manual tests exercised retention/prune; final observed latest snapshot is **`a59fc03f` at `2026-09-27 15:57:03 PDT`**, with `0cf1c62a` retained as the oldest daily/weekly/monthly point and redundant same-day `b1920314` removed.
- **Pre-migration recovery artifacts:** final pre-migration Restic snapshot `fbe1838e769cb740294ec0bf1017088a47ca5b97560083382af7cfcbebb2897f` remains the accepted host recovery point immediately before virtualization. JAR-25 raw image `/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25/rotom-nvme-live-pre-proxmox-2026-09-25.img`, SHA-256 `b62b2c8210bb2a6446abc29c5767c7d8531f547dfd2ec3f0259dbee8d74c5e90`, remains the catastrophic full-disk rollback path that would overwrite current Proxmox if restored to the NUC.
- **Current Restic storage:** NAS `Rotom_Restic_Backup/.data` is mounted at `/mnt/nas-rotom-restic-backup`; canonical repository is `/mnt/nas-rotom-restic-backup/rotom-restic-backup`, short ID `799babb2`, repository format version 2, compression auto. The old `Rotom_Home_Server_Backup/.data` share remains a point-in-time rollback copy and is not used by current backups.
- **Current active Restic control names:** `/usr/local/bin/backup-restic-to-nas` is the human-facing launcher; `/usr/local/sbin/rotom-restic-backup` is the root-protected implementation; `/etc/sudoers.d/rotom-restic-backup` authorizes Jared's exact command; `/etc/systemd/system/rotom-restic-backup.service` and `rotom-restic-backup.timer` are the service/timer; `/usr/local/sbin/rotom-restic-backup-status-api` and `rotom-restic-backup-status-api.service` provide Homepage status; `/etc/restic/nas-password` is the protected password-file path. The credential file is `root:root` mode `0600`; the sudoers fragment is mode `0440` and parsed successfully. Active-config grep returned no `UNAS|unas` matches, and old active `backup-to-unas`, `backup-restic-to-nas-root`, `unas-backup-status-api`, `update-to-nas`, `unas-backup*.service/timer`, and `/etc/restic/unas-password` paths were verified absent. Historical rollback copies keep their historical names.
- **Current Restic service/timer:** `rotom-restic-backup.service` is `Type=oneshot`, requires `/mnt/nas-rotom-restic-backup`, runs `/usr/local/sbin/rotom-restic-backup`, uses `Nice=10`, `IOSchedulingClass=best-effort`, and `IOSchedulingPriority=7`. `rotom-restic-backup.timer` is enabled/active and schedules `03:00` daily with `Persistent=true` and `RandomizedDelaySec=10m`. After recommissioning, the status API returned `schedule=Active`, `enabled=enabled`, `next_run=9/28/2026 3:07:23 AM`, `last_run=Unknown`, `last_result=success`.
- **Current Proxmox whole-VM manual/automatic split:** `/usr/local/bin/backup-proxmox-to-nas` is the manual VZDump operator command and runs VMID `100` with `vzdump --storage nas-rotom-proxmox-backup --mode snapshot --compress zstd`. Automatic whole-VM backup remains Proxmox-native job `rotom-daily` in `/etc/pve/jobs.cfg`: enabled, node `proxmox`, VMID 100, `05:00`, snapshot, zstd, `repeat-missed=0`, notes template `Scheduled Rotom VM backup - {{guestname}} VM {{vmid}} on {{node}}`, retention `keep-daily=7,keep-weekly=4,keep-monthly=6`. The native VZDump scheduler does **not** call the manual helper. Separately, Proxmox host-configuration Restic uses its own systemd service/timer at `04:00`.
- **Rollback/config history:** JAR-32 cutover rollback copies such as `/etc/fstab.pre-restic-share-cutover.20260926-235232`, `/usr/local/sbin/backup-to-unas.pre-restic-share-cutover.20260926-235232`, and `/etc/systemd/system/unas-backup.service.pre-restic-share-cutover.20260926-235232` remain historical rollback evidence and were intentionally not renamed. The disposable `/tmp/backup-to-unas-no-retention` helper remains absent.
- **Mounted Restic-root metadata:** current mounted `/mnt/nas-rotom-restic-backup` NAS root is numeric `988:988` mode `0700`; the older observation of the underlying local mountpoint as `root:root` mode `0700` while unmounted remains dated evidence, not a current mounted-filesystem fact. Root-run repository access and writes are verified.
- **Backup scope/retention:** the privileged script still stages the required NPM `database.sqlite`, Home Assistant `home-assistant_v2.db`, and Home Assistant `zigbee.db`; host Restic excludes `/mnt`. The normal manual or scheduled run performs `restic backup` followed by `restic forget --tag automatic --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune`. A manual backup can therefore expire a redundant same-day snapshot.

### Current recurring Proxmox VM backup schedule — 2026-09-27

PVE job `rotom-daily` is enabled and stored in `/etc/pve/jobs.cfg`. It is the sole automatic **whole-VM** Proxmox backup scheduler for VMID 100. Current parameters are `schedule 05:00`, `compress zstd`, `mode snapshot`, `node proxmox`, `storage nas-rotom-proxmox-backup`, `vmid 100`, `repeat-missed 0`, and `prune-backups keep-daily=7,keep-weekly=4,keep-monthly=6`.

The first unattended scheduler-triggered `05:00` execution still has not been observed in the supplied evidence. Manual/scheduled-equivalent VZDump execution and full VMA verification are verified; do not claim the unattended run has occurred until a future PVE task/archive proves it. Likewise, `rotom-restic-backup.timer` is enabled/active with a populated next trigger, but the first timer-triggered run under the renamed guest Restic unit has not yet been observed; the same backup implementation is verified through the manual launcher. `proxmox-restic-backup.timer` is enabled/active for `04:00`, but its first genuinely timer-triggered unattended run also remains unobserved; both direct service execution and interactive manual execution of its implementation are verified.

## 2A. Current VM-Era Restic Repository and Control Files

Current authoritative paths/state:

- Repository: `/mnt/nas-rotom-restic-backup/rotom-restic-backup`
- Manual launcher: `/usr/local/bin/backup-restic-to-nas` (`root:root`, mode `0755`)
- Privileged implementation: `/usr/local/sbin/rotom-restic-backup` (`root:root`, mode `0700`)
- Exact-command sudoers rule: `/etc/sudoers.d/rotom-restic-backup` (`root:root`, mode `0440`) -> `jared ALL=(root) NOPASSWD: /usr/local/sbin/rotom-restic-backup`
- Include/exclude lists: `/etc/restic/include.txt`, `/etc/restic/exclude.txt`
- Protected password file: `/etc/restic/nas-password` (`root:root`, mode `0600`; never document contents)
- Backup service: `/etc/systemd/system/rotom-restic-backup.service`
- Backup timer: `/etc/systemd/system/rotom-restic-backup.timer` — enabled/active, daily `03:00`, `Persistent=true`, `RandomizedDelaySec=10m`
- Status API executable/service: `/usr/local/sbin/rotom-restic-backup-status-api`, `/etc/systemd/system/rotom-restic-backup-status-api.service` — enabled/active on TCP `8787`, endpoint `/backup-status`
- Homepage endpoint remains `http://192.168.1.69:8787/backup-status`
- Current verified snapshot: `f666d63c`; repository check: 21/21 snapshots, no errors

The manual command `backup-restic-to-nas` is a **real backup command**, not a harmless test. It runs the same backup + retention/prune implementation used by the service. Historical scripts/unit names in rollback copies and preserved pre-migration sections are retained as dated evidence only.

## 2B. Current Proxmox Host-Configuration Restic Repository and Control Files

Current authoritative Proxmox host-config backup paths/state:

- NAS export: `Proxmox_Restic_Backup/.data`, authorized only to Proxmox `192.168.1.68`.
- Host mount: `/mnt/nas-proxmox-restic-backup`, configured in Proxmox `/etc/fstab` as NFS with `defaults,_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s`.
- Repository: `/mnt/nas-proxmox-restic-backup/proxmox-restic-backup`; repository ID `8ca0218c64`, format v2, compression auto.
- Protected password-file path: `/etc/restic/nas-password` (`root:root`, mode `0600`; contents never document).
- Generated recovery staging root: `/var/backups/proxmox-restic-recovery` (`files/`, `inventory/`, and `RESTORE-NOTES.txt`).
- Manual launcher: `/usr/local/bin/backup-restic-to-nas` (`root:root`, mode `0755`), which directly executes `/usr/local/sbin/proxmox-restic-backup` so an interactive administrator sees normal Restic output.
- Privileged implementation: `/usr/local/sbin/proxmox-restic-backup` (`root:root`, mode `0700`).
- Service: `/etc/systemd/system/proxmox-restic-backup.service`; `Type=oneshot`, `Environment=HOME=/root`, `RequiresMountsFor=/mnt/nas-proxmox-restic-backup`, `UMask=0077`, `Nice=10`, best-effort I/O priority 7.
- Timer: `/etc/systemd/system/proxmox-restic-backup.timer`; enabled/active, daily `04:00`, `Persistent=true`, `AccuracySec=1min`.
- Retention: `restic forget --tag automatic --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune`.
- Final observed latest snapshot: `a59fc03f` at `2026-09-27 15:57:03 PDT`; retained first/oldest point `0cf1c62a`.

The recovery bundle is a deliberate reconstruction set, not a whole-root backup. It copies the current non-secret `/etc/pve` tree while excluding `/etc/pve/priv` and private `*.key` material; includes host networking, hostname/hosts/fstab, boot/kernel/module configuration, APT configuration, SSH daemon configuration, systemd configuration, and `/usr/local/bin` + `/usr/local/sbin`; and preserves `/root/.ssh/authorized_keys` as a resolved **public-key** copy because the live path is a symlink into `/etc/pve/priv`. It does not copy SSH private keys or `/etc/restic/nas-password`. Generated inventories include Proxmox version, VMID 100 config, PVE storage/jobs, block/mount/disk state, network addresses/routes, enabled units, package lists, kernel/CPU, and boot information. VM disk images, VZDump archives, mounted NAS data, and Restic repository internals are not source data for this job.

Restore policy is intentionally conservative: restore a Proxmox Restic snapshot into a temporary directory on a newly installed compatible host, review/reconcile the preserved configuration, and then deliberately reapply needed host/PVE settings. Do **not** recursively restore the bundle over a live `/etc/pve` pmxcfs tree. The first snapshot `0cf1c62a` passed an isolated restore test: 284 files/directories were restored, critical PVE/network/script/inventory paths were present, representative `100.conf`, `storage.cfg`, and `jobs.cfg` files matched the live staging copies, `/etc/pve/priv` and the Restic password were absent, and the scratch restore directory was removed afterward.

The NFS guard has two stages. The script first performs `timeout 35s ls -A /mnt/nas-proxmox-restic-backup` to trigger the systemd automount, then requires `findmnt` to report NFS/NFS4 and requires the exact expected NAS source before opening Restic. This was added after a reboot test showed only the autofs layer active and the earlier `ls -ld` check failed to trigger the underlying NFS mount. The corrected path was then verified from an initial autofs-only state: the NFSv3 mount appeared, snapshot `d4378460` was saved, retention/prune completed, and the service returned success.

This repository is a separate encrypted recovery layer but **not** a separate physical failure domain: `Proxmox_Restic_Backup`, guest `Rotom_Restic_Backup`, and `Rotom_Proxmox_Backup` all terminate on the same UniFi UNAS 2 appliance. Off-site/site-loss protection remains future recovery-policy work.

## 2C. Final Pre-Migration Backup Architecture and Verified Baseline

At the final pre-migration baseline, Rotom backed up its Linux configuration and service data to the UniFi UNAS 2 using Restic. The established backup scope, repository format, scripts, schedule, retention policy, and root-run execution model remained unchanged through JAR-24. JAR-6 changed current service primary GIDs to Infra `5002`, Smarthome `5003`, Documents `5004`, Downloads `5005`, Web `5006`, Filesync `5007`, Apps `5008`, and Auth `5009` while preserving UIDs and keeping their local homes under `/home`; those local paths therefore remain within the existing host Restic source scope. JAR-6 also commissioned eight NAS shares under `/mnt`, which remain outside that scope because `/mnt` is explicitly excluded.

**Latest documented Restic snapshot: `fbe1838e` (`fbe1838e769cb740294ec0bf1017088a47ca5b97560083382af7cfcbebb2897f`), created `2026-09-25 18:14:22 PDT`, host `rotom`, tag `automatic`.** This is the final JAR-24 pre-migration host recovery point. It covers the established host roots `/etc`, `/home`, `/root`, `/opt`, `/srv`, `/usr/local`, `/var/backups/system-info`, and `/var/lib/docker/volumes`. JAR-24 explicitly verified the JAR-22 and JAR-23 audit trees, fresh staged Home Assistant/Zigbee/NPM databases, both Palworld `Level.sav` files, and the JAR-23 NAS-recovery helper/unit inside this snapshot. The canonical run completed the unchanged retention/prune policy.

Backup scope/rules, SQLite staging, repository structure, full-pack readability, and representative restore capability are now reverified at the JAR-24 migration gate. Snapshot `fbe1838e` supersedes `a6d86b20` as the final pre-migration host recovery point. Standard `restic check` passed, full `restic check --read-data` read all 971 packs with no errors, and a representative isolated restore recovered both audit artifacts, all three staged databases, and both Palworld world saves; the restored databases passed `PRAGMA quick_check`. Mounted NAS data under `/mnt` remains excluded. JAR-25 now adds a full raw NVMe rollback image on the UNAS Shared Drive; restoring that image to bare metal remains untested. A full end-to-end Palworld server restore also remains untested.

### JAR-24 final pre-migration recovery-point verification — 2026-09-25

JAR-24 began with a read-only preflight: the backup NFS mount and repository were reachable, `unas-backup.service` was inactive, both JAR-22 and JAR-23 `SHA256SUMS` manifests verified, staged `home-assistant_v2.db`, `zigbee.db`, and `npm-database.sqlite` each passed `PRAGMA quick_check`, repository snapshot listing succeeded, and zero failed systemd units were present.

An aborted helper used for snapshot-ID parsing left a Restic repository lock owned by an orphaned root Restic process. Two later canonical backup attempts saved valid intermediate snapshots but could not acquire the exclusive lock for retention/prune. After confirming there was no running Restic process and no active backup service, `restic unlock` removed exactly one stale lock. No Restic repository internals were manually rearranged or deleted.

The next canonical `/usr/local/bin/backup-to-nas` run saved snapshot `fbe1838e` at `2026-09-25T18:14:22.294529014-07:00`, applied the unchanged `keep 7 daily, 4 weekly, 12 monthly` policy, removed five redundant same-day Rotom snapshots (`983b9f6c`, `2b4e39b6`, `18aab38f`, `895f46eb`, `391fc3d3`), and completed prune: 18 packs were repacked, obsolete indexes were deleted, and 20 old packs were removed.

The final snapshot explicitly contains:

- `/home/jared/audits/jar-22-2026-09-22`, including `SHA256SUMS`;
- `/home/jared/audits/jar-23-2026-09-23`, including the JAR-23 acceptance summary and both Palworld checksum manifests;
- `/var/backups/system-info/sqlite/home-assistant_v2.db`;
- `/var/backups/system-info/sqlite/zigbee.db`;
- `/var/backups/system-info/sqlite/npm-database.sqlite`;
- Jared Palworld `DB40338954B844C28CEA21471A392F98/Level.sav`;
- Fran Palworld `396F5898378F4E9CAE89461F403653D9/Level.sav`;
- `/usr/local/sbin/rotom-nas-docker-recovery`;
- `/etc/systemd/system/rotom-nas-docker-recovery.service`.

Both `restic check` and `restic check --read-data` passed; the latter read all 971 packs with no errors. A representative isolated restore recovered the two audit artifacts, all three staged databases, and both Palworld saves (34.114 MiB total); all restored SQLite databases passed `PRAGMA quick_check`, then the temporary restore directory was removed. JAR-24 audit evidence is `/home/jared/audits/jar-24-2026-09-25`; `06-jar-24-summary.txt` SHA-256 is `d02af0c4e62174c0cfbde461a2e0cda93d82abf0a7f5fbe685226dc4f49730a7`.


### JAR-25 full raw NVMe rollback image — 2026-09-25

JAR-25 adds a second recovery mechanism that is distinct from Restic. It is a byte-for-byte raw image file of the existing system disk, not a Restic snapshot and not part of the Restic repository. Read-only identification fixed the source as `/dev/nvme0n1`, Samsung SSD 970 EVO Plus 250GB, serial `S59BNM0R702702E`, exact size `250059350016` bytes. The image was streamed directly to the physically separate UNAS Shared Drive at:

`/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25/rotom-nvme-live-pre-proxmox-2026-09-25.img`

The final file's logical size exactly equals the source disk (`250059350016` bytes), with recorded allocated size `250059358208` bytes. The SHA-256 calculated from the acquisition stream and the SHA-256 calculated by a complete reread of the stored NAS image both equal:

`b62b2c8210bb2a6446abc29c5767c7d8531f547dfd2ec3f0259dbee8d74c5e90`

The stored image was inspectable with `fdisk`; the EFI System Partition and Linux filesystem partition were both visible. `RESTORE-PROCEDURE.txt`, `CONSISTENCY-NOTE.txt`, source/image inspection files, checksum files, the final JAR-25 result, and a metadata checksum manifest are stored beside the image. Before acquisition, the workflow reverified final Restic snapshot `fbe1838e769cb740294ec0bf1017088a47ca5b97560083382af7cfcbebb2897f` and the critical JAR-23 artifacts.

The acquisition was deliberately performed from the running Mint installation rather than an offline live environment. The workflow stopped the 16 running Docker workloads, paused selected active maintenance scheduling, flushed filesystem writes, then read the whole disk while the ext4 root remained mounted and the host OS/SSH/networking/systemd/journald remained active. The image is therefore explicitly classified **best-effort live / crash-consistent**. Matching hashes prove the stored file matches the byte stream read from the NVMe; they do not convert the live acquisition into an offline point-in-time filesystem snapshot.

The original 16-container set was restarted after the image write and before the full NAS reread; final output reported all 16 restored and zero failed systemd units. No Proxmox installation or source-disk write occurred. A true restore of this image has not yet been rehearsed. The image is physically separate from the Rotom NVMe but shares the UniFi UNAS 2 hardware failure domain with the Restic repository and other NAS data.

## 3. Preserved Pre-Migration Restic Repository and Control Files

| Item | Verified location or value |
| --- | --- |
| Server | `rotom`; primary domain `rotom.casa` |
| NAS | UniFi UNAS 2, `192.168.1.70` |
| NAS share | `Rotom_Home_Server_Backup`, mounted through NFS |
| Backup mount | `/mnt/nas-rotom-backup` |
| Restic repository | `/mnt/nas-rotom-backup/rotom-restic-backup` |
| Repository identifier | `799babb2` (displayed short ID) |
| Repository format | Version 2; compression auto |
| Password-file reference | `/etc/restic/unas-password` |
| Backup script | `/usr/local/sbin/backup-to-unas` |
| Shared manual launcher | `/usr/local/bin/backup-to-nas` |
| Jared sudoers rule | `/etc/sudoers.d/update-to-nas` |
| Fran sudoers rule | `/etc/sudoers.d/fran-backup-to-nas` |
| Include file | `/etc/restic/include.txt` |
| Exclude file | `/etc/restic/exclude.txt` |
| Service | `/etc/systemd/system/unas-backup.service` |
| Timer | `/etc/systemd/system/unas-backup.timer` |
| Generated recovery inventory and staging | `/var/backups/system-info` |
| Backup share root | UNAS numeric `988:988`, mode `0700` |
| Backup export root handling | `no_root_squash,no_all_squash` verified on UNAS |

The password-file path is documented for operations; its contents must never be printed, pasted, or included in this guide. Backups can contain application credentials, private certificates, and personal data; restored staging trees require restricted access too. Keep a securely recoverable copy of the repository credential outside the failed host and outside the encrypted repository: backing up `/etc` alone does not solve the bootstrap problem of unlocking that repository.

The audited mount uses NFS v3 with systemd automount; its fstab options include `_netdev`, `nofail`, `x-systemd.automount`, and `x-systemd.device-timeout=30s`. Recover the exact export from the current storage documentation and configuration instead of guessing it. The backup service declares `RequiresMountsFor=/mnt/nas-rotom-backup`; the script also refuses to proceed if its mountpoint check fails.

Never manually reorganize or delete Restic repository internals, including `data`, `index`, `keys`, `snapshots`, or `config`.

## 4. Preserved Pre-Migration Backup Execution Path and Manual Access

`/usr/local/bin/backup-to-nas` is the canonical manual command for both human administrators. The shared launcher is `root:root` mode `0755` and hands off to the protected `/usr/local/sbin/backup-to-unas` script through `sudo`. The protected script remains `root:root` mode `0700`.

Jared's exact passwordless rule remains in `/etc/sudoers.d/update-to-nas`. Fran has a separate exact-command rule in `/etc/sudoers.d/fran-backup-to-nas`; supplied output confirmed that file as `root:root` mode `0440`, confirmed the complete sudoers configuration parses successfully, and confirmed Fran can request the protected script noninteractively. The user then confirmed the shared command worked for Fran.

On the **pre-migration Rotom host**, logged in as **Jared or Fran**, the established full-backup command was:

```bash
backup-to-nas
```

This runs the normal whole-server backup and its retention/prune phase. It is not a per-user backup. `/home/fran` was already included through the existing `/home` backup root, so no scope change was needed. After the 2026-09-18 share-root protection change, both Jared and Fran were verified using this launcher successfully even though both are denied ordinary direct access to `/mnt/nas-rotom-backup`; the root-run script performs the repository work. Jared's run created `d22fbd7a`, and Fran's run created `ed244f4e`; both completed retention/prune successfully. The underlying script, repository format, timer, retention policy, and NAS ownership were unchanged.

An earlier Jared launcher under `/home/jared/.local/bin` and a temporary Fran copy were present during setup. Their final cleanup was not shown. Do not rely on those copies as the shared interface; verify them before removing anything, and preserve `/usr/local/bin/backup-to-nas` as the documented command.

## 5. Preserved Pre-Migration Storage/Identity Changes Affecting Backup

The final pre-migration media identity is **UID `127`, primary GID `5000`**. Jellyfin, Radarr, and Sonarr remain `127:5000`; Gamarr remains `995:5001`. JAR-21 moved Prowlarr and qBittorrentVPN to the Downloads account at `901:5005`, with active project trees `/home/downloads/docker/prowlarr` and `/home/downloads/docker/qbittorrentvpn`. Older snapshots legitimately contain their earlier Media/Rotom/Infra paths and identities; inspect the selected snapshot before startup rather than restoring historical identity/path settings blindly.

The NAS media root is `988:5000`, mode `2770`. Its `library` and `torrents` trees use GID `5000`, with setgid, group-writable directories and group-writable files. Existing owner UIDs are mixed and were preserved; do not recursively change all NAS owners to UID `127` as a restore shortcut. Numeric IDs are authoritative across NFS. A name such as `fwupd-refresh` displayed on Rotom for UID/GID `988` is a local name lookup, not an instruction to assign the NAS data to that service.

On this UNAS setup, `rpc.mountd --manage-gids` replaces the client's supplementary group list using the NAS's UID lookup while retaining the request's primary GID. Client UID `1000` (`jared` on Rotom) resolves to `jwines760` on UNAS, with primary GID `988` and supplementary GID `987`. Merely adding client supplementary GID `5000` did not grant media access; using primary GID `5000` did. The adopted administration model keeps ordinary `jared` access separate from media and uses `sudo -iu media` for media operations. A final `id jared`/`getent group media` check on 2026-09-19 verified Jared is not a member of media GID `5000`. This is ordinary-account isolation; an administrator using sudo or privileged Docker access can still cross account boundaries.

**Backup access protection remains unchanged:** `/mnt/nas-rotom-backup` is numeric `988:988`, share-root mode `0700`, with verified `no_root_squash,no_all_squash` for the root-run backup. JAR-6 did not change Backup ownership, mode, export settings, repository internals, scripts, retention, or timer. After the normal Rotom reboot, the Backup automount was explicitly triggered and its exact source plus `988:988` mode `0700` metadata were reconfirmed. Historical ordinary-account denial evidence remains valid for the identities tested at that time; JAR-6's new-share access tests did not broaden the Backup share. The Btrfs filesystem remains `noacl`.

### Prowlarr JAR-21 path/ownership — backup implications

The active Prowlarr tree is `/home/downloads/docker/prowlarr`, within the unchanged `/home` Restic include root. The old `/home/infra/docker/prowlarr` rollback tree was removed after JAR-21 functional acceptance and a successful pre-cleanup backup. Final post-cleanup snapshot `a6d86b20` contains the active Downloads-owned configuration. Older snapshots preserve earlier Media/Rotom/Infra locations and must be interpreted as historical state rather than copied blindly over the current project.

### qBittorrentVPN and Gamarr migration — backup implications

The active local application trees are `/home/downloads/docker/qbittorrentvpn` and `/home/game/docker/gamarr`, both beneath the existing `/home` Restic root. qBittorrent's `.env` is included in encrypted Restic backup scope; never print or paste its credential contents. The old `/home/infra/docker/qbittorrentvpn` rollback tree was removed after guarded acceptance. Final snapshot `a6d86b20` contains the active Downloads-owned qBittorrent configuration and current Gamarr configuration. Active torrent data is `/mnt/nas-downloads/torrents` and final libraries remain under `/mnt/nas-media/library` and `/mnt/nas-game/library`; all are outside host Restic because `/mnt` is excluded.

The 2026-09-19 Homebridge container-name correction changed the then-current `/home/smart-home/docker/homebridge/compose.yaml` and `/home/jared/rotom-manual-verify.sh`; both known paths are within the normal `/home` Restic include scope. That Homebridge tree is now `/home/smarthome/docker/homebridge`. Snapshot `f0d51eaa` predates the correction; final JAR-19 snapshot `e886c55d` supplies the latest documented recovery point after the correction and later identity/JAR-19 work, but it predates final JAR-6 state and the service-home `nas` symlinks.

### JAR-6 service-to-NAS expansion — pre-migration baseline

| Service account | Current UID:primary GID | NAS mount | Backup implication |
|---|---|---|---|
| `media` | `127:5000` | `/mnt/nas-media` | Existing NAS content remains outside host Restic |
| `game` | `995:5001` | `/mnt/nas-game` | Existing NAS content remains outside host Restic |
| `infra` | `997:5002` | `/mnt/nas-infra` | New share currently empty; outside host Restic |
| `smarthome` | `126:5003` | `/mnt/nas-smarthome` | New share currently empty; HA/Homebridge remain local and backed up under `/home` |
| `documents` | `900:5004` | `/mnt/nas-documents` | New share currently empty; outside host Restic |
| `downloads` | `901:5005` | `/mnt/nas-downloads` | Active torrent storage is outside host Restic; active Prowlarr/qBittorrent config under `/home/downloads/docker` is inside host Restic |
| `web` | `902:5006` | `/mnt/nas-web` | New share currently empty; website data remains local/backed up under `/home/web` |
| `filesync` | `903:5007` | `/mnt/nas-filesync` | New share currently empty; outside host Restic |
| `apps` | `904:5008` | `/mnt/nas-apps` | New share currently empty; outside host Restic |
| `auth` | `905:5009` | `/mnt/nas-auth` | New share currently empty; outside host Restic |

JAR-6 originally commissioned the eight NAS boundaries with no authoritative workload data. JAR-21 subsequently activated the Downloads share for authoritative torrent data. `/etc/restic/exclude.txt` still explicitly contains `/mnt`, and the post-reboot closeout verified no JAR-6 NAS path was added to `/etc/restic/include.txt`. Therefore the existing host Restic job does not protect content placed on any of these shares. Before future critical data moves to one, define and verify a NAS-side protection/recovery strategy separately.

On 2026-09-21, local service-home NAS symbolic links were added for all ten service identities and then renamed to the final convention `/home/<service>/nas-<service> -> /mnt/nas-<service>`. Final target/traversal verification ended with `ALL NAS SHORTCUT RENAMES: PASS`. This does **not** change Restic source scope: `/mnt` remains excluded, no NAS path was added to the include file, and the shortcuts must not be treated as a mechanism for bringing NAS-resident data into the host backup. The canonical backup boundary remains the real filesystem paths and include/exclude rules above. Final JAR-21 snapshot `a6d86b20` postdates the shortcut topology and the completed Downloads migration.

The primary-GID changes themselves affect local `/home` metadata, not backup scope: those home trees remain under the existing `/home` include root. Arcane live data under the Docker named-volume source remains within the existing `/var/lib/docker/volumes` include root. Historical backups and containerd snapshot-layer objects with old GIDs were intentionally not rewritten.

### JAR-9 rollback recovery note — 2026-09-21

Snapshot `b5f77aeb` (`2026-09-20 22:46:49 PDT`) is retained intentionally as a recovery point for the **JAR-9 state immediately before rollback**. It contains `/home/game-servers`, the JAR-9 Media Gamarr path, `fstab`, and the then-current verifier; those are not the present active paths after rollback. The current active paths are again `/home/game` and `/home/game/docker/gamarr`. No fresh post-rollback Restic snapshot has yet been documented. Retained JAR-9 rollback-state files under `/home` and `/etc` are safety artifacts, not active configuration.

## 6. Preserved Pre-Migration Backup Scope and Exclusions

### Exact include rules

The audited `/etc/restic/include.txt` contains:

```text
/etc
/home
/root
/opt
/srv
/usr/local
/var/lib/docker/volumes
```

The script also supplies **`/var/backups/system-info` explicitly on the Restic command line**. Its absence from `include.txt` is intentional and does not represent a coverage gap. All eight roots appear in the verified snapshot metadata.

### Exact exclude rules

The audited `/etc/restic/exclude.txt` contains:

```text
/home/*/.cache
/home/*/.local/share/Trash
/root/.cache
**/node_modules
**/.cache
**/cache
**/tmp
**/*.log
/var/lib/docker/overlay2
/var/lib/docker/containers
/mnt
/media
```

These rules apply within the included trees. Coverage of a service directory does not mean every file beneath it is retained: matching caches, temporary directories, logs, and dependencies are excluded. Docker image layers and container writable layers are not recovery sources; persistent bind mounts, named volumes, Compose definitions, and configuration are the important sources.

### Verified service coverage

Historical path listings in snapshot `1a55dd4f` (`2026-09-14 03:09:49 PDT`) verified the service coverage before SQLite hardening under the paths that existed then. Snapshot `0d64ab6d` (`2026-09-18 23:08:43 PDT`) verifies the post-migration Palworld paths under `/home/game` while retaining the same `/home` backup root; the later SQLite verification remains separately recorded below.

| Service or data | Covered path |
| --- | --- |
| Home Assistant | `/home/smarthome/docker/home-assistant` |
| Homebridge | `/home/smarthome/docker/homebridge` |
| Nginx Proxy Manager data | `/home/infra/docker/nginx-proxy-manager/data` |
| NPM certificates | `/home/infra/docker/nginx-proxy-manager/letsencrypt` |
| Media service configuration | `/home/media/docker` (active Jellyfin/Radarr/Sonarr) |
| Active Prowlarr configuration | `/home/downloads/docker/prowlarr`; covered by final JAR-24 snapshot `fbe1838e` |
| Active qBittorrentVPN configuration | `/home/downloads/docker/qbittorrentvpn`; covered by final JAR-24 snapshot `fbe1838e` |
| Active Gamarr configuration | `/home/game/docker/gamarr`; covered by final JAR-24 snapshot `fbe1838e`; earlier restore-era evidence remains historical |
| Jared's Palworld server | `/home/game/docker/palworld-server-jared` |
| Fran's Palworld server | `/home/game/docker/palworld-server-fran` |
| Aloha Millworks website | `/home/web/docker/alohamillworks.com` |
| Jared Wines website | `/home/web/docker/jaredwines.com`; intentionally stopped as of JAR-19 closeout |
| Docker named volumes | `/var/lib/docker/volumes`, an explicit backup root |

### All `/mnt` mounts are outside the host Restic source scope

**The host Restic job excludes `/mnt`, so `/mnt/nas-media`, `/mnt/nas-game`, `/mnt/nas-shared-drive`, and `/mnt/nas-rotom-restic-backup` are all outside its backup source scope.** The verified include roots do not add any of those mounts back into the job.

This excludes NAS content under `/mnt/nas-media/library`, `/mnt/nas-game/library`, the active downloader torrent tree, and `/mnt/nas-shared-drive`. Backed-up application configuration is not a backup of NAS-resident library, torrent, or shared-drive data, so any non-reproducible NAS data needs a separate NAS protection policy. `/mnt/nas-rotom-restic-backup` is the current destination containing the Restic repository; it is not a source backing itself up. The retained old `Rotom_Home_Server_Backup` share is a rollback copy on the same UNAS, not an independent off-site copy.


### JAR-22 pre-migration audit artifacts — pending JAR-24 snapshot verification

JAR-22 created a read-only audit set at `/home/jared/audits/jar-22-2026-09-22`; the captured directory was 140K at closeout and contains host, CPU, storage, network, Docker, identity, NFS/automount, maintenance, and targeted reconciliation text artifacts plus `SHA256SUMS`. Because `/home` is an existing Restic include root and the audit path does not match the documented cache/tmp/log exclusions, it is **within configured host Restic source scope**. This is a scope statement, not proof that a particular snapshot already contains the files.

JAR-24 explicitly verified `/home/jared/audits/jar-22-2026-09-22` inside final pre-migration snapshot `fbe1838e`. JAR-22 itself did not run or alter Restic, retention, pruning, repository internals, or the backup timer; that earlier pending requirement is now satisfied by the JAR-24 evidence.


### JAR-23 application-aware preservation artifacts — 2026-09-25

JAR-23 created a second pre-migration audit/preservation set at `/home/jared/audits/jar-23-2026-09-23`, which is under the existing `/home` Restic source root. It contains stopped-state Palworld manifests, database-preservation evidence, website checksum manifests, stateful-app persistence inventory, NAS/startup diagnostics, and the consolidated acceptance summary plus `SHA256SUMS`. The consolidated acceptance artifact `10-jar-23-acceptance-summary.txt` has SHA-256 `fbe3f9b483fa0f126af4f1f84edb5c52313b3a98a1c4dde1921f3c6e55aafbb5`.

Current Palworld preservation evidence supersedes historical file counts without changing the known world IDs. Jared world `DB40338954B844C28CEA21471A392F98` was stopped cleanly and captured with 135 `.sav` checksum entries; Fran world `396F5898378F4E9CAE89461F403653D9` was captured with 108. Both same existing containers returned healthy after the stopped-state capture. The daily application-created archives were also present on September 25.

Home Assistant's native backup `/home/smarthome/docker/home-assistant/backups/Automatic_backup_2026.9.3_2026-09-25_05.37_51002539.tar` existed at 11,458,560 bytes. JAR-23 then refreshed all three established SQLite staging copies at approximately 17:18 PDT using the same SQLite online-backup pattern documented below; each staged DB is `root:root` mode `0600` and passed `PRAGMA quick_check`. NPM's live `data` and `letsencrypt` trees remained present, with 44 certificate files observed. Website content for Aloha Millworks and intentionally undeployed Jared Wines was checksummed into non-secret audit manifests; secret-like file contents were not copied into the audit.

JAR-23 did **not** run Restic, retention, or prune; at JAR-23 closeout `a6d86b20` was still the latest documented point. JAR-24 has now satisfied that deferred gate: final snapshot `fbe1838e` contains both `/home/jared/audits/jar-22-2026-09-22` and `/home/jared/audits/jar-23-2026-09-23` together with the fresh `/var/backups/system-info/sqlite` copies, and full repository/restore verification passed.

## 7. Preserved Pre-Migration Backup Execution and Consistency Handling

The script uses `set -Eeuo pipefail`, checks the backup mount, and generates:

```text
/var/backups/system-info/packages.txt
/var/backups/system-info/docker-containers.txt
/var/backups/system-info/docker-images.txt
```

Package inventory comes from `dpkg-query`; Docker inventory comes from `docker ps -a` and `docker images`. Docker inventory commands tolerate errors, so verify their files when using them for recovery. These lists help rebuild the host; they are not exported container images or application database dumps.

The flow after hardening is:

```text
Mount check → system inventory → SQLite-consistent staging
    → Restic backup of included roots and system-info, tagged automatic
    → retention with forget --prune
```

### NPM and Home Assistant SQLite staging

Hardening was installed at approximately **23:27 PDT on September 14, 2026**. The original script was preserved as:

```text
/usr/local/sbin/backup-to-unas.before-sqlite-backups-20260914-232739
```

The modified script passed `bash -n`. All three live databases and all three generated staging copies passed SQLite `PRAGMA quick_check`.

| Live database | Consistent staging copy |
| --- | --- |
| `/home/infra/docker/nginx-proxy-manager/data/database.sqlite` | `/var/backups/system-info/sqlite/npm-database.sqlite` |
| `/home/smarthome/docker/home-assistant/home-assistant_v2.db` | `/var/backups/system-info/sqlite/home-assistant_v2.db` |
| `/home/smarthome/docker/home-assistant/zigbee.db` | `/var/backups/system-info/sqlite/zigbee.db` |

The installed block uses Python's SQLite online backup API with read-only source connections. It writes temporary `.new` copies, checks them, sets mode `0600`, and replaces the staging destinations. The staging directory uses mode `0700`; the verified files are owned by `root:root`. Failures abort the script before a new Restic backup can proceed through this block.

Snapshot **`e98243f5` contains all three staging files**, closing the earlier timing gap: snapshot `1a55dd4f` preceded hardening and could not contain them.

NPM and Home Assistant remain running during this process. Each staged database is transactionally consistent; the three copies are not a single synchronized snapshot of all applications. Original live application trees remain covered as well. For database recovery, prefer the validated staged copies over assuming a raw live file capture is consistent. This hardening does not establish application-consistent backup of every other database on Rotom.

### Both Palworld servers

The current service account is `game` at UID/GID `995:5001`, home `/home/game`. During the controlled 2026-09-18 migration, both containers were stopped before the account/home move and recreated afterward. The current verified bind mounts are:

| Container | Host bind mount → container path |
| --- | --- |
| `palworld-server-jared` | `/home/game/docker/palworld-server-jared` → `/palworld` |
| `palworld-server-fran` | `/home/game/docker/palworld-server-fran` → `/palworld` |

Current exact world-save roots are:

```text
/home/game/docker/palworld-server-jared/Pal/Saved/SaveGames
/home/game/docker/palworld-server-fran/Pal/Saved/SaveGames
```

World identifiers are `DB40338954B844C28CEA21471A392F98` for Jared and `396F5898378F4E9CAE89461F403653D9` for Fran. Before restart, SHA-256 manifests covered 140 Jared `.sav` files and 112 Fran `.sav` files; after the home move, both before/after manifests compared byte-for-byte identical. The temporary migration manifests were later deleted after verification.

Both server trees also contain application-created archives under their respective `backups` directories. The newest archives inspected during the migration passed `gzip -t` and contained each world `Level.sav` plus player saves. These application backups were retained. Snapshot `0d64ab6d` explicitly contains both current world `Level.sav` paths under `/home/game`. Historical Restic snapshots from before the rename legitimately contain `/home/game-server/...`; snapshots created before the 2026-09-19 GID migration may also preserve GID `985`. Do not rewrite or manually reorganize repository contents to change historical paths or metadata. Final JAR-24 snapshot `fbe1838e` includes both current Palworld trees and both exact world `Level.sav` paths; the JAR-24 representative restore recovered both world saves successfully. Earlier `f0d51eaa` restore evidence remains historical.

The current Compose files use `.:/palworld`, so the home move did not require a bind-path edit inside Compose. Post-migration Docker metadata verified both Compose working directories and config-file paths under `/home/game`, and both containers returned `running` / `healthy`. Restic does not stop or pause these servers during ordinary backups. A representative isolated Restic restore successfully recovered both current `Level.sav` files, but a full end-to-end Palworld server/world recovery rehearsal has not been performed. Preserve both live saves and application-created archives; never initialize a replacement world over an existing save tree.

## 8. Preserved Pre-Migration Scheduled Execution

Verified timer settings:

```ini
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
RandomizedDelaySec=10m
```

The timer schedules a nightly run around **03:00 local server time**, with up to ten minutes of randomized delay. `Persistent=true` allows a missed calendar run to be caught up when the timer becomes active again. Confirm the actual next trigger with `systemctl list-timers`; do not treat a past predicted trigger as a completed run.

The service is `Type=oneshot`, runs `/usr/local/sbin/backup-to-unas` as root (no explicit `User` or `Group`), waits on network/mount dependencies, and uses `Nice=10`, best-effort I/O scheduling, priority `7`.

**`inactive (dead)` between successful runs is normal.** After the 2026-09-18 protection change, `unas-backup.timer` was verified `enabled`, `active`, and `waiting`. A later 2026-09-19 check verified the timer-triggered `unas-backup.service` run completed successfully at 03:04:05 PDT with exit status `0/SUCCESS`, including normal retention/prune processing; the timer remained active/waiting with its next trigger shown for 2026-09-20. This closes the earlier deferred check for the first scheduled run under share-root mode `0700`.

Direct `backup-to-nas` invocations do not update systemd's last service execution; they invoke the protected root script through sudo outside the service unit. The September 18 manual acceptance runs and the later successful timer-triggered run together verify both manual and scheduled execution through the protected root backup path.

Earlier service logs reported that Restic could not locate a cache because neither `HOME` nor `XDG_CACHE_HOME` was defined. The run still succeeded; the logs also warned that pruning without a cache may be slow. No cache fix was verified in this record.

## 9. Preserved Pre-Migration Retention and Prune Behavior

The script applies this policy after backup:

```text
restic forget --tag automatic --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune
```

This is the recorded policy, not a command to run as a diagnostic. It retains daily, weekly, and monthly recovery points; overlapping categories do not imply exactly 23 snapshots. Historical snapshots carry both `smart-hub` and `rotom` host names, and the audit showed separate retention groups for those hosts. Do not delete older snapshots simply because their host name is historical.

**Running the complete backup script manually also applies retention and prunes repository data.** This mattered during the Palworld account migration: the stopped-state snapshot `53c989ec` created at `23:00:33 PDT` was removed as a redundant same-day automatic snapshot when the post-migration run created `0d64ab6d` at `23:08:43 PDT` and applied `forget --prune`. A manual backup is therefore not guaranteed to remain as a separate same-day rollback point under the current policy. The repacking, index rebuilding, and pack removal were successful retention work; they do not constitute a full integrity check. Inspect current snapshots and review the policy before any intentional retention change; never manually delete repository files.

## 10. Pre-Migration Verification Evidence

Run these read-only checks **on the Rotom host as `jared`**, using `sudo` for protected repository access. No credential contents are displayed.

```bash
findmnt -T /mnt/nas-rotom-backup
systemctl list-timers unas-backup.timer --no-pager
systemctl status unas-backup.service --no-pager -l
systemctl show unas-backup.service -p Result -p ExecMainStatus
journalctl -u unas-backup.service -n 60 --no-pager
```

Verify that `findmnt` resolves to the intended NAS export before accessing the repository. Inspect logs locally; redact sensitive material before sharing any operational output.

```bash
sudo env \
  RESTIC_REPOSITORY=/mnt/nas-rotom-backup/rotom-restic-backup \
  RESTIC_PASSWORD_FILE=/etc/restic/unas-password \
  restic snapshots --compact

sudo env \
  RESTIC_REPOSITORY=/mnt/nas-rotom-backup/rotom-restic-backup \
  RESTIC_PASSWORD_FILE=/etc/restic/unas-password \
  restic ls e98243f5 /var/backups/system-info/sqlite
```

Expected staging entries are `home-assistant_v2.db`, `npm-database.sqlite`, and `zigbee.db`. The snapshot ID pins this historical verification; retention may eventually expire it. For ongoing checks, select a current snapshot belonging to Rotom from the snapshot list.

### Integrity-check status

**At the original audit point, no scheduled `restic check` was found in timer/cron/configuration searches, and the then-supplied checks contained no prior check history.** This is historical evidence only. Later JAR-24 and JAR-33 verification explicitly ran successful repository checks as documented above.

Snapshot listing establishes discoverability and path coverage, while SQLite `quick_check` validates staged database structure. On 2026-09-19, standard `restic check` passed and `restic check --read-data` read all 933 packs across 19 snapshots with no errors. A representative isolated restore also passed for both Palworld `Level.sav` files, Home Assistant data/configuration, and selected service Compose files. These tests materially strengthen recovery evidence but do not equal a full bare-metal restore or a complete end-to-end Palworld server recovery rehearsal.

## 11. Preserved Pre-Migration Restore Kit and Legacy Warning

The JAR-32 whole-share mirror carries this same legacy kit into the current `Rotom_Restic_Backup` share at `/mnt/nas-rotom-restic-backup/rotom-linux-home-server-backup-restore-kit`; that current-path statement follows from the verified exact mirror and was not a separate content-open test. The original pre-migration location remains:

Verified kit directory:

```text
/mnt/nas-rotom-backup/rotom-linux-home-server-backup-restore-kit
```

It contains:

- `Linux_to_UniFi_UNAS_Backup_Guide.pdf`
- `Linux_to_UniFi_UNAS_2_Restic_Restore_Guide.pdf`
- `rotom-linux-home-server-backup-restore-kit.zip`
- `.DS_Store` metadata

The inspected ZIP contains the enclosing directory, **the two PDFs and `.DS_Store` only**. It is documentation, not a self-contained recovery bundle: it does not include the Restic repository, repository credential, backup script, or systemd units. Do not assume it can recover Rotom by itself. The kit lives on the same NAS backup mount and is excluded from this host backup; an independent accessible copy is prudent, but none was verified here.

**Legacy restore-guide warning:** the restore PDF still references `smart-hub`, `/mnt/unas-backup`, `Smart_Hub_Backup`, and `/home/smart-hub`. Do not copy its commands verbatim onto current Rotom. Use the current repository and mount paths in this document, and preserve the actual service-account layout. A blanket hostname substitution is not enough to update account, mount, network, and ownership assumptions.

## 12. Recovery Procedure — Current Paths with Preserved Historical Evidence

This is the recovery order supported by the audit, with current Rotom paths. It is a staged recovery plan, not a claim that a bare-metal restore was tested.

1. **Prepare the replacement Linux host.** Establish console/SSH access, networking, correct time, Restic, and Docker/Compose as needed. Keep application services from starting against empty data directories.
2. **Access the NAS and repository.** Restore the current NFS mount arrangement and confirm the actual NAS export with `findmnt`. Obtain the repository credential through its secure recovery process without displaying it. Do not initialize a new repository over the existing one.
3. **Choose a recovery point.** List snapshots and select the intended host/date explicitly. `fbe1838e` is the final verified pre-migration recovery point; `5b4ed601` is the first verified VM-era snapshot; `f666d63c` is the current verified guest Restic recovery point from `2026-09-27 13:30:28 PDT`. Historical JAR-33 snapshot `74d3b0ca` was removed by normal same-day retention and is no longer available. For whole-VM recovery, the sole current verified VZDump is `vzdump-qemu-100-2026_09_27-14_08_51.vma.zst`. No snapshot/archive ID is a permanent recommendation to restore that date—choose the state appropriate to the incident.
4. **Restore into a new restricted staging directory.** Check free space, inspect the selected snapshot's paths, and recover files away from production. Validate staged databases and the required application data before promoting anything.
5. **Recreate account and storage relationships.** Preserve numeric UID/GID ownership for the current `infra`, `media`, `game`, `smarthome`, `documents`, `downloaders`, `web`, `filesync`, `apps`, `auth`, and personal accounts; older snapshots can contain historical account names including `rotom`, `smart-home`, `web-host`, and `downloads`, which must be reconciled deliberately during restore. The current Palworld identity is `game` at `995:5001` with home `/home/game`; older snapshots can legitimately contain the historical `game-server` name and `/home/game-server` path, so reconcile account/path metadata deliberately before service startup. For the current media architecture, reconcile older GID `129` account records, file groups, and Compose settings with UID `127` / primary GID `5000` before starting the stack. Preserve existing mixed NAS owner UIDs. The current Restic share root presents as numeric `988:988` mode `0700` while mounted; that is NAS-side ownership, not literal `root:root`. Verify root-run repository access and the restored export's identity mapping/options before resuming backups rather than assuming the retired share's export flags automatically apply. Re-test intended ordinary-user denial after reconstruction. GID `5001` is the current Game service group and must be restored for `game`; current service storage GIDs are Infra `5002`, Smarthome `5003`, Documents `5004`, Downloaders `5005`, Web `5006`, Filesync `5007`, Apps `5008`, and Auth `5009`; restore/rebuild those numeric identities deliberately rather than replaying older snapshots' group metadata. Avoid broad recursive ownership changes.
6. **Restore main data and selectively reconcile `/etc`.** Recover relevant `/home`, `/root`, `/opt`, `/srv`, and `/usr/local` data. Review systemd units, fstab, networking, users/groups, and host-specific configuration before applying them. Do not blindly overwrite a fresh host's entire `/etc`.
7. **Recover Docker storage and application state.** Review each existing `compose.yaml`, persistent bind mounts, named volumes, networks, and credentials. Keep Docker stopped while replacing its volume data. Recreate images from recorded definitions/inventory as appropriate; inventory text is not an image backup.
8. **Recover NPM and Home Assistant databases while their services are stopped.** Preserve the current destination first. Use the staged SQLite copies and map NPM's `npm-database.sqlite` back to `data/database.sqlite`. Restore each application's configuration and related files from the selected recovery point. Handle existing SQLite WAL/SHM sidecars as part of a deliberate offline replacement; do not mix stale sidecars with a replacement database. Restore service-appropriate ownership and validate before startup.
9. **Recover each Palworld world separately.** Keep the target server stopped, preserve its current tree, inspect the chosen save/archive, and restore to that server's exact `/home/game/docker/palworld-server-*/Pal/Saved/SaveGames` layout. Retain current UID/GID `995:5001`, world/player identifiers, and ownership. If restoring data from an older snapshot that records GID `985`, reconcile that historical metadata to current GID `5001` before starting Palworld. If restoring a pre-migration snapshot that contains `/home/game-server`, stage and reconcile it to the current `game` account/home rather than starting containers against empty replacement paths. Never recreate or reinitialize a server in a way that overwrites an existing world.
10. **Start and verify services in dependency order.** Establish storage and Docker networking, then infrastructure/reverse proxy, applications, and game servers. Restore qBittorrent at `/home/downloaders/docker/qbittorrentvpn` as `901:5005`, recreate its protected `.env` without exposing credentials, restore `/mnt/nas-downloaders/.rotom-qbt-nas-ready` and the `create_host_path: false` downloader binds, restore/enable the two read-only compatibility bindfs services, and confirm current `/mnt/nas-downloaders -> Downloader/.data` before starting it. Restore Prowlarr at `/home/downloaders/docker/prowlarr` as `901:5005`. Restore Gamarr at `/home/game/docker/gamarr` as `995:5001`. Verify NPM routes/certificates, Home Assistant configuration/history and integrations, Homebridge, media paths, both websites, and both Palworld worlds. Confirm qBittorrent's VPN and kill-switch protections before resuming downloads.
11. **Resume and verify backups deliberately.** Review the recovered script, include/exclude files, credential-file permissions, timer, and mount dependencies. Current guest automation uses enabled/active `rotom-restic-backup.timer` on the daily `03:00` schedule with up to ten minutes randomized delay; restore that state only after validating the repository, mount, script, and status API on the recovered host. Verify a successful scheduled run, fresh snapshots, all three SQLite staging files, and expected retention behavior. Remember that a full backup run also executes retention/prune.

Do not restore directly over production data as a first test. Before any replacement, inspect the current destination and retain a reversible copy. Use `vim` for manual configuration edits on Rotom. This guide deliberately avoids generic overwrite commands because safe promotion depends on the actual restored host, ownership, and application state.

## 13. Disaster-Recovery Boundaries and Known Gaps

Primary evidence: “Review Rotom Architecture,” conversation ID `6aa8ca46-feac-83e8-ac95-81025db541f5`, including the final complete listing of snapshot `e98243f5`; uploaded reports `rotom-backup-restore-audit.txt`, `rotom-backup-verification.txt`, `rotom-backup-verification-remaining.txt`, `rotom-backup-final-check.txt`, `rotom-backup-last-check.txt`, `rotom-backup-fix-audit.txt`, and `rotom-backup-hardening-results.txt`.

UID/GID and NAS-access update evidence: “Torrent Seeding Explanation,” conversation ID `6aaa2d2e-04a8-83e8-a95f-d9c69fd8512e`. This update records the media changes and the proposed account-to-NAS pattern; it does not verify a new backup run, restore, or storage migration.

Shared manual-access evidence: “Add Fran Backup Access,” conversation ID `6aaa75d9-7a10-83e8-bffa-6acab47b3765`. Supplied output verifies the shared launcher and Fran's exact sudo authorization.

Backup-access protection evidence: the September 18, 2026 chat supplied live Rotom and UNAS output for export options, UID/GID mappings, Btrfs `noacl`, the `0770`→`0700` share-root change, complete ordinary-account denial, root mount/repository access, Jared and Fran manual backups, resulting snapshots, retention/prune, and timer/service state. Later September 18 migration output supplied the `game-server` → `game` account/home change, Palworld save-integrity comparisons, current container paths/health, and post-migration Restic snapshot verification.

Resolved: exact include/exclude rules, explicit system-info inclusion, both Palworld save paths and Restic coverage, actual Palworld archives, SQLite staging checks, and staging inclusion in the later completed snapshot.

The September 18 protection work verified both human launchers, root repository access, retention/prune, and the armed timer. The 2026-09-19 follow-up established earlier full-pack/read and representative-restore evidence; JAR-21 later produced snapshot `a6d86b20`. JAR-24 remains the final pre-migration Restic migration-gate recovery evidence: snapshot `fbe1838e` at `2026-09-25 18:14:22 PDT` completed normal retention/prune, passed standard and full `--read-data` checks across all 971 packs, and passed a representative isolated restore with restored SQLite integrity checks. JAR-25 then added the full raw NVMe rollback image on the Shared Drive. JAR-32 added first VM-era snapshot `5b4ed601`; JAR-66/JAR-33 then completed the separate Proxmox whole-VM path and accepted the Phase B **Rotom Virtualization Baseline**, and JAR-67 established daily `05:00` whole-VM backups with `7 daily / 4 weekly / 6 monthly` retention. Later on 2026-09-27, current operational recovery points were refreshed: guest Restic snapshot `f666d63c` was created and verified with 21/21 repository checks, guest `rotom-restic-backup.timer` was re-enabled/active, all older VMID 100 VZDump artifacts were intentionally deleted, and sole current archive `vzdump-qemu-100-2026_09_27-14_08_51.vma.zst` was created and passed full VMA verification. Remaining work is narrower: observe the first unattended `rotom-daily` run, decide whether the new sole VM archive should be protected from routine pruning, later converge the broader JAR-47 recovery policy, observe direct NAS root-mode/GID persistence across a future UniFi Drive restart/update, rehearse an actual bare-metal restore from the JAR-25 image, and perform a full end-to-end Palworld server restore rehearsal. The disposable JAR-32 helper `/tmp/backup-to-unas-no-retention` was freshly verified absent on 2026-09-27, so that cleanup item is resolved. NAS source data mounted under `/mnt`—including Media, Game, the eight service shares, Shared Drive, and the backup mount itself—is outside the host Restic source scope and requires independent protection or repository-specific recovery handling as applicable.

### JAR-19 website migration — backup implications

JAR-19 copied both website trees from `/home/web-host/docker` to `/home/web/docker`, verified Aloha from the new Compose working directory, preserved Jared Wines stopped, and then removed the legacy `web-host` user/group/home after dependency guards passed. A root-only rollback copy of the complete former home and identity evidence remains at `/root/jar-19-20260921-024634`, which is inside the existing `/root` Restic source scope. Final snapshot `e886c55d` was saved at `2026-09-21 02:47:55 PDT` after account retirement and final website verification. Normal retention removed intermediate same-day snapshot `2c22607e`; do not manually manipulate Restic internals to recover that pruned snapshot.

## 14. Historical Consolidated Verification — 2026-09-19

- Latest path-verified snapshot: `f0d51eaa` at `2026-09-19 06:39:30 PDT`.
- `restic check`: PASS.
- `restic check --read-data`: PASS; 933/933 packs and 19/19 snapshots checked with no errors.
- Representative restore: PASS into a restricted `/tmp` staging tree; both current Palworld worlds restored non-empty with `995:5001`, alongside selected Home Assistant and service configuration files; scratch data was deleted afterward.
- This is verified restore evidence, but it is not a claim that a replacement host, full application stack, or Palworld server was booted from restored data.
- A later comprehensive 2026-09-19 read-only audit reconfirmed `/etc/restic/include.txt` and `/etc/restic/exclude.txt`, `unas-backup.timer`, `Result=success` / `ExecMainStatus=0` for the latest inspected service run, repository ID visibility with 19 snapshots, and `f0d51eaa` as the latest listed snapshot at `06:39:30 PDT`.
- At the 2026-09-19 bare-metal checkpoint, `unas-backup-status-api.service` was enabled/active on `0.0.0.0:8787`; logs showed Homepage requests from `172.24.0.3`, and UFW permitted TCP `8787` from `172.24.0.0/16`. **Current VM note:** JAR-31 verified the status API restored and active, but UFW is not installed in the Debian VM; do not carry the old UFW rule forward as current state.
- This re-verification did not create, forget, prune, or restore a new snapshot, so it does not change the existing full-DR and full-Palworld-recovery limitations.

## 14B. JAR-6 Backup-Boundary Verification — 2026-09-21

JAR-6 did not change backup architecture. After the normal Rotom reboot, read-only closeout checks verified `/mnt` remains explicitly excluded, no new JAR-6 NAS path appears in `include.txt`, the existing Restic repository directory is present, `unas-backup.service` still declares `RequiresMountsFor=/mnt/nas-rotom-backup`, and `unas-backup.timer` remains enabled and active. Backup source/root metadata and Shared Drive source/root metadata were unchanged. No manual backup or retention/prune operation was run merely as a JAR-6 diagnostic.

## 14A. Historical Identity-Migration Backup Verification — 2026-09-21

- Pre-change backup completed and created snapshot `cd230c0a` before the affected Compose projects were stopped.
- The normal post-change `backup-to-nas` run created snapshot `b67540b0` at `2026-09-21 02:23:25 PDT`.
- The retention policy then removed same-day snapshot `cd230c0a` and pruned normally; this was expected retention behavior, not a manual repository deletion.
- Current host backup paths include `/home/infra`, `/home/smarthome`, `/home/documents`, `/home/downloads`, `/home/web`, `/home/filesync`, `/home/apps`, and `/home/auth` through the unchanged `/home` backup root.
- Rollback evidence is retained at `/root/rotom-identity-batch-20260921-021305`, within the unchanged `/root` backup source.
- Application checks passed before the post-change backup: Homepage, Home Assistant, Homebridge, Prowlarr, and qBittorrent returned HTTP 200; qBittorrentVPN had WireGuard `wg0`.

## 14C. Historical JAR-21 Final Backup Verification — 2026-09-21

- Pre-cleanup `backup-to-nas` completed successfully before obsolete rollback paths were deleted.
- After cleanup and final health verification, `backup-to-nas` completed again and retained snapshot `a6d86b20` at `2026-09-21 19:44:55 PDT`.
- Snapshot sources include `/home`, `/etc`, `/root`, `/usr/local`, `/var/backups/system-info`, and `/var/lib/docker/volumes`; therefore active `/home/downloads/docker/{prowlarr,qbittorrentvpn}`, the bindfs systemd units, and Arcane named-volume database state are covered.
- `/mnt` remains excluded. `/mnt/nas-downloads/torrents`, Media/Game libraries, and other NAS source data are not protected by host Restic.
- Final live checks showed no failed systemd units and system state `running`.

## 15. Historical Backup Verification — 2026-09-18

At the September 18 stage, the backup share root was verified `988:988` mode `0700`. The original ordinary-account matrix included the then-named `game-server` account at UID/GID `995:985`. A later 2026-09-19 matrix superseded that identity-specific limitation and verified current `game` (`995:5001`) plus all six other ordinary named accounts denied read/write/traverse while root retained access.

During the Palworld migration, a stopped-state backup produced snapshot `53c989ec` at 23:00:33 PDT. After the verified rename to `game`, home move to `/home/game`, byte-for-byte `.sav` comparison, Compose recreation, and healthy restart, `backup-to-nas` created snapshot `0d64ab6d` at 23:08:43 PDT. The normal retention policy then removed the redundant same-day `53c989ec` snapshot and completed prune successfully. This remains historical recovery evidence; path-verified snapshot `f0d51eaa` from 2026-09-19 supersedes it as the latest snapshot explicitly verified in this documentation.

The timer was enabled and active/waiting at this historical checkpoint. A later 2026-09-19 timer-triggered run completed successfully under share-root mode `0700`, closing that deferred scheduled-run check. Persistence of mode `0700` across a future UNAS/UniFi Drive restart or update remains unverified.

## 16. Historical Backup Verification — 2026-09-15

A read-only repository listing after the scheduled 03:01 run showed 17 snapshots. The latest is afc91edf, created 2026-09-15 03:01:20 PDT, tagged automatic for rotom. The backup service recorded Result=success and exit status 0. This supersedes the historical snapshot count and latest-snapshot statements above. The full-restore limitation remains unchanged.

## 17. Reconstruction-Critical Information

- Restic repository: `/mnt/nas-rotom-restic-backup/rotom-restic-backup` on NAS `Rotom_Restic_Backup`.
- Restic manual launcher: `/usr/local/bin/backup-restic-to-nas`; privileged implementation: `/usr/local/sbin/rotom-restic-backup`.
- Jared exact-command privilege rule: `/etc/sudoers.d/rotom-restic-backup` -> `/usr/local/sbin/rotom-restic-backup`.
- Restic include/exclude definitions: `/etc/restic/include.txt` and `/etc/restic/exclude.txt`; the script also supplies `/var/backups/system-info` explicitly.
- Restic service/timer: `/etc/systemd/system/rotom-restic-backup.service` and `/etc/systemd/system/rotom-restic-backup.timer`; timer enabled/active, daily `03:00`, persistent, randomized delay up to 10 minutes.
- Restic status API: `/usr/local/sbin/rotom-restic-backup-status-api` + `/etc/systemd/system/rotom-restic-backup-status-api.service`; Homepage endpoint `http://192.168.1.69:8787/backup-status`.
- Credential location: `/etc/restic/nas-password`; contents must never be printed or copied into documentation.
- Current verified guest snapshot: `f666d63c` at `2026-09-27 13:30:28 PDT`; post-run repository check passed 21/21 snapshots.
- Proxmox manual helper: `/usr/local/bin/backup-proxmox-to-nas`.
- Proxmox automatic scheduler: native PVE job `rotom-daily` in `/etc/pve/jobs.cfg`, VMID 100, `05:00`, snapshot + zstd, `repeat-missed=0`, retention 7 daily / 4 weekly / 6 monthly. Do not replace it with a systemd timer.
- Current sole verified VM archive: `/mnt/pve/nas-rotom-proxmox-backup/dump/vzdump-qemu-100-2026_09_27-14_08_51.vma.zst`, `45,424,272,161` bytes, full VMA verify exit `0`; no protection marker was present in the final observed listing.
- Host Restic excludes `/mnt`; NAS Media/Game/Shared Drive content therefore requires its own protection policy unless safely reproducible.

## 18. Proposed Improvements

No new backup architecture is adopted by this restructure. Future improvements should remain clearly Proposed until implemented and verified. Existing outstanding recovery tests remain under **Disaster-Recovery Boundaries and Known Gaps**.

## 19. Related Documentation

- `04-NAS-and-Storage.md` — canonical backup mount/export and NAS storage boundary.
- `06-Maintenance-and-Automation.md` — canonical recurring schedule/maintenance context.
- `07-Users-and-Permissions.md` — canonical manual-access authorization and account privilege model.