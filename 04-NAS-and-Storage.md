# 04 - Rotom NAS and Storage Architecture

**Documentation set:** Rotom Project Documentation  
**Document role:** Canonical source for NAS exports, NFS mounts, storage layout, automount behavior, and storage contracts  
**Hosts:** PVE hypervisor `pve`, Debian VM `rotom`, and UniFi UNAS 2  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-09-29 — empty Downloads NAS boundary retired
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `05-Backup-and-Restore.md`, `07-Users-and-Permissions.md`, `08-Rotom-Directory-Tree.txt`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## 1. Purpose and Scope

This document records the current Phase B storage architecture and preserves the pre-migration Rotom storage contracts needed for restoration. It covers current Proxmox/VM local storage plus the historical UniFi NAS exports, mount options, `/mnt/nas-*` contracts, automount behavior, permissions, Docker binds, media/game paths, hardlink behavior, and service dependencies that later tickets must restore deliberately.

### Evidence provenance

**Project:** Rotom-Home-Server  
**Document:** `04-NAS-and-Storage.md`  
**Baseline evidence:** 2026-09-14 22:46 PDT; read-only update 2026-09-15  
**Documentation updated:** 2026-09-27 — current Proxmox VZDump archive set and disconnect-safe manual path recorded; storage boundaries unchanged  
**Status:** All twelve required application/storage NFS mounts remain guest-only systemd automounts. Guest `Rotom_Restic_Backup` remains unchanged. JAR-68 normalized the two PVE-only backup exports to `PVE_Restic_Backup` for host-config Restic and `Rotom_VM_Backup` for whole-VM VZDump; canonical mounts are `/mnt/nas-pve-restic-backup` and `/mnt/pve/nas-rotom-vm-backup`. The PVE host still has no Rotom application NFS mounts.

The original audit was read-only. It did not recursively enumerate the NAS, modify mounts or permissions, restart containers, inspect secrets, traverse Restic repository internals, or create a hardlink test file. Later user-supplied checks, completed media GID changes, and the 2026-09-18 backup-share access-protection deployment are recorded below; they are separate from that audit. Unchanged capacity, mount, and systemd observations retain their original evidence dates.

## 2. Current Storage State — 2026-09-29 JAR-47 final recovery policy

The Rotom guest storage contract remains unchanged: twelve NFS systemd automounts plus read-only Media/Game bindfs compatibility views. PVE has only backup-specific NAS mounts; it does not host the guest application shares.

### NAS protection classification — JAR-47

UniFi Drive snapshots are enabled for every NAS drive. The supplied Media settings evidence records a **daily 12:00 AM UNAS-local** snapshot schedule with a **16-snapshot** limit. `Media/library` (including `library/games`) is authoritative NAS data and uses this snapshot policy for local rollback. `Media/torrents` is transient/reproducible payload and is deliberately outside the authoritative-library claim. The current Documents, Gameserver, Customapps, Filesync, Auth, Web, Smarthome, and Infra reserved shares may remain empty; empty boundaries are not represented as backup-protected data. Paperless remains VM-local, so `nas-documents` has no current authoritative Paperless content. Snapshot retention protects against ordinary deletion/corruption but remains on the same UNAS appliance and is not off-site protection.

### PVE host-config Restic storage

- NAS share/export: `PVE_Restic_Backup/.data`, authorized to PVE `192.168.1.68`.
- Host mount: `/mnt/nas-pve-restic-backup`, `/etc/fstab` systemd automount.
- Repository directory: `/mnt/nas-pve-restic-backup/pve-restic-backup`.
- Repository ID: `8ca0218c645dc2968c2b21d67dbda3840794d8bc9c157c0515a85af30eb3d0e3`.
- Superseded local mount `/mnt/nas-proxmox-restic-backup` was verified empty/unmounted/unreferenced and removed with `rmdir`.

### Whole-VM PVE storage

- NAS share/export: `Rotom_VM_Backup/.data`, authorized to PVE `192.168.1.68`.
- PVE storage ID: `nas-rotom-vm-backup`.
- Mount: `/mnt/pve/nas-rotom-vm-backup`; PVE-managed NFS, `content backup`.
- Six earlier VMID 100 backups were deliberately removed after exact storage/VM safety checks. The storage now contains the fresh verified baseline `vzdump-qemu-100-2026_09_28-00_09_19.vma.zst`, `47,415,540,796` bytes. zstd integrity and full `vma verify` passed; VM100 remained running. The archive is currently unprotected and participates in normal retention.

### Guest storage and post-reboot acceptance

All twelve guest NFS automounts and both read-only bindfs views passed final post-reboot verification. `/mnt/nas-downloaders/.rotom-qbt-nas-ready` remained valid; qBittorrentVPN saw the expected mounts and WireGuard `10.2.0.2/32`. Current finished media/library and torrent/download separation remains unchanged.

### Retired empty `Downloads` boundary — 2026-09-29

The separate `Downloads/.data` drive was mounted for final inspection and contained 0 bytes with no entries. No active workload configuration referenced `/mnt/nas-downloads` or `Downloads/.data`. Its precise Rotom fstab entry, generated mount/automount, and empty local mountpoint were removed; Jared then deleted the empty NAS drive. This retirement does not affect active `Downloader/.data` at `/mnt/nas-downloaders`, or retained Game rollback data.

The old PVE backup share names are no longer part of active PVE configuration. Historical sections below may preserve their dated names as evidence.

## 2A. Preserved Pre-Migration NAS and Storage Overview

At the final pre-migration baseline, Rotom used one local NVMe system disk and NFS storage from the UniFi NAS at `192.168.1.70`. That storage model had twelve fstab-backed NAS mountpoints: existing Media, Game, Backup, and Shared Drive plus eight JAR-6 per-service storage boundaries. A normal Rotom reboot on 2026-09-21 verified the eight new automounts and their exact NFS sources return correctly.

| Mount | Source | Filesystem | Root metadata | Primary purpose |
|---|---|---|---|---|
| `/mnt/nas-media` | `192.168.1.70:/var/nfs/shared/Media` | NFSv3 `sec=sys` | `988:5000` mode `2770` | Final movie/show/game library data; JAR-52 game content is under `library/games`; old torrent tree removed by JAR-21 |
| `/mnt/nas-game` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Game/.data` | NFSv3 `sec=sys` | `988:5001` mode `2770` | Compatibility Game-library source retained intact for JAR-52 rollback; Palworld remains local VM state |
| `/mnt/nas-infra` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Infra/.data` | NFSv3 `sec=sys` | `988:5002` mode `2770` | Infra storage boundary; currently no workload data |
| `/mnt/nas-smarthome` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Smarthome/.data` | NFSv3 `sec=sys` | `988:5003` mode `2770` | Smarthome storage boundary; HA/Homebridge remain local |
| `/mnt/nas-documents` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Documents/.data` | NFSv3 `sec=sys` | `988:5004` mode `2770` | Documents storage boundary; currently empty |
| `/mnt/nas-downloads` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Downloads/.data` | NFSv3 `sec=sys` | `988:5005` mode `2770` | Active centralized torrent storage; canonical tree `/mnt/nas-downloads/torrents` |
| `/mnt/nas-web` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Web/.data` | NFSv3 `sec=sys` | `988:5006` mode `2770` | Web storage boundary; websites remain local |
| `/mnt/nas-filesync` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Filesync/.data` | NFSv3 `sec=sys` | `988:5007` mode `2770` | Filesync storage boundary; currently empty |
| `/mnt/nas-apps` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Apps/.data` | NFSv3 `sec=sys` | `988:5008` mode `2770` | Apps storage boundary; currently empty |
| `/mnt/nas-auth` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Auth/.data` | NFSv3 `sec=sys` | `988:5009` mode `2770` | Auth storage boundary; currently empty |
| `/mnt/nas-rotom-backup` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Rotom_Home_Server_Backup/.data` | NFSv3 | `988:988` mode `0700` | Restic repository and recovery material |
| `/mnt/nas-shared-drive` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Shared_Drive/.data` | NFSv3 | `988:988` mode `0770` | General Shared Drive |

Seven JAR-6 service shares remain boundary-only. JAR-21 activated the Downloads share for authoritative torrent data at `/mnt/nas-downloads/torrents`. Host Restic excludes `/mnt`, so Downloads torrent data and the other NAS trees are outside the existing host backup source scope.

### JAR-52 Media game-library move — 2026-09-28

JAR-52 copied, without deletion, `/mnt/nas-game/library/games/{pc,roms}` to `/mnt/nas-media/library/games/{pc,roms}`. The target directories are owned `995:5000`, mode `2775`, with setgid, so the retained Gamarr UID 995 uses the Media primary storage GID 5000. The one-file source and target inventories were each 223,408,925 bytes and their SHA-256 manifests matched. Gamarr now binds only the Media games directory at its unchanged in-container library root `/game/library/games`; the existing read-only downloader torrent view remains at `/game/torrents`.

The old Game library remains available and unmodified as rollback material; no Palworld save, `/mnt/nas-gameserver` path, qBittorrentVPN bind, downloader payload, or NAS export was changed. Both the Media and Game mounts had 4.1 TiB free at verification. Guest Restic excludes `/mnt`; JAR-47 subsequently established the separate Media snapshot policy described above.

### JAR-56 Media torrent topology — 2026-09-28

The active qBittorrentVPN payload bind is now `/mnt/nas-media/torrents` at both `/media/torrents` and `/game/torrents`; its v2 stack/appdata are `/srv/rotom/stacks/downloader/qbittorrentvpn` and `/srv/rotom/appdata/downloader/qbittorrentvpn`. The Downloader source tree remains unchanged as rollback material. The copied trees matched SHA-256 manifests at 10,482,680,947 bytes across three files. qBittorrentVPN retains UID 901 with primary GID 5000 for Media writes, its WireGuard design, and its host port.

The former empty Media torrent directory was corrected to `root:5000` mode `2775`; the Media sentinel is `/mnt/nas-media/.rotom-qbt-media-ready` at `901:5000` mode `2750`. Compose requires that sentinel with `create_host_path: false`. The enabled 15-second `rotom-qbittorrent-media-guard.timer` checks the underlying NFS mount and sentinel, then stops a running qBittorrentVPN if either is absent. Radarr/Sonarr/Gamarr now consume the Media torrent tree. Real filesystem hardlink probes passed. Jared deferred authenticated application-driven import and post-import seeding validation for later manual completion; retain all Downloader paths, bindfs views, and source data until that validation and JAR-47 policy review.

### Preserved pre-migration service-home NAS convenience links — 2026-09-21

Each operational service account now has a local home-directory shortcut named `nas-<service>` pointing to its corresponding canonical NAS mount:

| Service account | Convenience path | Canonical target |
|---|---|---|
| `media` | `/home/media/nas-media` | `/mnt/nas-media` |
| `game` | `/home/game/nas-game` | `/mnt/nas-game` |
| `infra` | `/home/infra/nas-infra` | `/mnt/nas-infra` |
| `smarthome` | `/home/smarthome/nas-smarthome` | `/mnt/nas-smarthome` |
| `documents` | `/home/documents/nas-documents` | `/mnt/nas-documents` |
| `downloads` | `/home/downloads/nas-downloads` | `/mnt/nas-downloads` |
| `web` | `/home/web/nas-web` | `/mnt/nas-web` |
| `filesync` | `/home/filesync/nas-filesync` | `/mnt/nas-filesync` |
| `apps` | `/home/apps/nas-apps` | `/mnt/nas-apps` |
| `auth` | `/home/auth/nas-auth` | `/mnt/nas-auth` |

The links were initially created with the generic name `nas`, then renamed in place to the explicit names above. Supplied verification confirmed all ten final paths are symbolic links to the same targets and that each intended service account can traverse its own shortcut; the final verification ended with `ALL NAS SHORTCUT RENAMES: PASS`. These are convenience aliases only. Keep `/mnt/nas-*` as the canonical path in Compose files, systemd units, backup/recovery logic, qBittorrent storage guards, diagnostics, and documentation. Creating and renaming the links did not change NFS exports, fstab/systemd automount behavior, NAS permissions, Docker binds, or application data.

### Preserved pre-migration JAR-21 centralized Downloads torrent architecture — 2026-09-21

Active torrent storage is now `/mnt/nas-downloads/torrents`. The old `/mnt/nas-media/torrents` and `/mnt/nas-game/torrents` rollback trees were removed only after the Media tree checksum-matched Downloads, the Game tree was verified file-empty, the active qBittorrent mounts were verified to reference Downloads, and a Restic backup completed successfully. Final closeout verified the old trees absent and the active Downloads tree present at 9.8G.

qBittorrentVPN uses `/mnt/nas-downloads/torrents` read/write at both `/media/torrents` and `/game/torrents`, preserving the existing category-visible paths without exposing Media/Game libraries to qBittorrent. Its fail-closed startup sentinel is `/mnt/nas-downloads/.rotom-qbt-nas-ready`, mounted read-only at `/run/rotom-nas-downloads` with `create_host_path: false`.

Two persistent bindfs services expose identity-mapped, read-only views of the Downloads share:

- `rotom-downloads-media-ro.service`: `/mnt/nas-downloads` -> `/mnt/nas-downloads-media-ro`, forced `media:media`, read-only.
- `rotom-downloads-game-ro.service`: `/mnt/nas-downloads` -> `/mnt/nas-downloads-game-ro`, forced `game:game`, read-only.

Radarr and Sonarr mount `/mnt/nas-downloads-media-ro/torrents` at `/media/torrents:ro`; Gamarr mounts `/mnt/nas-downloads-game-ro/torrents` at `/game/torrents:ro`. Radarr/Sonarr retain their broad `/mnt/nas-media -> /media` library bind. Since JAR-52, Gamarr instead has the narrow `/mnt/nas-media/library/games -> /game/library/games` library bind; the old `/mnt/nas-game/library/games` is rollback-only. These current Downloads-to-library paths still cross filesystems, so imports are copies rather than hardlinks until JAR-56. The end-to-end Arr workflow was accepted by the user after JAR-21 cutover.

## 3. Current UniFi NAS / NFS Server Contract and Preserved History

### JAR-55 reserved v2 domain boundaries — 2026-09-28

The guest now has fifteen fstab-backed systemd automounts. JAR-55 added the empty reserved exports `Gameserver/.data`, `Downloads/.data`, and `Customapps/.data`, authorized only to `192.168.1.69`, at `/mnt/nas-gameserver`, `/mnt/nas-downloads`, and `/mnt/nas-customapps`. They use NFSv3 `sec=sys`, `_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s`, roots `988:5001`, `988:5005`, and `988:5008`, and mode `2770`. Each owning identity can write while unrelated service identities are denied. `Game`, `Downloader`, and `Apps` remain active compatibility shares; no data/workload was moved. Guest Restic excludes `/mnt`, so these NAS-resident boundaries require a separate NAS protection policy.

**NFS server:** `192.168.1.70`

### Current NAS NFS management observations — JAR-66

During JAR-66 the NAS reported UniFi Drive `4.4.9`. The live NFS export state and the Drive-managed definitions agreed after correction: `Rotom_Proxmox_Backup` was exported only to `192.168.1.68`, while `Rotom_Restic_Backup` remained exported only to `192.168.1.69`. The management files observed under `/etc/exports.d` include paired `shared-*.json` and `shared-*.exports` definitions; `udcd.service` identifies itself as the **UniFi-Drive Config Daemon**, `nfs-config.service` as **Preprocess NFS configuration**, and `nfs-server.service` is the active NFS server unit on this build.

Treat `/etc/exports.d` as UniFi Drive-managed state, not as a hand-maintained configuration interface. JAR-66 made no manual edit to these files. For troubleshooting, compare the Drive-managed definition with `showmount -e 127.0.0.1` and `exportfs -v` on the NAS, then verify client visibility with `pvesm scan nfs 192.168.1.70` from Proxmox. The exact UI sequence Jared ultimately used to correct the client authorization was not captured, so the RPD records the verified final state and diagnostic contract rather than inventing a UI procedure.

JAR-29 repeated live export discovery from the new VM with `/usr/sbin/showmount -e 192.168.1.70` rather than inferring paths from UI names. The JAR-29 advertised-export capture included both `Downloads/.data` and `Downloader/.data`. JAR-31 current `showmount` output for the download name showed `Downloader/.data` exported to `192.168.1.69`, and the live guest now mounts that source. The older list below is retained as the JAR-29 export-discovery snapshot:

```text
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Infra/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Smarthome/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Documents/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Downloads/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Downloader/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Web/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Filesync/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Apps/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Auth/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Game/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Media/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Rotom_Home_Server_Backup/.data
/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Shared_Drive/.data
```

The historical bare-metal Media mount deliberately remained on `192.168.1.70:/var/nfs/shared/Media` through JAR-22. JAR-29 live discovery from the new VM confirmed `Media/.data` is exported and the VM uses that `.data` source. For downloads, JAR-31 now establishes `Downloader/.data` as the active current export/mount; the former “separate Downloader export unused” statement is superseded.

### Rolled-back JAR-9 `Game_Servers` share

`/mnt/nas-game-servers` remains absent from `/etc/fstab` and unmounted. During the JAR-6 UNAS audit the former `Game_Servers/.data` path was observed absent on the NAS rather than merely unexported. No JAR-6 command deleted or reinitialized that path; the documentation records the observed as-built deviation only. The active Game storage contract remains `/mnt/nas-game` backed by `Game/.data`.

## 4. Preserved Pre-Migration Mounts and Exports

The eight JAR-6 service mounts were verified as real NFS filesystems after explicitly triggering the corresponding systemd automount. Each uses NFSv3 with `sec=sys` and the exact source listed below:

| Mount | Exact source |
|---|---|
| `/mnt/nas-infra` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Infra/.data` |
| `/mnt/nas-smarthome` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Smarthome/.data` |
| `/mnt/nas-documents` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Documents/.data` |
| `/mnt/nas-downloads` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Downloads/.data` |
| `/mnt/nas-web` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Web/.data` |
| `/mnt/nas-filesync` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Filesync/.data` |
| `/mnt/nas-apps` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Apps/.data` |
| `/mnt/nas-auth` | `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Auth/.data` |

Existing sources remain `/var/nfs/shared/Media` for Media, `Game/.data` for Game, `Rotom_Home_Server_Backup/.data` for Backup, and `Shared_Drive/.data` for Shared Drive. Post-reboot verification re-established the new service mounts and later explicitly triggered Backup/Shared Drive, confirming those existing sources and root metadata were unchanged.

**Automount verification lesson:** querying an automount root with `findmnt -T` can return the `autofs` layer (`systemd-1`) rather than the underlying NFS mount. For Rotom commissioning/recovery checks, first traverse the path to trigger it, wait for the `.mount` unit, then query the exact mountpoint with `findmnt -M <mountpoint> -t nfs,nfs4` (or equivalent). Do not treat the expected `autofs` layer as an NFS failure.


### JAR-29 VM NAS restoration and acceptance — 2026-09-26

**Historical Phase B checkpoint:** JAR-29 recorded `/mnt/nas-downloaders` against `Downloads/.data`. JAR-31 live verification later superseded that **current-source** claim with `Downloader/.data`; the rest of this subsection remains the dated JAR-29 acceptance record.

JAR-29 installed the guest NFS client, recreated the ten service identities, and commissioned twelve fstab-backed systemd automounts inside the Debian VM. The Linux-side Downloads identity/path was intentionally renamed to `downloaders` / `/mnt/nas-downloaders` while preserving UID:GID `901:5005` and the existing UNAS `Downloads/.data` contents. Media now mounts the currently advertised `Media/.data` export.

All twelve mounts negotiated NFSv3. Positive functional tests created disposable files as each intended service identity and observed the exact UID:GID (`127:5000`, `995:5001`, `997:5002`, `126:5003`, `900:5004`, `901:5005`, `902:5006`, `903:5007`, `904:5008`, `905:5009`) with mode `0664`; every file was removed. An unrelated service identity was denied on every tested service root. Backup and Shared Drive root metadata remained `988:988` modes `0700` and `0770` respectively; no write test was performed inside the protected Restic repository or JAR-25 image tree.

A normal reboot verified all twelve automount listeners return active with no eager NFS mounts and that all twelve real shares then mount successfully on demand. A controlled unavailable-source reboot using temporary documentation-range `192.0.2.70` fstab sources verified the VM boots normally without NAS availability. Restoring the exact known-good fstab and reloading systemd recovered every mount on demand without another reboot. The current fstab uses `x-systemd.mount-timeout=30s`; the older `x-systemd.device-timeout=30s` option is preserved only as historical configuration because systemd 257 reported it ignored for NFS.

The service-home links were recreated as `/home/<service>/nas-<service> -> /mnt/nas-<service>`, including `/home/downloaders/nas-downloaders`. The old local `/mnt/nas-downloads` path is absent. JAR-30 has installed Docker and verified a disposable NFS bind only; production application restoration has not yet occurred, so historical qBittorrent/bindfs/sentinel paths under the old Downloads naming still require deliberate reconciliation later.

### JAR-22 twelve-mount revalidation — 2026-09-22

JAR-22 deliberately traversed each canonical NAS mountpoint to trigger its existing systemd automount and then queried the underlying NFS mount. All twelve current mount contracts were active and matched the documented sources. Media remained `192.168.1.70:/var/nfs/shared/Media`; Game, Infra, Smarthome, Documents, Downloads, Web, Filesync, Apps, Auth, Backup, and Shared Drive resolved to their documented UNAS `.data` exports. Every observed mount used NFSv3 with `sec=sys`.

Root metadata also matched the current storage contract: Media `988:5000` mode `2770`, Game `988:5001` mode `2770`, Infra through Auth `988:5002` through `988:5009` mode `2770`, Backup `988:988` mode `0700`, and Shared Drive `988:988` mode `0770`. Every corresponding `.automount` unit reported active. This was read-only validation of existing behavior; no fstab entry, export, permission, sentinel, bindfs view, or NAS object was changed.


### JAR-23 boot-time NFS/bindfs recovery — 2026-09-25

A September 23 reboot sequence exposed a timing failure without changing the underlying storage contracts. The systemd `.automount` units were established early, but Docker restore triggered `/mnt/nas-downloads`, `/mnt/nas-media`, and `/mnt/nas-game` before the real UNAS NFS mounts were usable. The two Downloads bindfs services failed their required Downloads mount dependency, while Docker recorded `no such device` / mount-source errors for NAS-backed containers. Later inspection showed all three canonical NFS mounts healthy and the storage data intact.

The Downloads safety contract was reverified before recovery: `/mnt/nas-downloads/torrents` existed as a setgid directory owned `901:5005`, `/mnt/nas-downloads/.rotom-qbt-nas-ready` existed as a setgid directory owned `901:5005`, qBittorrentVPN was running with the sentinel mounted read-only at `/run/rotom-nas-downloads`, and both persistent bindfs views were then started successfully. Media and Game service-account traversal passed. Existing Radarr, Sonarr, and Gamarr containers were started in place and verified without recreation.

To make transient boot-time NFS readiness recoverable, JAR-23 added `/usr/local/sbin/rotom-nas-docker-recovery` and enabled `/etc/systemd/system/rotom-nas-docker-recovery.service`. The helper actively starts/verifies the real Downloads/Media/Game NFS mount units, checks the Downloads torrent tree and sentinel, restores both bindfs views, verifies Media/Game traversal, and starts only stopped NAS-backed containers whose Docker error indicates a NAS/mount startup failure. It currently covers qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin. A manual healthy-state execution completed successfully without restarting already-running containers. Full reboot validation remains deferred.

### JAR-24 final-backup storage verification — 2026-09-25

The JAR-24 preflight verified `/mnt/nas-rotom-backup` as the existing NFSv3 backup mount and confirmed the Restic repository directory `/mnt/nas-rotom-backup/rotom-restic-backup` was reachable before any backup work. No NAS export, fstab entry, mount option, share permission, service-storage path, library path, or torrent path was changed.

After confirming that no Restic process and no backup service were active, Restic's supported `unlock` removed one stale repository lock left by an aborted helper. No repository file was manually deleted, moved, or reorganized. The canonical backup then saved final snapshot `fbe1838e` at `2026-09-25 18:14:22 PDT`, completed the existing retention/prune policy, and later passed both structural and full-pack readability checks.

This does not change Rotom's protection boundary: `/mnt` is excluded from host Restic. NAS-resident Media/Game libraries, Downloads torrents, Shared Drive data, per-service NAS shares, and the Restic repository itself remain separate storage/protection domains.


### JAR-25 Shared Drive bare-metal image — 2026-09-25

JAR-25 selected the existing Shared Drive as the physically separate destination for the pre-Proxmox whole-disk rollback image. Preflight resolved `/mnt/nas-shared-drive` to NFSv3 source `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Shared_Drive/.data` and reported `5075299729408` bytes available, comfortably above the source NVMe's exact `250059350016` bytes. No fstab entry, export, share root, or mount option was changed.

The final image is `/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25/rotom-nvme-live-pre-proxmox-2026-09-25.img`. Logical size is exactly `250059350016` bytes; recorded allocated size is `250059358208` bytes. Acquisition-stream and full stored-image reread SHA-256 both equal `b62b2c8210bb2a6446abc29c5767c7d8531f547dfd2ec3f0259dbee8d74c5e90`. `fdisk` inspection of the stored file exposed the EFI System Partition and Linux filesystem partition. Restore instructions and supporting metadata/checksum files are stored in the same JAR-25 directory.

Post-closeout access inspection showed the JAR-25 directory as numeric `977:988` mode `0700` with ACL `user::rwx,group::---,other::---`. The raw image presents as numeric `977:988` mode `0660` with `user::rw-,group::rw-,other::---`. Functional dry-run testing using client identity `988:988` could create/read/delete within the protected directory. The observed numeric owner mapping is a UNAS/NFS presentation detail; do not infer ownership by the local Linux service name associated with those numbers. Because the parent is `0700`, group/other cannot traverse the pathname to the image. Preserve that parent restriction because the raw disk image contains all host-local state, including secrets.

This image is on a different physical device from the Rotom NVMe but on the same UNAS appliance that hosts the Restic repository and other NAS data. JAR-25 therefore adds an independent whole-disk recovery artifact relative to the source NVMe, not an independent NAS-hardware failure domain.

## 5. Preserved Pre-Migration Persistent Mount Configuration

All twelve NAS paths use Rotom's established fstab automount pattern:

```text
defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s
```

The eight JAR-6 entries are:

```text
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Infra/.data /mnt/nas-infra nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Smarthome/.data /mnt/nas-smarthome nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Documents/.data /mnt/nas-documents nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Downloads/.data /mnt/nas-downloads nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Web/.data /mnt/nas-web nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Filesync/.data /mnt/nas-filesync nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Apps/.data /mnt/nas-apps nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Auth/.data /mnt/nas-auth nfs defaults,_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s 0 0
```

Existing Media, Game, Backup, and Shared Drive fstab entries were preserved. The successful JAR-6 commissioning backup of the pre-edit file is `/root/fstab.jar6-retry-20260921-163659.bak`. `findmnt --verify --tab-file /etc/fstab` reported zero parse errors/errors after commissioning and again after reboot (with the pre-existing warning unrelated to JAR-6).

## 6. Preserved Pre-Migration systemd Mount and Automount Behavior

Systemd generates paired `.automount` and `.mount` units from the fstab entries. JAR-6 verified all eight new `.automount` units active, triggered each share independently as its intended service account, and verified the corresponding `.mount` unit plus exact NFS source. A normal Rotom reboot then reconfirmed all eight `.automount` units returned and every exact source mounted on demand.

Backup and Shared Drive are also automount-backed. Immediately after reboot they must be traversed or otherwise triggered before expecting an underlying NFS result from an NFS-filtered `findmnt` query; JAR-6 continuation testing triggered both and verified their source and root metadata unchanged.

The `nofail`/automount design intentionally avoids making general NAS availability a hard host-boot requirement. Docker still has no global dependency on these NAS mounts. `unas-backup.service` remains the service-specific exception and retains `RequiresMountsFor=/mnt/nas-rotom-backup`.

## 7. Preserved Pre-Migration Operational and NAS-Unavailable Behavior

`docker.service` has no global `RequiresMountsFor=` dependency on any NAS path. It depends on `network-online.target`, but Docker as a whole is intentionally not coupled to NAS availability.

The verified configuration establishes that:

- Media, Backup, Game, and the separately recorded Shared Drive path have persistent systemd automount entries generated from `/etc/fstab`;
- the documented Media and Backup automounts are attached to `remote-fs.target`;
- `/mnt/nas-media` access is configured to trigger the NFS mount on demand;
- `/mnt/nas-game` and `/mnt/nas-rotom-backup` access are configured the same on-demand way;
- `unas-backup.service` explicitly requires `/mnt/nas-rotom-backup`;
- an unavailable NAS is not intended to prevent Rotom from completing boot solely because of these `fstab` entries.

A deliberate NAS-disconnected **host boot** test and a live runtime-loss test while qBittorrent is already running have **not** been performed. JAR-21 directly verified qBittorrent recreation fails closed when `/mnt/nas-downloads/.rotom-qbt-nas-ready` is unavailable and succeeds again after restoration. The active Downloads torrent and sentinel binds use `create_host_path: false`, preventing local-directory fallback on recreation.

No global `RequiresMountsFor=/mnt/nas-downloads` dependency is configured on `docker.service`. This keeps unrelated Docker workloads independent of NAS availability while qBittorrent enforces its own service-specific startup guard.

## 8. Preserved NAS Directory Structure / Restoration Target

The audit intentionally limited traversal depth.

### `/mnt/nas-media`

Current JAR-21 storage role:

```text
/mnt/nas-media
└── library
    ├── movies
    └── shows
```

The former `/mnt/nas-media/torrents` rollback tree was removed after checksum verification against active Downloads storage.

### `/mnt/nas-game`

Current JAR-21 storage role:

```text
/mnt/nas-game
└── library
    └── games
        ├── pc
        └── roms
```

The former `/mnt/nas-game/torrents` rollback tree was removed after it was verified to contain no files. Palworld saves remain local under `/home/game/docker`.

### `/mnt/nas-downloads`

Current active torrent layout:

```text
/mnt/nas-downloads
├── .rotom-qbt-nas-ready
└── torrents
    ├── games
    ├── incomplete
    │   ├── games
    │   ├── movies
    │   └── shows
    ├── movies
    └── shows
```


### `/mnt/nas-rotom-backup`

Immediate children observed:

```text
/mnt/nas-rotom-backup
├── rotom-linux-home-server-backup-restore-kit
├── rotom-restic-backup
└── .DS_Store
```

The Restic repository was not recursively inspected. Its contents remain critical opaque repository data and should not be manually reorganized.


### `/mnt/nas-shared-drive`

JAR-25 added the protected bare-metal recovery subtree:

```text
/mnt/nas-shared-drive
└── rotom-bare-metal
    └── jar-25-2026-09-25
        ├── CONSISTENCY-NOTE.txt
        ├── JAR-25-RESULT.txt
        ├── METADATA-SHA256SUMS
        ├── RESTORE-PROCEDURE.txt
        ├── image-fdisk.txt
        ├── rotom-nvme-live-pre-proxmox-2026-09-25.image.sha256
        ├── rotom-nvme-live-pre-proxmox-2026-09-25.img
        ├── rotom-nvme-live-pre-proxmox-2026-09-25.source.sha256
        ├── source-fdisk.txt
        ├── source-lsblk.txt
        ├── source-partition-table.sfdisk
        ├── source-root-ext4.txt
        ├── runtime-containers-before.txt
        ├── runtime-containers-after.txt
        ├── runtime-paused-units.txt
        ├── runtime-status-before.txt
        └── runtime-systemd-after.txt
```

The listing above records the core non-secret JAR-25 evidence files produced by the completed workflow. An optional UEFI-NVRAM capture and image partition-table dump may also be present when the corresponding inspection commands succeeded; do not require those optional filenames when validating the primary image/checksum contract. No `.partial` image remained after acceptance.

## 9. Preserved Pre-Migration NFS Identity, Ownership, Permissions, and ACLs

### Current media identity and ownership

The Media, Game, Backup, and Shared Drive mounts are now all freshly verified as NFSv3 with `sec=sys`. Numeric Unix UID/GID values govern ownership; Rotom translates them into local names for display. The media service now retains UID **127** and uses primary GID **5000**, replacing its former GID 129. The later conversation records the Media NAS group migration to 5000 while preserving existing file-owner UIDs.

Recorded post-change examples:

| Path | Mode | UID | GID | Evidence / interpretation |
| --- | --- | ---: | ---: | --- |
| `/mnt/nas-media` | `2770` | 988 | 5000 | Media share root; setgid, group access, no access for others |
| `/mnt/nas-media/library/movies` | `2775` | 977 | 5000 | Existing NAS owner preserved; media group can write |
| `/mnt/nas-media/library/shows` | `2775` | 977 | 5000 | Existing NAS owner preserved; media group can write |
| Historical `/mnt/nas-media/torrents/...` objects | varied | varied | 5000 | Pre-JAR-21 evidence only; the old Media torrent tree is now removed |

The historical scans of the Media `library/` and former `torrents/` tree found the recorded GID/mode consistency at that time. JAR-21 later removed the Media torrent tree. Setgid/group-write semantics remain relevant to the Media library; active torrent ownership now belongs to the Downloads share.

### Game identity and access — 2026-09-19

The `game` service retains UID `995` and now uses primary GID `5001`. The Game share root was changed non-recursively from `988:988` mode `0770` to `988:5001` mode `2770`. Verification showed `game` read/write/traverse access and denied all three operations to the then-named `rotom` (`997:986`) and `smart-home` (`126:128`) identities plus the other tested accounts. Those numeric-ID results carry through the later renames to `infra` and `smarthome`; the six new service identities were not part of this matrix. A disposable service-account write test created `/mnt/nas-game/.rotom-game-write-test` as `995:5001` mode `0664` and removed it successfully. Historical qBittorrent testing through the former `/mnt/nas-game/torrents` tree created `997:5001`; JAR-21 later removed that file-empty rollback tree. Game library directories remain `995:5001` mode `2775`. This is ordinary-account isolation only; sudo/root and Docker administrators can still cross the boundary.

### Isolated NFS access and server-side group lookup

The conversation identified UNAS `rpc.mountd --manage-gids` behavior, including the process and `RPCMOUNTDOPTS=--manage-gids` setting. This replaces the request's **supplementary** group list with groups looked up for that UID on the NAS; it does not replace the request's primary GID. Therefore adding a user to Rotom's local supplementary `media` group is not a reliable way to grant access on this setup. These semantics are also described by the upstream [nfs-utils mountd documentation](https://github.com/linux-nfs/nfs-utils/blob/master/utils/mountd/mountd.man).

The recorded test used Jared's UID 1000:

- On UNAS, UID 1000 resolves to `jwines760`, with primary GID 988 and supplementary GID 987.
- A normal Rotom request with primary GID 1000 and local supplementary GID 5000 was denied access to the migrated Media share.
- A temporary request using primary GID 5000 succeeded.
- Media processes using UID 127 and primary GID 5000 can access the Media share through its group permissions.

This explains the observed behavior with UNAS Isolated Mode. It is not evidence that every NFS identity is mapped to one fixed UID/GID. The 2026-09-18 protection work additionally inspected the actual UNAS export options: `Media` and `Rotom_Home_Server_Backup` use `no_root_squash`, while `Shared_Drive` was observed with `root_squash,all_squash,anonuid=977,anongid=988`. The backup export also uses `no_all_squash`. Thus Rotom root is preserved as UID 0 on the backup export. Numeric GID 5000 Media access and numeric GID 5001 Game access were demonstrated; this document does not claim that corresponding named groups were created in the UNAS account database.

The adopted operating policy is that `jared` stays outside the media group and administers media storage by switching identity with `sudo -iu media`. A final 2026-09-19 `id jared` / `getent group media` check verified that Jared is not a supplementary member of media GID `5000`. The denial with Jared's normal primary GID was observed separately and is consistent with that intended ordinary-account isolation.

This separates routine account access. It does not prevent a root, sudo, or Docker-daemon administrator from assuming another identity or accessing service data. The baseline Docker-group membership remains relevant to that limit.

### JAR-6 service-share identity and access — 2026-09-21

JAR-6 preserves NAS owner UID `988` and assigns each new service share the matching service primary GID: Infra `5002`, Smarthome `5003`, Documents `5004`, Downloads `5005`, Web `5006`, Filesync `5007`, Apps `5008`, Auth `5009`. Every new `.data` root is mode `2770`. The change was non-recursive: only each share root was modified, and no existing descendant ownership was normalized.

Functional acceptance testing used the actual Rotom service identities because UNAS `rpc.mountd --manage-gids` makes numeric request identity behavior material. On every new share the intended account could traverse/list/create, new files inherited the expected service GID, child directories inherited setgid, and disposable objects were removed. A different unrelated service account was denied traverse/read/write. A post-reboot repetition confirmed the positive/negative boundary persisted.

The eight new shares have no application data and no Docker binds. The private `2770` roots intentionally deny ordinary Jared traversal; metadata inspection should use the intended service account or privileged administrative access rather than treating Jared's denial as a fault.

### Backup access protection — 2026-09-18

The backup share keeps numeric ownership `988:988`, but its **share-root mode is now `0700`**, changed non-recursively from `0770` to remove ordinary group access while preserving the root-run backup path.

| Path | Mode | UID | GID | Evidence |
| --- | --- | ---: | ---: | --- |
| `/mnt/nas-rotom-backup` | `0700` | 988 | 988 | Verified on both Rotom and UNAS after the change |
| `/mnt/nas-rotom-backup/rotom-linux-home-server-backup-restore-kit` | `0770` | 1000 | 1000 | Original audit; not changed by the non-recursive share-root chmod |
| `/mnt/nas-rotom-backup/rotom-restic-backup` | `0770` | 977 | 988 | Original audit; root read/write/traverse verified after the share-root change |

Before the change, UID `1000` mapped on UNAS to `jwines760`, primary GID `988`, which explained Jared's ordinary access through the root's group bits. After `chmod 0700` on the share root, the original matrix verified all then-current ordinary accounts denied. A fresh 2026-09-19 matrix after the Game GID migration additionally verified current `game` (`995:5001`) and all six other ordinary named accounts denied read, write, and traverse access. Root retained read/write/traverse access to the share root and repository because the backup export uses `no_root_squash,no_all_squash`.

The hosting Btrfs filesystem was verified mounted with `noacl`; `getfacl` and `setfacl` were unavailable. Therefore a named-user ACL deny was not an available implementation path. Changing Jared's UNAS primary GID was rejected as broader than this share-specific protection change.

UNAS UID/GID `988` is the non-login `unifi-drive` service identity and had active `rclone`/`unifi-drive` processes. Rotom's numeric UID/GID `988` resolves locally to the non-login `fwupd-refresh` account; no UID/GID 988 process was observed on Rotom during the check. The numeric owner match means a Rotom process deliberately running as UID 988 would still be owner-equivalent over this NFS export; the change is an ordinary-account accident-prevention boundary, not a hard security boundary against privileged administrators.

Do not recursively change Restic ownership or permissions. Rollback for the share-root protection is the single non-recursive restoration of mode `0770` on the UNAS share root, but no rollback was needed during this deployment.

### Historical baseline and numeric-name interpretation

Before the Media migration, the original audit recorded the Media root as `988:988` mode `0770`, `library/` and `torrents/` as `977:988` mode `0775`, and `torrents/incomplete` as `1004:988` mode `0770`. These values are historical and must not be used as current restore targets for Media. The sampled baseline ACLs contained only ordinary owner/group/other entries; no named-user or named-group ACL entries were observed on those paths.

UID/GID 988 displaying as `fwupd-refresh` on Rotom is a local numeric-name collision, not proof that Rotom's firmware-update service owns NAS data in a meaningful cross-machine sense. After the migration, the Media root may display as `fwupd-refresh:media` because its numbers are `988:5000`. UNAS UID/GID 988 is now verified as the `unifi-drive` service identity; exact NAS identities behind owner UIDs 977 and 1004 remain incompletely documented. Preserve those owners; do not replace them solely to make `ls` names look familiar.

Use numeric inspection on **Rotom** (as Jared; use the service identity for paths beneath Media):

```bash
ls -ldn /mnt/nas-media /mnt/nas-game /mnt/nas-rotom-backup
stat -c '%A %a %u:%g %n' /mnt/nas-media /mnt/nas-game /mnt/nas-rotom-backup
sudo -iu media
id
stat -c '%A %a %u:%g %n' /mnt/nas-media/library /mnt/nas-downloads/torrents
exit
```

These commands inspect metadata; they do not alter permissions or repository contents.

## 10. Preserved Pre-Migration Docker-to-NAS Storage Relationships

### `/mnt/nas-media`

| Container | Host source | Container path | Access |
| --- | --- | --- | --- |
| Jellyfin | `/mnt/nas-media/library` | `/data/library` | read-only |
| Sonarr | `/mnt/nas-media` | `/media` | read-write |
| Radarr | `/mnt/nas-media` | `/media` | read-write |

### `/mnt/nas-game`

| Container | Host source | Container path | Access |
| --- | --- | --- | --- |
| Gamarr | `/mnt/nas-game` | `/game` | read-write |

Palworld remains on local storage under `/home/game/docker`.

### `/mnt/nas-downloads` and read-only views

| Container | Host source | Container path | Access |
| --- | --- | --- | --- |
| qBittorrentVPN | `/mnt/nas-downloads/torrents` | `/media/torrents` | read-write |
| qBittorrentVPN | `/mnt/nas-downloads/torrents` | `/game/torrents` | read-write |
| qBittorrentVPN sentinel | `/mnt/nas-downloads/.rotom-qbt-nas-ready` | `/run/rotom-nas-downloads` | read-only |
| Sonarr | `/mnt/nas-downloads-media-ro/torrents` | `/media/torrents` | read-only |
| Radarr | `/mnt/nas-downloads-media-ro/torrents` | `/media/torrents` | read-only |
| Gamarr | `/mnt/nas-downloads-game-ro/torrents` | `/game/torrents` | read-only |

The two `*-ro` sources are bindfs views of `/mnt/nas-downloads`, not separate NAS exports.

### `/mnt/nas-rotom-backup`

No Docker container is intended to bind-mount `/mnt/nas-rotom-backup`. The former Glances read-only bind remains removed.

## 11. Preserved Pre-Migration Service Storage Contracts

### Jellyfin

```text
/home/media/docker/jellyfin/config -> /config       RW
/mnt/nas-media/library             -> /data/library RO
```

### Sonarr / Radarr

```text
/home/media/docker/<service>/config       -> /config         RW
/mnt/nas-media                            -> /media          RW
/mnt/nas-downloads-media-ro/torrents      -> /media/torrents RO
```

### qBittorrentVPN

```text
/home/downloads/docker/qbittorrentvpn/config -> /config                  RW
/mnt/nas-downloads/torrents                  -> /media/torrents          RW
/mnt/nas-downloads/torrents                  -> /game/torrents           RW
/mnt/nas-downloads/.rotom-qbt-nas-ready      -> /run/rotom-nas-downloads RO
/etc/localtime                                -> /etc/localtime            RO
```

### Gamarr

```text
/home/game/docker/gamarr/config          -> /config         RW
/mnt/nas-game                            -> /game           RW
/mnt/nas-downloads-game-ro/torrents      -> /game/torrents  RO
```

### Prowlarr

```text
/home/downloads/docker/prowlarr/config -> /config RW
```

Prowlarr has no direct NAS bind.

### Service IDs

Jellyfin, Radarr, and Sonarr remain `127:5000`; Gamarr remains `995:5001`. JAR-21 moved both Prowlarr and qBittorrentVPN to Downloads identity `901:5005`. The active Downloads share uses GID `5005`; the read-only bindfs views deliberately present its files as Media or Game identities to the corresponding Arr applications.

## 12. Preserved Pre-Migration Media Application Root Folders

A read-only SQLite query on 2026-09-19 verified the application-configured root folders:

- Sonarr: `/media/library/shows/`
- Radarr: `/media/library/movies/`

These match the verified Docker storage contract in which both applications see the complete Media NAS at `/media`.

## 13. Preserved Pre-Migration Hardlink Behavior and Compatibility

JAR-21 intentionally ended the prior same-filesystem torrent/library layout. Active torrent data now resides on `/mnt/nas-downloads`, while final movies/shows remain on `/mnt/nas-media` and final games remain on `/mnt/nas-game`. These are separate NFS filesystems, so a hardlink cannot span the source and destination filesystems.

Radarr, Sonarr, and Gamarr import through read-only Downloads views and therefore **copy** completed data into their final libraries. qBittorrent retains the source under `/mnt/nas-downloads/torrents` for seeding. The user confirmed a real Arr download/import works end to end after the cutover. Preserve this copy-import expectation when troubleshooting disk usage or import behavior; do not attempt to recover the old hardlink model by broadening write access to the read-only views.

## 14. Preserved Pre-Migration Service-to-Storage Dependency Map

### Directly dependent on `/mnt/nas-media`

- Jellyfin
- Sonarr
- Radarr

### Directly dependent on `/mnt/nas-game`

- Gamarr

Palworld is not dependent on `/mnt/nas-game`; its saves remain local.

### Directly dependent on `/mnt/nas-downloads`

- qBittorrentVPN, read/write through `/mnt/nas-downloads/torrents` and the Downloads startup sentinel
- Radarr/Sonarr indirectly through `rotom-downloads-media-ro.service` and `/mnt/nas-downloads-media-ro`
- Gamarr indirectly through `rotom-downloads-game-ro.service` and `/mnt/nas-downloads-game-ro`

Prowlarr participates in the download workflow but has no NAS filesystem bind.

### Current backup dependency

- `rotom-restic-backup.service`, via `RequiresMountsFor=/mnt/nas-rotom-restic-backup`
- `/usr/local/sbin/rotom-restic-backup` checks `/mnt/nas-rotom-restic-backup` before opening `/mnt/nas-rotom-restic-backup/rotom-restic-backup`

The former `/mnt/nas-rotom-backup` dependency is retired and remains only in dated historical sections.

Docker itself has no global NAS dependency.

## 15. Reconstruction-Critical Information

A Rotom rebuild must preserve the following storage contract.

### NAS server

```text
192.168.1.70
```

### Media mount — current VM-era contract

```text
Source:     192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Media/.data
Target:     /mnt/nas-media
Filesystem: nfs
fstab:      defaults,_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s
```

The historical bare-metal `/var/nfs/shared/Media` source remains preserved in dated pre-migration sections only.

### Game mount — current VM-era contract

```text
Source:     192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Game/.data
Target:     /mnt/nas-game
Filesystem: nfs
fstab:      defaults,_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s
```

### Backup mount — current VM-era contract

```text
Source:     192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Rotom_Restic_Backup/.data
Target:     /mnt/nas-rotom-restic-backup
Filesystem: nfs
fstab:      defaults,_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s
Repository: /mnt/nas-rotom-restic-backup/rotom-restic-backup
```

### Docker path contract — current VM-era sources

```text
Jellyfin:       /mnt/nas-media/library                  -> /data/library           RO
Sonarr:         /mnt/nas-media                          -> /media                  RW
                /mnt/nas-downloads-media-ro/torrents    -> /media/torrents         RO
Radarr:         /mnt/nas-media                          -> /media                  RW
                /mnt/nas-downloads-media-ro/torrents    -> /media/torrents         RO
qBittorrentVPN: /mnt/nas-downloaders/torrents           -> /media/torrents         RW
                /mnt/nas-downloaders/torrents           -> /game/torrents          RW
                /mnt/nas-downloaders/.rotom-qbt-nas-ready -> /run/rotom-nas-downloads RO
Gamarr:         /mnt/nas-game                           -> /game                   RW
                /mnt/nas-downloads-game-ro/torrents     -> /game/torrents          RO
Prowlarr:       /home/downloaders/docker/prowlarr/config -> /config RW; no NAS bind
Glances:        no `/mnt/nas-rotom-restic-backup` bind
```

The bindfs service names and qBittorrent container-side sentinel path intentionally retain the historical `downloads` spelling for compatibility; their host source is current `/mnt/nas-downloaders`.

### Numeric identity contract

```text
Rotom media account: UID 127, primary GID 5000
Media-owned Compose services (Jellyfin/Radarr/Sonarr): PUID=127, PGID=5000, UMASK=002
Downloaders account: UID 901, primary GID 5005
Central qBittorrentVPN: owner downloaders; PUID=901, PGID=5005, UMASK=002
Prowlarr active owner: downloaders UID 901, primary GID 5005; PUID=901, PGID=5005, UMASK=002
Gamarr: owner game; PUID=995, PGID=5001
Media NAS root: UID 988, GID 5000, mode 2770
Downloaders NAS root: UID 988, GID 5005, mode 2770
Media data: preserve existing owner UIDs; GID 5000, group write, directory setgid
Game account: UID 995, primary GID 5001
Game NAS root: UID 988, GID 5001, mode 2770; game allowed, tested unrelated named accounts denied
Downloaders torrent subtree: owned/written under Downloaders identity/GID 5005
Game library directories: UID 995, GID 5001, mode 2775
Current service accounts/shares: infra 997:5002, smarthome 126:5003, documents 900:5004, downloaders 901:5005, web 902:5006, filesync 903:5007, apps 904:5008, auth 905:5009; each service share root UID 988, matching GID, mode 2770
Backup root: UID 988, GID 988, mode 0700; ordinary named accounts denied, root-run backup retained
```

Rebuild Media with GID 5000, not the historical GID 129. Preserve the existing NAS owners, bind-mount layout, and service-directory ownership. Restore configuration and data without applying blanket ownership changes to NAS or Restic contents.

### Current library and download layout

```text
/mnt/nas-media/library/movies
/mnt/nas-media/library/shows
/mnt/nas-game/library/games/pc
/mnt/nas-game/library/games/roms
/mnt/nas-downloaders/.rotom-qbt-nas-ready
/mnt/nas-downloaders/torrents/games
/mnt/nas-downloaders/torrents/incomplete/games
/mnt/nas-downloaders/torrents/incomplete/movies
/mnt/nas-downloaders/torrents/incomplete/shows
/mnt/nas-downloaders/torrents/movies
/mnt/nas-downloaders/torrents/shows
```

The Downloaders sentinel is part of qBittorrent's startup fail-closed protection and must not be removed casually.

## 16. Preserved Pre-Migration Service-Account NAS Pattern

The service-storage identity pattern is now implemented for all ten service domains. UIDs remain stable while service primary GIDs align with NAS group ownership.

| Rotom account | UID | Primary GID | Pre-migration NAS mount | Historical pre-migration use |
|---|---:|---:|---|---|
| media | 127 | 5000 | `/mnt/nas-media` | Active final movie/show library |
| game | 995 | 5001 | `/mnt/nas-game` | Active final Game library |
| infra | 997 | 5002 | `/mnt/nas-infra` | Commissioned boundary; no workload data |
| smarthome | 126 | 5003 | `/mnt/nas-smarthome` | Commissioned boundary; HA/Homebridge local |
| documents | 900 | 5004 | `/mnt/nas-documents` | Commissioned boundary; empty |
| downloads | 901 | 5005 | `/mnt/nas-downloads` | Active centralized torrent storage and download-stack identity |
| web | 902 | 5006 | `/mnt/nas-web` | Commissioned boundary; websites local |
| filesync | 903 | 5007 | `/mnt/nas-filesync` | Commissioned boundary; empty |
| apps | 904 | 5008 | `/mnt/nas-apps` | Commissioned boundary; empty |
| auth | 905 | 5009 | `/mnt/nas-auth` | Commissioned boundary; empty |

Jared and Fran are not assigned these service storage groups as an ordinary-access mechanism. Use `sudo -iu <service-user>` for service-specific administration. This is not a security boundary against sudo/root or Docker-daemon administrators.

JAR-6 changed only storage identity/boundary state for the new shares. It did not move Home Assistant/Homebridge, Palworld, websites, Docker databases, or other local application state. Future workload placement onto any still-boundary-only commissioned share requires a separate application-storage review. At the final pre-migration baseline, Downloads was no longer boundary-only; its qBittorrent missing-mount protection used the sentinel and `create_host_path: false`. Current state uses the `downloaders` identity and `/mnt/nas-downloaders`.

Backup and Shared Drive remain separate from the service-GID sequence. Backup is `988:988` mode `0700`; Shared Drive is `988:988` mode `0770`.

## 16A. Live Post-Boot Storage Verification — 2026-09-27

A read-only audit after a fresh Rotom VM reboot verified all twelve current fstab entries use the expected UNAS `.data` sources and `_netdev,nofail,x-systemd.automount,x-systemd.mount-timeout=30s` pattern. All twelve generated `.automount` units were loaded and active. The active NFS tree showed current `Downloader/.data`, Media, Game, service shares, Shared Drive, and `Rotom_Restic_Backup/.data`.

The two compatibility views remained mounted read-only from `/mnt/nas-downloaders`; traversal tests passed as `media` and `game`. Mounted Restic storage presented numeric `988:988` mode `0700`. The earlier raw `/mnt` listing showed the automount-facing path as `root:root`/`0755` before clean mounted-root interpretation; because this audit did not isolate the underlying local directory while fully unmounted, only the mounted NFS metadata is current authority. No permission repair is indicated.

Service-home symlinks were also freshly verified for all ten service identities, including `/home/downloaders/nas-downloaders -> /mnt/nas-downloaders`.

## 17. Outstanding / Needs Verification

Current JAR-31 production storage is restored and the reboot recovery path is verified. JAR-32/JAR-33 also verified the VM-era Restic repository and baseline; automatic Restic execution remains disabled as an operational commissioning choice, not missing storage evidence. `/mnt` remains outside the guest Restic source scope.

- **Needs Verification — UNAS metadata persistence:** recheck service-share root metadata after a future UNAS/UniFi Drive restart/update, especially Downloader `988:5005`, Media `988:5000`, Game `988:5001`, current Restic share `Rotom_Restic_Backup` at `988:988` mode `0700`, and service GIDs `5002`–`5009`. Resolve read-only with `findmnt` and `stat` from Rotom plus UNAS-side export/share inspection after the event.
- **Needs Verification — live Downloader-NFS loss:** boot/recreation recovery is verified, but sudden NFS loss while qBittorrent is already running is not. Read-only inspection can verify the absence/presence of a watchdog in the current Compose/systemd definitions; actual runtime behavior requires a separately planned non-destructive maintenance test.

**Retired/historical:** do not recreate the old `Downloads/.data` export merely because it appears below. Current fstab/mount/showmount evidence uses `Downloader/.data`; `Downloads/.data` is retained only as dated recovery context.

## 18. Pre-Migration Architecture Summary

Rotom keeps the operating system and Docker configuration on local NVMe while using the UNAS for bulk storage. Media and Game are final-library shares. JAR-21 activates the JAR-6 Downloads share as the centralized torrent domain, with qBittorrent-specific fail-closed startup protection and read-only bindfs views for the Arr applications. The other seven JAR-6 service shares remain boundary-only.

At the final pre-migration baseline, all twelve NAS mountpoints used fstab-backed systemd automounts. The eight JAR-6 shares use exact discovered `.data` exports, NFSv3 `sec=sys`, roots `988:5002` through `988:5009`, and mode `2770`. Their ordinary-account isolation and normal Rotom reboot persistence are verified. Backup and Shared Drive remain separate existing mounts. Docker has no global NAS dependency; `unas-backup.service` retains its specific backup-mount requirement. Host Restic excludes `/mnt`, so NAS share content requires a separate protection policy unless safely reproducible.

## 19. Historical Consolidated Verification — 2026-09-19 (pre-JAR-21)

- Current automounts for Media, Game, Backup, and Shared Drive are loaded/active/running from `/etc/fstab`, with live **NFSv3** filesystems beneath them. The fresh audit captured `vers=3` for all four mounts.
- qBittorrent sentinel directories are confirmed present: Media `997:5000` mode `2555`; Game `997:5001` mode `2555`. The earlier unprivileged `test -e` false-negative was caused by parent-directory traversal restrictions, not missing sentinels.
- Jared's final account state excludes media GID `5000`.
- The latest read-only access testing (2026-09-19) confirms the intended boundaries for the identities tested then: `media` had read/write/traverse on `/mnt/nas-media` only, `game` on `/mnt/nas-game` only, and the other tested accounts were denied on Media, Game, or Backup as recorded. The later `infra`/`smarthome` name changes preserved numeric IDs, so those two results remain applicable; the six new 2026-09-21 service identities were not tested for ordinary NAS access. Root retains the privileged backup path.
- Filesystem device IDs remain matched within each hardlink workflow: Media `library` and `torrents` both report device `91`; Game `library` and `torrents` both report device `104`.

## 19A. Historical JAR-6 Consolidated Storage Verification — 2026-09-21 (pre-JAR-21)

- Exact exports for all eight new shares were discovered with `showmount`, then persisted in fstab.
- Every new share root is `988:<service GID>` mode `2770`; only the root was changed non-recursively.
- Intended-account and unrelated-account functional tests passed on all eight shares; disposable objects were removed.
- A normal Rotom reboot restored all eight `.automount` units, exact NFSv3 `sec=sys` sources, GIDs/modes, and positive/negative access behavior.
- Media and Game sources/metadata remained unchanged. Backup and Shared Drive returned unchanged after explicit post-boot trigger.
- qBittorrent's Media/Game torrent and sentinel binds remained unchanged; no new service share is used by Docker.
- `/mnt/nas-game-servers` remains absent/unmounted; the former UNAS `Game_Servers/.data` path was observed absent during JAR-6 audit.
- Host Restic still excludes `/mnt`; no JAR-6 NAS path entered the include scope.

## 19B. Historical JAR-21 Consolidated Storage Verification — 2026-09-21

- `/mnt/nas-downloads/torrents` is the active torrent tree and was 9.8G at final closeout.
- Old `/mnt/nas-media/torrents` and `/mnt/nas-game/torrents` trees are absent.
- `rotom-downloads-media-ro.service` and `rotom-downloads-game-ro.service` are both enabled and active; `findmnt` shows the corresponding bindfs views read-only.
- qBittorrent active mounts use Downloads for both `/media/torrents` and `/game/torrents` plus the read-only Downloads sentinel; no old Media/Game qBittorrent NAS mounts remain.
- A real Arr download/import was accepted by the user after cutover; cross-filesystem import is intentionally copy-based.
- Final application checks passed and system state was `running` with no failed systemd units.

## 20. Historical Storage Verification — 2026-09-15

A third active NFSv3 mount is present: the 192.168.1.70 Shared_Drive export mounted at /mnt/nas-shared-drive. It has the same fstab automount pattern, and at the 2026-09-15 audit it reported the same observed capacity figures as the documented media and backup exports. Its observed top-level entries are Important and rotom. This updates the original two-active-mount scope; the NAS-side purpose, ACLs, and identity mapping of this shared-drive export remain unverified.

This paragraph records the September 15 observation. A later 2026-09-19 audit now verifies the Shared Drive is currently mounted as NFSv3 through its fstab-backed systemd automount. Its effective NAS permission model and intended service role remain separate questions.

## 21. Related Documentation

- See `07-Users-and-Permissions.md` for canonical Linux UID/GID, group, ACL, and account-access definitions.
- See `02-Docker-Services.md` for canonical Compose/bind-mount deployment details from the Docker perspective.
- See `05-Backup-and-Restore.md` for backup scope and the distinction between host backup and NAS-data protection.
- See `08-Rotom-Directory-Tree.txt` for observed filesystem topology.
