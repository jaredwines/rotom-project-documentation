# 01 - Rotom Server Inventory

**Documentation set:** Rotom Project Documentation  
**Document role:** Architecture/index starting point and high-level Rotom summary  
**Hosts:** Proxmox hypervisor `proxmox` and portable Debian VM `rotom`  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-09-27 — live post-boot verification reconciled; current Phase B state refreshed
**Related canonical sources:** `02-Docker-Services.md`, `03-Network-and-Domains.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`, `08-Rotom-Directory-Tree.txt`  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

## 1. Purpose and Scope

This document is the front-page architecture/index for Rotom. It summarizes the current system and points to the canonical subject document for detailed facts; it is not the canonical home for full account, container, network, mount, backup, automation, or filesystem registers.

Purpose: front-page index and quick-reference architecture summary.

**RPD state labels:** **Current** means the latest supported recorded state; **Historical** preserves dated evidence that may no longer apply; **Retired** means intentionally no longer active; **Proposed** is not implemented; **Needs Verification** means the RPD lacks enough current evidence and should not be treated as settled.

## 2. Current Rotom Summary

The live architecture changed at the Phase B cutover. The old Linux Mint bare-metal installation remains preserved through JAR-24/JAR-25 recovery artifacts and the historical sections of this documentation. The physical NUC runs Proxmox only. The Debian `rotom` guest now has its network/storage identities, Docker/Compose foundation, production application layer, NAS recovery helper, and selected backup-administration controls restored through **JAR-31**; JAR-32 verified the dedicated Restic cutover and first VM-era write, JAR-66 established dedicated Proxmox VM-backup storage, and JAR-33 accepted the combined recovery checkpoint as **Rotom Virtualization Baseline**. The final JAR-31 reboot acceptance restored the exact 16-container running set with Jared Wines intentionally absent and zero failed systemd units.

| Item | Verified value |
|---|---|
| Physical host | `proxmox`; FQDN `proxmox.rotom.casa`; Proxmox VE `9.2.20`; kernel `7.0.14-19-pve` |
| Physical hardware | Wortmann_AG `1009664;1400107` / Intel `NUC8BEB`; Intel Core i5-8259U (4C/8T); 15 GiB RAM; Samsung 970 EVO Plus 250GB NVMe |
| Proxmox management | `vmbr0` at `192.168.1.68/24`; gateway `192.168.1.1`; DNS `192.168.1.1`; physical NIC MAC `1c:69:7a:0e:f2:f7`; host does not own `.69` |
| Proxmox administration | `root`; `/usr/bin/zsh` + Oh My Zsh; Mac key-only SSH verified via aliases `proxmox` / `proxmox.rotom.casa` to `192.168.1.68` using dedicated `~/.ssh/id_ed25519_proxmox`; prior RSA authorized key preserved; password-auth policy unchanged |
| Proxmox storage/workload boundary | ext4/LVM-thin with `local` + `local-lvm`; hypervisor has no Rotom application Docker/Podman runtime or application NFS mounts. Dedicated backup storage `nas-rotom-proxmox-backup` is the intentional hypervisor NFS exception, mounted by PVE at `/mnt/pve/nas-rotom-proxmox-backup` from UNAS `Rotom_Proxmox_Backup`; it is `content backup` only, negotiated NFSv3, and has no Proxmox `/etc/fstab` entry. A small read-only `rotom-temp-api.service` is the only Rotom monitoring helper on the host |
| Host stability baseline | BIOS `BECFL357.86A.0098.2026.0204.1428`; updated Intel microcode; `i915.enable_dc=0` retained; ConBee II disconnected; exact earlier reset cause not isolated |
| Rotom VM | VMID `100`, name `rotom`; q35 + OVMF; CPU `x86-64-v2-AES`; 8 vCPU; 12 GiB fixed RAM; no iGPU passthrough |
| Rotom VM storage | 100 GiB thin VirtIO SCSI disk on `local-lvm`; discard/iothread/SSD flags enabled; Debian root ext4; manual TRIM verified |
| Rotom guest OS/network | Debian 13.7 (`trixie`), kernel `6.12.107+deb13-amd64`; `rotom.casa`; VirtIO MAC `BC:24:11:97:10:47`; UniFi-reserved DHCP `192.168.1.69/24`; gateway/DNS `192.168.1.1` |
| Rotom administration | `jared` UID/GID `1000:1000`; sudo; `/usr/bin/zsh` + Oh My Zsh; Mac key-only SSH via `~/.ssh/id_ed25519_rotom`; password-auth policy unchanged |
| Docker runtime | Docker Engine Community `29.8.1`; containerd `2.3.6`; runc `1.5.1`; Compose `v5.5.1`; buildx `v0.37.1`; Docker/containerd enabled; `/var/lib/docker` on VM-local ext4; daemon-wide `json-file` logging `10m` × `3` |
| Current application state | JAR-31 restored the exact 16-container running set: Arcane, Cloudflare DDNS, Homepage, Glances, NPM, Jellyfin, Radarr, Sonarr, Prowlarr, qBittorrentVPN, Jared/Fran Palworld, Gamarr, Home Assistant, Homebridge, and Aloha Millworks. `jaredwines.com` remains intentionally absent/stopped |
| Downloader storage | Linux identity `downloaders` `901:5005`; `/mnt/nas-downloaders` currently mounts UNAS **`Downloader/.data`**; root `988:5005` mode `2770`; live sentinel and torrent tree verified |
| NAS recovery | Adapted `rotom-nas-docker-recovery.service` enabled. Real reboot: first attempt hit Media NFS not ready, configured retry succeeded and recovered qBittorrentVPN, Radarr, Sonarr, Gamarr, Jellyfin automatically |
| Backup administration | **Rotom Virtualization Baseline:** verified Proxmox VZDump `vzdump-qemu-100-2026_09_27-01_19_56.vma.zst` on dedicated `nas-rotom-proxmox-backup` plus guest Restic snapshot `74d3b0ca` in repository `799babb2`. VZDump passed full VMA verification; post-backup `restic check` passed 21/21 snapshots. Status API remains enabled/active on TCP `8787`; `unas-backup.timer` remains deliberately disabled/inactive |
| Homepage monitoring | Internal site-monitor names are pinned inside the Homepage container to `192.168.1.69`, eliminating post-restore `EAI_AGAIN` failures without weakening Host validation. Backup widget uses `http://192.168.1.69:8787/backup-status`; Disk Usage uses VM `disk:sda`; CPU Temperature graphs Proxmox `coretemp` `Package id 0` through the read-only `192.168.1.68:8788` Glances-compatible sensor bridge |
| Pre-migration recovery | JAR-24 Restic snapshot `fbe1838e769cb740294ec0bf1017088a47ca5b97560083382af7cfcbebb2897f`; JAR-25 raw NVMe image SHA-256 `b62b2c8210bb2a6446abc29c5767c7d8531f547dfd2ec3f0259dbee8d74c5e90` |
| Phase B exit / next boundary | **Rotom Virtualization Baseline accepted.** Phase B virtualization gate is complete; JAR-34 may begin `/srv/rotom` redesign work. Separately decide when to enable the Restic timer and Sunday updater, define long-term Proxmox backup scheduling/retention under JAR-47, recreate Fran if desired, and continue iGPU/ConBee stability follow-up |

## 3. Current Workload Ownership — Restored Through JAR-31

JAR-31 restored the production Compose/application layer into the Debian VM while retaining the Phase B service-account split. The Linux-side download identity is **`downloaders`** at UID:GID `901:5005`; current active Prowlarr/qBittorrent project discovery uses `/home/downloaders/docker`. Historical sections retain the old `downloads` name where they describe pre-migration state.

ChatGPT-controlled browser work is not part of the Rotom administration or documentation workflow. Available Sources replacements are handled manually by Jared when needed. Apple Passwords/iCloud Passwords remains the credential source of truth; never record credential values or authentication material in the project documentation. See document 06 for the standing Mac and documentation workflow.

| Account | Current responsibility |
|---|---|
| infra | Arcane, Cloudflare DDNS, Homepage, Glances, Nginx Proxy Manager |
| media | Jellyfin, Radarr, Sonarr |
| game | Palworld servers for Fran and Jared; Gamarr |
| smarthome | Home Assistant and Homebridge |
| documents | Service-account skeleton reserved for future document workloads; no application deployed |
| downloaders | Active Prowlarr/qBittorrentVPN domain; UID:GID `901:5005`; `/mnt/nas-downloaders` currently mounts UNAS `Downloader/.data` |
| web | Aloha Millworks active; Jared Wines project retained but intentionally undeployed |
| filesync | Service-account skeleton reserved for future synchronization workloads; built-in Linux `sync` remains untouched |
| apps | Service-account skeleton reserved for future application workloads |
| auth | Service-account skeleton reserved for future authentication workloads |
| jared | Interactive administrator with full sudo; Fran's pre-migration administrator identity is preserved but not yet recreated in the VM |

The pre-migration system granted Docker group GID `984` to all named operational accounts. JAR-30 deliberately did not recreate that root-equivalent privilege model. A fresh 2026-09-27 post-boot read again returned `docker:x:989:` with no members, confirming the current VM still uses `sudo docker` rather than broad Docker-group membership. Historical GID `984` membership must not be inferred current.

### Current service identities, storage, and application state

The ten storage-facing service accounts retain their JAR-29 numeric identity contract. Current workload restoration did not require broadening ordinary NFS permissions.

| Account | UID : primary GID | Current NAS mount | Current state |
|---|---:|---|---|
| media | 127 : 5000 | `/mnt/nas-media` | Active Jellyfin/Radarr/Sonarr owner; NFSv3 on demand; ordinary storage boundary retained |
| game | 995 : 5001 | `/mnt/nas-game` | Active Jared/Fran Palworld + Gamarr owner; both Palworld containers healthy after final reboot |
| infra | 997 : 5002 | `/mnt/nas-infra` | Active core infrastructure owner; Arcane restored with preserved named volume |
| smarthome | 126 : 5003 | `/mnt/nas-smarthome` | Active Home Assistant/Homebridge owner; application state remains local |
| documents | 900 : 5004 | `/mnt/nas-documents` | Restored storage boundary; no application deployed |
| downloaders | 901 : 5005 | `/mnt/nas-downloaders` | Active Prowlarr/qBittorrentVPN owner; current backing export `Downloader/.data`; qBittorrent `wg0` verified after reboot |
| web | 902 : 5006 | `/mnt/nas-web` | Aloha active; Jared Wines intentionally undeployed |
| filesync | 903 : 5007 | `/mnt/nas-filesync` | Restored storage boundary; no application deployed |
| apps | 904 : 5008 | `/mnt/nas-apps` | Restored storage boundary; no application deployed |
| auth | 905 : 5009 | `/mnt/nas-auth` | Restored storage boundary; no application deployed |

All ten service-home shortcuts remain `/home/<service>/nas-<service> -> /mnt/nas-<service>`, including `/home/downloaders/nas-downloaders`. Backup and Shared Drive remain separate root-controlled mounts. Fran is not yet recreated in the new VM.

### Preserved pre-migration identities and NAS separation

The table below preserves the final pre-migration names and paths for recovery history. JAR-6 completed the dedicated service-storage identity pattern for the eight previously uncommissioned service accounts. JAR-29 later recreated the numeric IDs in the VM and renamed only the Linux-side `downloads` identity/path to `downloaders` while preserving `901:5005`.

| Account | Pre-migration UID : primary GID | Pre-migration NAS mount | Historical status |
|---|---:|---|---|
| media | 127 : 5000 | `/mnt/nas-media` | Current at pre-migration baseline; unchanged by JAR-6 |
| game | 995 : 5001 | `/mnt/nas-game` | Current at pre-migration baseline; unchanged by JAR-6 |
| infra | 997 : 5002 | `/mnt/nas-infra` | JAR-6 commissioned, empty storage boundary |
| smarthome | 126 : 5003 | `/mnt/nas-smarthome` | JAR-6 commissioned, Home Assistant/Homebridge remained local |
| documents | 900 : 5004 | `/mnt/nas-documents` | JAR-6 commissioned, empty storage boundary |
| downloads | 901 : 5005 | `/mnt/nas-downloads` | Active Prowlarr/qBittorrentVPN domain and centralized torrent storage after JAR-21 |
| web | 902 : 5006 | `/mnt/nas-web` | JAR-6 commissioned, website content remained local |
| filesync | 903 : 5007 | `/mnt/nas-filesync` | JAR-6 commissioned, empty storage boundary |
| apps | 904 : 5008 | `/mnt/nas-apps` | JAR-6 commissioned, empty storage boundary |
| auth | 905 : 5009 | `/mnt/nas-auth` | JAR-6 commissioned, empty storage boundary |

At the pre-migration baseline, all named operational accounts were members of Docker group GID `984`. That historical privilege model is restoration evidence only; it is not current JAR-30 guest membership.

At the pre-migration baseline, Jared was `1000:1000` and Fran was `1001:1001`; only Jared is currently recreated in the Debian VM. Their existing personal UNAS drives are outside the JAR-6 NFS service-share scope and are not mounted as `/mnt/nas-jared` or `/mnt/nas-fran`. Seven of the eight JAR-6 service shares remain boundary-only. JAR-21 activated `/mnt/nas-downloads` as the centralized torrent store for the Downloads service account. Each JAR-6 share root remains numeric owner UID `988`, matching service GID `5002`–`5009`, mode `2770`; ordinary unrelated service-account access was functionally denied. Media remains the final movie/show library domain and Game remains the final game-library domain; qBittorrent no longer writes torrent data to either share.

At the pre-migration baseline, each of the ten service homes had a verified explicit convenience link using `/home/<service>/nas-<service> -> /mnt/nas-<service>`. Those paths were `/home/media/nas-media`, `/home/game/nas-game`, `/home/infra/nas-infra`, `/home/smarthome/nas-smarthome`, `/home/documents/nas-documents`, `/home/downloads/nas-downloads`, `/home/web/nas-web`, `/home/filesync/nas-filesync`, `/home/apps/nas-apps`, and `/home/auth/nas-auth`; JAR-29 current state uses `/home/downloaders/nas-downloaders` instead of the old Downloads shortcut. The links were initially created as generic `nas` names and then renamed in place; supplied verification confirmed every final link target and confirmed the intended service identity could traverse its shortcut, ending with `ALL NAS SHORTCUT RENAMES: PASS`. These links do not replace the canonical `/mnt/nas-*` paths; operational configuration, Docker binds, systemd dependencies, storage guards, and troubleshooting should continue to use the real mountpoints.

At the pre-migration baseline, the Palworld service identity was `game`, UID `995`, primary GID `5001`, home `/home/game`, and a member of Docker group GID `984`. The old passwd/group names and `/home/game-server` were absent. On 2026-09-19 both Palworld Compose files were changed to explicit `PUID=995` / `PGID=5001`, validated, and recreated against the existing trees. Both containers returned healthy and their application processes resolved as `game:game`. A stopped-state checksum manifest covering 234 `.sav` files compared with no differences after recreation. Their verified world IDs remain `DB40338954B844C28CEA21471A392F98` for Jared and `396F5898378F4E9CAE89461F403653D9` for Fran as restoration evidence; JAR-31 later restored both servers in the current VM and verified them healthy after reboot.

The adopted access policy keeps `jared` outside the media group. Administer media by connecting from the Mac with `ssh rotom.casa`, then running `sudo -iu media` on Rotom as `jared`. A final 2026-09-19 `id jared` / `getent group media` check verified that Jared is not a supplementary member of media GID `5000`. Media's `127:5000` identity and Jared's ordinary NFS access denial are also verified. See 07 for the recorded identity and access model.

## 4. Preserved Pre-Migration Docker and Application-Network Baseline

- **Not current runtime:** the bullets in this section describe the verified pre-migration workload baseline to be restored in later tickets.
- Containers used `unless-stopped` restart behavior. Arcane, Homepage, both Palworld services, and Gamarr currently report healthy Docker checks; Gamarr's healthcheck is image-provided and no explicit Compose `healthcheck:` directive is present.
- Most applications use an individual Compose-created bridge network. JAR-22 verified the currently present Docker network set no longer includes the historical `jaredwinescom_default` network. The active named application networks remain Radarr, qBittorrentVPN, Jellyfin, Sonarr, Prowlarr, Arcane, Homepage/Glances, Aloha Millworks, Jared Palworld, Fran Palworld, and Gamarr; the 2026-09-19 subnet/IP map remains historical evidence for those networks unless refreshed separately.
- Nginx Proxy Manager, Home Assistant, Homebridge (`homebridge` container), and Cloudflare DDNS use host networking.
- Docker NAT publishes bridge services on host ports. Glances is loopback-only on 61208.
- Nginx Proxy Manager listens on TCP 80, 81, and 443. It generally proxies `rotom.casa` requests to Rotom's LAN address and a host-network listener or published Docker port. Gamarr is now included through `gamarr.rotom.casa -> 192.168.1.69:6767`; the route uses NPM certificate ID `24`, forces SSL, has no access list, and returned HTTP `302` through the verified HTTPS path.
- Cloudflare DDNS manages rotom.casa each minute and at startup. It is unproxied and does not update IPv6.
- LAN DNS resolves rotom.casa services to 192.168.1.69; public DNS resolved the domain to 68.8.40.225.
- On 2026-09-19 the Homebridge container name was corrected from historical `homebrige` to `homebridge`. Homebridge started successfully after the rename and restored its cached accessories; a whole-home scan found no remaining old-spelling references.

## 5. Storage Summary

| Path | Current role |
|---|---|
| `/mnt/nas-media` | NFSv3 `Media/.data`; root `988:5000` mode `2770`; active Jellyfin/Radarr/Sonarr final-library domain |
| `/mnt/nas-game` | NFSv3 `Game/.data`; root `988:5001` mode `2770`; active Gamarr final-library domain |
| `/mnt/nas-infra` | Infra service share; root `988:5002` mode `2770`; application configs remain local under `/home/infra` |
| `/mnt/nas-smarthome` | Smarthome service share; root `988:5003` mode `2770`; HA/Homebridge state remains local under `/home/smarthome` |
| `/mnt/nas-documents` | Documents service share; root `988:5004` mode `2770`; no application deployed |
| `/mnt/nas-downloaders` | NFSv3 **`Downloader/.data`**; root `988:5005` mode `2770`; active qBittorrent torrent domain with sentinel and torrent categories verified |
| `/mnt/nas-downloads-media-ro` | Active read-only bindfs compatibility view sourced from `/mnt/nas-downloaders` for Media applications |
| `/mnt/nas-downloads-game-ro` | Active read-only bindfs compatibility view sourced from `/mnt/nas-downloaders` for Game applications |
| `/mnt/nas-web` | Web service share; root `988:5006` mode `2770`; website content remains local under `/home/web` |
| `/mnt/nas-filesync` | Filesync service share; root `988:5007` mode `2770`; no application deployed |
| `/mnt/nas-apps` | Apps service share; root `988:5008` mode `2770`; no application deployed |
| `/mnt/nas-auth` | Auth service share; root `988:5009` mode `2770`; no application deployed |
| `/mnt/nas-rotom-restic-backup` | NFSv3 `Rotom_Restic_Backup/.data`; mounted share root `988:988` mode `0700`; canonical repository `/mnt/nas-rotom-restic-backup/rotom-restic-backup`; JAR-33 baseline snapshot `74d3b0ca` verified; automatic timer still disabled |
| `/mnt/nas-shared-drive` | NFSv3 Shared Drive; contains JAR-25 raw NVMe rollback image; outside host Restic because `/mnt` is excluded |
| `/home/infra/docker` | Active core infrastructure Compose trees: Arcane, DDNS, Homepage/Glances, NPM |
| `/home/media/docker` | Active Jellyfin/Radarr/Sonarr Compose/config trees |
| `/home/downloaders/docker` | Active Prowlarr/qBittorrentVPN Compose/config trees; Arcane mounts this root read-only for discovery |
| `/home/smarthome/docker` | Active Home Assistant/Homebridge Compose/config trees |
| `/home/game/docker` | Active both Palworld projects + Gamarr; Palworld saves remain critical persistent data |
| `/home/web/docker` | Aloha active; Jared Wines project retained but intentionally undeployed |
| `/var/lib/docker` | Current Docker runtime root on VM-local ext4; not NFS-backed |
| `/var/lib/docker/volumes` | Current Docker named-volume store; includes restored `arcane_arcane-data` |
| `/var/backups/system-info` | Active backup staging/recovery inventory path; all three required SQLite sources were present and first VM-era snapshot `5b4ed601` completed successfully |

All twelve canonical NFS mountpoints remain guest fstab/systemd automounts. The 2026-09-27 post-boot audit freshly verified all twelve generated `.automount` units active and the current NFS sources, including `Downloader/.data` and `Rotom_Restic_Backup/.data`. JAR-31 additionally restored the two bindfs compatibility views and the targeted NAS/Docker recovery helper; fresh Media/Game traversal checks again passed. The former `/mnt/nas-rotom-backup` export is no longer part of current Rotom configuration: it is unmounted and the old UNAS `Rotom_Home_Server_Backup` share is retained only as rollback data. The mounted `/mnt/nas-rotom-restic-backup` NFS root freshly verified as numeric `988:988` mode `0700`. The audit did not isolate the underlying local directory while fully unmounted, so its local-directory mode remains non-authoritative unless separately checked.

Historical JAR-21/JAR-29 records name `/mnt/nas-downloads` and/or `Downloads/.data`; those remain historical recovery evidence. Current live fstab, generated mount unit, active NFS source, and `showmount` identify `/mnt/nas-downloaders -> Downloader/.data`.

### JAR-22 pre-migration audit baseline — 2026-09-22

JAR-22 captured a read-only bare-metal baseline under `/home/jared/audits/jar-22-2026-09-22`. Verified hardware/runtime facts are Linux Mint 22.3, kernel `7.0.0-31-generic`, UEFI boot, Intel Core i5-8259U with 4 cores / 8 threads and VT-x/VMX, one 232.9G NVMe, and a 232.4G ext4 root filesystem using 52G with 165G available. The measured major local persistent-data paths total approximately 21.6 GiB; `/home/game` accounts for 15G, `/root` 2.9G, and `/home/web` 1.9G. These measurements are sizing evidence for the future Rotom VM and are not a claim that every byte on `/` is application-persistent data.

Network identity is now directly reconciled: `eno1` is `192.168.1.69/24`, MAC `1c:69:7a:0e:f2:f7`; the user confirmed in UniFi that `.69` is reserved to that MAC and that `.68` is open/available for the proposed Proxmox management address. No reservation was changed. All twelve NFS mounts and all service UID/GID mappings were also reverified. The audit directory is under the existing `/home` Restic source scope; JAR-24 later explicitly verified that exact JAR-22 audit tree inside final pre-migration snapshot `fbe1838e`.


### JAR-23 preservation and resilience baseline — 2026-09-25

JAR-23 completed application-aware preservation acceptance without changing the host OS, LAN identity, service UID/GID model, NAS export definitions, or Docker application topology. Both Palworld worlds have stopped-state checksum manifests in `/home/jared/audits/jar-23-2026-09-23` (Jared: 135 `.sav` entries; Fran: 108), Home Assistant/NPM SQLite staging was refreshed and integrity-checked, website content was checksummed, and every current stateful Compose project/persistence path was inventoried. The final consolidated check reported 16 expected running containers and zero failed systemd units; Jared Wines remains intentionally undeployed with its Compose project retained and no container object.

A September 23 reboot sequence exposed a boot-time NFS/automount race: NetworkManager wait-online completed, but Docker restore reached NAS-backed bind mounts before the real NFS mounts were usable, leaving both Downloads bindfs services and Radarr/Sonarr/Gamarr stopped. Data and mount contracts were intact once NFS recovered. JAR-23 restored the existing services and added the enabled host recovery helper `/usr/local/sbin/rotom-nas-docker-recovery` with unit `/etc/systemd/system/rotom-nas-docker-recovery.service`. Its manual healthy-state run passed; reboot validation is deferred. The helper is targeted to Downloads/Media/Game NAS-backed workloads and does not make all Docker services depend on the NAS.

### JAR-24 final pre-migration Restic baseline — 2026-09-25

JAR-24 completed the final host-level Restic recovery gate without changing Rotom's OS, service topology, LAN identity, service UID/GID model, NAS export definitions, Docker Compose files, or backup retention policy. After a stale lock from an aborted read-only helper was confirmed orphaned, Restic's supported `unlock` removed that one stale lock; the canonical `backup-to-nas` procedure then saved final snapshot `fbe1838e` (`fbe1838e769cb740294ec0bf1017088a47ca5b97560083382af7cfcbebb2897f`) at `2026-09-25 18:14:22 PDT` and completed the existing 7-daily / 4-weekly / 12-monthly retention/prune cycle.

The final snapshot explicitly contains both preservation audit trees (`/home/jared/audits/jar-22-2026-09-22` and `/home/jared/audits/jar-23-2026-09-23`), all three staged SQLite databases under `/var/backups/system-info/sqlite`, both current Palworld `Level.sav` files, and the JAR-23 NAS recovery helper/unit. `restic check` and full `restic check --read-data` passed, reading all 971 packs with no errors; a representative isolated restore and restored-database `PRAGMA quick_check` also passed. `/mnt` remains excluded, so NAS-resident data is still a separate protection domain. Audit evidence is under `/home/jared/audits/jar-24-2026-09-25`.


### JAR-25 bare-metal rollback baseline — 2026-09-25

JAR-25 completed the preservation-phase whole-disk rollback gate without changing the installed OS, partition table, Docker Compose definitions, LAN identity, service UID/GID model, or NAS export definitions. Read-only identification fixed the source as `/dev/nvme0n1`, Samsung SSD 970 EVO Plus 250GB, serial `S59BNM0R702702E`, exact size `250059350016` bytes. The destination is the physically separate UNAS Shared Drive mounted at `/mnt/nas-shared-drive`.

The final raw image is `/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25/rotom-nvme-live-pre-proxmox-2026-09-25.img`. Its logical size exactly matches the source disk (`250059350016` bytes); acquisition-stream and complete stored-image reread hashes both equal `b62b2c8210bb2a6446abc29c5767c7d8531f547dfd2ec3f0259dbee8d74c5e90`. Stored-image partition inspection exposed the EFI System Partition and Linux filesystem partition, and restore instructions were written alongside the image. The JAR-24 final Restic snapshot and JAR-23 critical artifacts were reverified before acquisition.

The image was intentionally taken while Mint's root filesystem remained mounted. All 16 running Docker workloads were stopped after filesystem/maintenance preparation and restored after the first imaging pass; final verification reported the original 16-container set running and zero failed systemd units. This is therefore a **best-effort live / crash-consistent** rollback image, not an offline frozen image. A real bare-metal restore has not yet been rehearsed. The image and Restic repository use different shares but the same physical UNAS appliance, so a UNAS hardware failure can affect both recovery paths.

## 6. Current Backup and Maintenance Summary

The Phase B recovery gate is now established as **Rotom Virtualization Baseline** rather than the earlier JAR-31 control-layer-only state. Guest-level Restic and whole-VM Proxmox recovery are deliberately separate.

- **Whole-VM baseline:** PVE storage `nas-rotom-proxmox-backup` is active at `/mnt/pve/nas-rotom-proxmox-backup`, backed by UNAS `Rotom_Proxmox_Backup/.data` exported only to Proxmox `192.168.1.68`. It is PVE-managed, `content backup` only, negotiated NFSv3, and is not present in Proxmox `/etc/fstab`. A write/read/delete test passed. Verified archive `vzdump-qemu-100-2026_09_27-01_19_56.vma.zst` is listed by Proxmox at `45,337,727,897` bytes (about 43 GB); snapshot-mode backup used guest-agent `fs-freeze`/`fs-thaw`, completed successfully, and full `zstd -dc | vma verify` returned exit code `0`.
- **Guest Restic baseline:** repository `799babb2` remains at `/mnt/nas-rotom-restic-backup/rotom-restic-backup` on UNAS `Rotom_Restic_Backup/.data`, exported only to Rotom `192.168.1.69`. JAR-33 manual `backup-to-nas` created snapshot `74d3b0ca` at `2026-09-27 01:51:17` (host `rotom`, tag `automatic`, 17.398 GiB); post-run `restic check` passed all 21 snapshots. The canonical run also applied the existing `7 daily / 4 weekly / 12 monthly` retention policy, removed redundant same-day snapshots `e512e6b5` and `8724c798`, and pruned about `56.849 MiB`.
- **Automation state:** `unas-backup-status-api.service` remains enabled/active on TCP `8787`; `unas-backup.timer` remains deliberately **disabled/inactive**. `/usr/local/sbin/weekly-linux-update` is restored and syntax-valid, but `/etc/cron.d/weekly-linux-update` remains absent. No scheduled Proxmox backup jobs were configured during JAR-66/JAR-33; long-term PVE backup schedule/retention remains deferred to JAR-47. Fran's historical backup sudoers file remains absent because Fran has not been recreated.
- **Rollback layers retained:** first VM-era Restic snapshot `5b4ed601`, final pre-migration snapshot `fbe1838e`, the old UNAS `Rotom_Home_Server_Backup` point-in-time copy, and the JAR-25 bare-metal NVMe image remain retained as documented recovery artifacts.

The historical bare-metal schedule, retention policy, application staging, Palworld jobs, Arcane automation, and backup execution details remain documented in `05-Backup-and-Restore.md` and `06-Maintenance-and-Automation.md`; current commissioning status is explicit there.

## 7. Current Startup and Dependency Summary

1. The Debian VM still boots independently of NAS availability through the fstab/systemd automount design.
2. Docker and containerd are enabled and returned active after the JAR-31 acceptance reboot. Docker runtime state remains local at `/var/lib/docker`.
3. The adapted `rotom-nas-docker-recovery.service` is enabled with retry-on-failure behavior. On the JAR-31 reboot its first invocation reached `/mnt/nas-downloaders` but failed when `/mnt/nas-media` was not ready; systemd retried about 30 seconds later, then the helper mounted/verified Downloader, Media, and Game storage plus both read-only bindfs views and automatically recovered qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin. Final unit state was `active (exited)`, `Result=success`, `ExecMainStatus=0`.
4. The two compatibility-view services are active: `/mnt/nas-downloads-media-ro` and `/mnt/nas-downloads-game-ro` are read-only bindfs views sourced from current `/mnt/nas-downloaders`.
5. The exact pre-reboot set of 16 running containers was restored after reboot; critical healthchecks passed, qBittorrent `wg0` exists, core HTTP checks passed, and zero systemd units were failed.
6. The Proxmox host remains independent of Rotom application NFS and application Docker; no application share is mounted there.

## 8. Current Architecture-Level Findings and Limitations

- The Phase B application layer is restored. The final running set is 16 containers; Jared Wines remains intentionally undeployed.
- UID/GID `901:5005` is named `downloaders`, with `/home/downloaders` and `/mnt/nas-downloaders`. **Current live evidence identifies the backing UNAS export as `Downloader/.data`, not the earlier JAR-29/JAR-30 `Downloads/.data` claim.** Current fstab, generated mount unit, active NFS source, and `showmount` output agree.
- `/mnt/nas-downloaders` root metadata is `988:5005` mode `2770`; `.rotom-qbt-nas-ready` and `torrents/{games,incomplete,movies,shows}` were observed. Historical `Downloads/.data` references remain dated historical evidence.
- Docker's root `/var/lib/docker` remains VM-local ext4. The application networks now occupy the verified bridge map recorded in documents 02/03; Homepage is `172.24.0.0/16`, Jared Palworld was deliberately moved to `172.27.0.0/16`, and Arcane is `172.28.0.0/16`.
- Jared Palworld was stopped cleanly for the network relocation, a stopped-state archive was made, five active-save paths matched the pre-move gate, and the server returned healthy with the preserved image and zero restarts.
- A fresh 2026-09-27 post-boot read verified the package-created Docker group remains GID `989` with no members; the historical pre-migration GID `984` broad-membership model remains retired.
- Backup controls are restored but automated backup execution remains intentionally held back. The Restic repository must never be manually reorganized.
- Homepage's local backend intentionally enforces `HOMEPAGE_ALLOWED_HOSTS=rotom.casa`; `curl http://127.0.0.1:3001/` returning 400 is expected host validation, while the same request with `Host: rotom.casa` and the HTTPS apex both returned 200.
- Mac key-only SSH remains verified for Rotom and Proxmox; password-authentication policy is unchanged and no key contents/passphrases are recorded.

### 2026-09-27 live post-boot verification refresh

A read-only audit after the 02:20 PDT reboot refreshed the current Phase B evidence: zero failed units; Docker `29.8.1` / Compose `v5.5.1`; 16/16 containers running with zero restart counts; current bridge subnets `172.18.0.0/16` through `172.28.0.0/16`; qBittorrent `wg0` present; all ten service accounts and `/home/<service>/nas-<service>` symlinks correct; Docker group `989` empty; all twelve NFS automounts active; both bindfs compatibility traversals passing; mounted Restic root `988:988` mode `0700`; backup timer disabled/inactive; weekly-update cron absent; core HTTP probes successful; and the Proxmox temperature API reachable. Arcane automation settings were also freshly reread and are current as documented in `06-Maintenance-and-Automation.md`.

## 9. Outstanding Decisions and Needs Verification

JAR-31 compatibility/application restoration and the JAR-33 virtualization baseline are complete. The remaining items are either intentional commissioning decisions or facts that still need current evidence:

- **Intentional / not yet commissioned:** Fran is not recreated and her historical backup sudoers rule remains absent; `unas-backup.timer` remains disabled/inactive; the reboot-capable weekly updater script is present but its cron schedule is absent; long-term Proxmox backup scheduling/retention remains owned by JAR-47; Rotom VM iGPU passthrough remains uncommissioned.
- **Needs Verification — UNAS metadata persistence:** service-share root UID/GID/mode persistence across a future UNAS/UniFi Drive restart/update has not been observed. Resolve after such an event with read-only `stat`/`findmnt` checks from Rotom plus UNAS export inspection; do not change ownership merely to make the result match documentation.
- **Needs Verification — prior Proxmox resets:** the ConBee II remains intentionally disconnected and the RPD does not isolate one causal explanation for the earlier resets. Read-only logs can narrow evidence (`journalctl`, Proxmox task/system logs), but a single cause should not be claimed without new evidence.
- **Recovery/operations follow-up:** a full Palworld disaster-recovery rehearsal, alert-delivery test, certificate-renewal observation, and final public-exposure review remain unperformed or incompletely evidenced. These are tests/decisions, not contradictory current-state facts.

## 10. Rotom Project Documentation Model

**Rotom Project Documentation** is the proper name for **every file intentionally kept in this project's Available Sources**. Available Sources defines the complete maintained documentation set and is the membership boundary regardless of filename, extension, file format, document type, or purpose. A file does not need to be Markdown, a guide, a standard, or a numbered reference file to belong to the set.

The numbered files **00–08** form the **Core Numbered Reference Set**. A complete Available Sources inventory on 2026-09-27 confirms they are also the entire current Rotom Project Documentation set: nine files total, with no additional current non-numbered source. They remain the primary current-state references for change history, server inventory, Docker services, networking, storage, backup/recovery, maintenance/automation, users/permissions, and the directory tree. `Rotom-Architecture-and-Deployment-Standard.md`, `Rotom-Home-Server-Guide-Maintenance.md`, and `Rotom-Home-Server-Manual-Maintenance.md` were removed from Available Sources by user decision on 2026-09-20 and therefore are no longer current Rotom Project Documentation. The same membership rule applies automatically to any file intentionally added to or removed from Available Sources in the future.

Within the Rotom project, **RPD** is the accepted acronym for **Rotom Project Documentation**. “Project docs,” “rotom project docs,” “project documentation,” “project documents,” and “project reference files” are also informal aliases unless a narrower subset is specified. Capitalization does not change the meaning. Raw command output, audit attachments, chat exports, screenshots, generated packages, local working files, and other material that exists **outside** Available Sources remains supporting or temporary material rather than Rotom Project Documentation. If such a file is intentionally placed in Available Sources, it becomes part of Rotom Project Documentation by membership.

Consult the relevant available documents before giving Rotom-specific advice or documenting changes. Respect their evidence dates and distinguish current recorded configuration, historical findings, proposals, and unverified details. If a required document is unavailable, identify the gap rather than assuming its contents.

The canonical standing request is **“Update the Rotom Project Documentation using everything we changed, verified, and learned in this chat.”** The shorthand **“Update the RPD using everything we changed, verified, and learned in this chat.”** is exactly equivalent. Either phrase means review the current chat and the complete Available Sources set; determine which files are materially affected; update every affected file while leaving unaffected files unchanged; record every substantive documentation change in `00-Rotom-Change-Log.md`; then provide replacement copies of **every current Rotom Project Documentation file** and one ZIP containing the complete current set. The ZIP filename is **`Rotom-Project-Documentation-YYYY-MM-DD.zip`**, using the documentation-update date in Rotom's `America/Los_Angeles` project timezone. Review and packaging apply to every file type and extension. Preserve existing filenames whenever possible and do not add checksum manifests, packaging metadata, or other extra files unless explicitly requested. Jared manually replaces files in `/Users/jared/Documents/ChatGPT/Rotom-Home-Server` and the project's **Available Sources** list. ChatGPT must not automatically write to the Mac documentation folder, replace or synchronize Available Sources, open or control a browser for synchronization, deploy the documentation to Rotom, or claim a manual destination is current until Jared confirms it. Generated packages and separate artifacts belong under `Downloads/`; the local `sources/` mirror stays read-only. This request does not by itself change running services or any other live Rotom state.

The September 16 update also incorporates “Change Homepage URL” plus the user's confirmation that Ghostty was removed. Homepage routing is verified by the supplied chat output. Earlier browser-workflow decisions are historical and are superseded by the current no-browser-automation policy.

For project-wide Rotom command execution guidance, large or multi-step command blocks should be isolated in a child Bash process by default so errors and shell-control statements cannot terminate the interactive SSH session. Small single-purpose commands may run directly. The detailed convention is maintained in `06-Maintenance-and-Automation.md`.

## 11. Current Source Map

Available Sources determines the complete Rotom Project Documentation membership. The map below records the current Project-backed Available Sources set for navigation, but membership does not depend on a filename appearing in this table. When the Available Sources set changes, update this map as part of the next substantive documentation update. Follow the maintenance guide in [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md) and update its dated history in the same task.

### Core Numbered Reference Set

| Area | Project source |
|---|---|
| Documentation change history and maintenance guide | [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md) |
| Documentation definition, index, and server overview | [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md) |
| Docker services, Compose paths, images, container/runtime deployment | [02-Docker-Services.md](02-Docker-Services.md) |
| Host network, DNS, ports, Docker networks, NPM, Cloudflare | [03-Network-and-Domains.md](03-Network-and-Domains.md) |
| NAS exports, mounts, automounts, storage layout and contracts | [04-NAS-and-Storage.md](04-NAS-and-Storage.md) |
| Restic, backup scope, retention, verification, restore boundaries | [05-Backup-and-Restore.md](05-Backup-and-Restore.md) |
| Timers, updates, monitoring, automation, documentation workflow | [06-Maintenance-and-Automation.md](06-Maintenance-and-Automation.md) |
| Accounts, UID/GID, groups, sudo/Docker privilege, ACL/access boundaries | [07-Users-and-Permissions.md](07-Users-and-Permissions.md) |
| Observed filesystem topology and important paths | [08-Rotom-Directory-Tree.txt](08-Rotom-Directory-Tree.txt) |

### Current membership

The current Project-backed Available Sources set is exactly the nine-file Core Numbered Reference Set (`00`–`08`) listed above. There are no additional current non-numbered Rotom Project Documentation files. Future files intentionally added to Available Sources become Rotom Project Documentation automatically, regardless of file type or extension, and belong in the complete delivery/ZIP produced by the canonical documentation-update request; files removed from Available Sources cease to be current members without erasing their historical change-log references.

On Rotom, `/home/infra/documentation/README.md` is the directory-level index and `/home/infra/documentation/AGENTS.md` supplies operating instructions. A fresh 2026-09-19 top-level listing confirms those are the only files currently deployed there; none of the current nine Rotom Project Documentation files are present at that location. These server-side control files are not part of the current Available Sources set unless intentionally added there in the future.

## 12. Historical Verification Record — 2026-09-19 through 2026-09-21

### Evidence provenance

Evidence: Project sources 02 through 08, with read-only Rotom verification on 2026-09-15. UID/GID, NAS access, the authoritative documentation directory, and shared manual backup access were updated on 2026-09-16 from supplied command output and recorded decisions in “Torrent Seeding Explanation,” “Create Rotom docs setup guide,” and “Add Fran Backup Access.” On 2026-09-18, supplied live Rotom and UNAS output verified the NAS-backup access-protection deployment, Glances bind removal, Homepage/Glances recreation, root-run backup behavior, manual backups for Jared and Fran, and the current Homepage Docker network. Later the same day, a controlled Palworld service-account migration was completed and verified: `game-server` became `game` while retaining UID/GID `995:985`, the home moved to `/home/game`, both Palworld projects were recreated from the new path, both worlds were integrity-checked, and a post-migration Restic snapshot was verified. On 2026-09-19, the separate game-storage identity work was implemented: `game` kept UID `995` but moved to primary GID `5001`, both Palworld stacks were updated and verified at `995:5001`, and the UniFi `Game` export was commissioned as `/mnt/nas-game` with share root `988:5001` mode `2770` and an active systemd automount. Later on 2026-09-19, Prowlarr was migrated from the `media` service account to `rotom`: the active project moved to `/home/rotom/docker/prowlarr`, its persistent config/database was preserved, runtime identity was verified as `997:986` / `rotom:rotom`, port `9696` and `prowlarr.rotom.casa` remained unchanged, application/indexer tests passed, and the old `/home/media/docker/prowlarr` tree was retained for rollback. The same day the central downloader/Game-library deployment was completed: qBittorrentVPN moved from `/home/media/docker/qbittorrentvpn` to `/home/rotom/docker/qbittorrentvpn`, changed to `PUID=997` / `PGID=5000`, adopted `/media/torrents` and `/game/torrents`, gained verified NAS startup fail-closed guards, and retained verified WireGuard/kill-switch behavior; Sonarr and Radarr changed their NAS container root to `/media`. Gamarr was deployed under `/home/game/docker/gamarr` as `995:5001`, port `6767`, with `/mnt/nas-game -> /game`, and the Game NAS was populated with separate `library/` and `torrents/` trees. Later on 2026-09-19, the Homebridge container-name typo was corrected from `homebrige` to `homebridge`; Homebridge restarted successfully, restored its cached accessories, and a whole-home scan found no remaining old-spelling references. Unrelated inventory and health observations retain their original audit dates.

- System DMI identifies a Wortmann_AG `1009664;1400107` / PC-MICRO system with Intel `NUC8BEB` motherboard; this resolves the earlier apparent Wortmann-versus-NUC discrepancy.
- At the 2026-09-19 pre-migration audit, Jellyfin ran as UID/GID `127:5000`, could access `/dev/dri/card0` and `renderD128`, and its bundled FFmpeg passed synthetic H.264 QSV and VAAPI encode tests. The application configuration observed then was `HardwareAccelerationType=none`. These are historical bare-metal findings; current VM iGPU passthrough remains uncommissioned.
- Sonarr root folder is `/media/library/shows/`; Radarr root folder is `/media/library/movies/`.
- qBittorrent Media/Game NAS sentinels are present as mode `2555`, UID `997`, GID `5000`/`5001`. Startup/recreation fail-closed behavior is verified; live runtime NFS-loss behavior is not.
- At the 2026-09-19 audit, the NPM wildcard certificate served `*.rotom.casa` and `rotom.casa`, issued by Let's Encrypt and valid through 2026-12-10. Treat that validity window as dated evidence, not a perpetual current-certificate claim.
- A later comprehensive 2026-09-19 read-only audit reverified the then-current 17 service records, Docker networks, service mounts, UFW state, split DNS, DDNS, NPM routes, four NFSv3 mounts, Media/Game/Backup access, Restic, WOL, Docker logging, and the bare-metal `casper-md5check.service` failure. Those counts/firewall/failed-unit facts are historical and are superseded by the current VM sections above.
- Gamarr now has an active NPM route at `gamarr.rotom.casa -> 192.168.1.69:6767`; the NPM row is enabled with certificate ID `24`, forced SSL, no access list, and the HTTPS probe returned `302`. Gamarr also reports `healthy` from an image-provided healthcheck; its Compose file has no explicit `healthcheck:` key.
- The NPM database still contains the legacy Smart Hub/Portainer/OliveTin/PalTools records as `enabled=1`, but the inspected generated `proxy_host/*.conf` set does not contain those legacy IDs. Treat them as stale/unresolved database records rather than active generated routes until administrator intent is decided.
- `/home/infra/documentation` is the current control-document path after the 2026-09-21 identity migration; the migration preserved the prior ACL model and control-file tree.
- At the final bare-metal pre-migration baseline, `/etc/cron.d/weekly-linux-update` scheduled the updater for Sunday 04:00 as root. The script ran `apt-get update`, `upgrade`, `autoremove`, and `autoclean`, logged to `/var/log/weekly-linux-update.log`, and rebooted only when `/var/run/reboot-required` existed. **Current VM state differs:** the script is restored, but the cron file is intentionally absent; see section 6 and document 06.


## 12D. JAR-6 Per-Service NAS and Storage-GID Verification — 2026-09-21

JAR-6 migrated the eight service primary GIDs to `5002`–`5009` with UIDs unchanged, reconciled matching ownership in each service home, changed Arcane live data to Infra GID `5002`, and updated Arcane/Cloudflare-DDNS/Prowlarr identity settings while leaving qBittorrent `PGID=5000` unchanged. Eight new Isolated-mode UNAS shares were commissioned at `/mnt/nas-infra`, `/mnt/nas-smarthome`, `/mnt/nas-documents`, `/mnt/nas-downloads`, `/mnt/nas-web`, `/mnt/nas-filesync`, `/mnt/nas-apps`, and `/mnt/nas-auth`. Their roots are `988:5002` through `988:5009`, mode `2770`, and their exact NFSv3 `sec=sys` exports are persisted in `/etc/fstab` with the standard systemd automount pattern. Positive/negative access testing passed for all eight shares and all disposable objects were removed. A normal Rotom reboot confirmed service GIDs, automounts, exact exports, access behavior, Backup/Shared Drive, and qBittorrent protections persist. No application or personal data was migrated to the new shares. UNAS-side root metadata persistence across a future UNAS/UniFi Drive restart/update remains deferred.

## 12C. JAR-19 Website Identity Migration Verification — 2026-09-21

JAR-19 migrated both website projects from the legacy `web-host 128:130` identity and `/home/web-host` to the existing `web 902:902` service account under `/home/web/docker`. Aloha was recreated from `/home/web/docker/alohamillworks.com`; Docker Compose labels point to the new working directory/config path, local port `7778` returned HTTP 200, and the proxied HTTPS site also returned 200. Jared Wines was migrated to `/home/web/docker/jaredwines.com` and deliberately left stopped, matching its pre-change state.

After a verified root-only rollback copy was created at `/root/jar-19-20260921-024634`, final guards found no UID 128 process, no running Docker bind using `/home/web-host`, no required operational path reference outside expected Linux account records, and no persistent UID/GID `128:130` dependency outside the retiring home. The `web-host` user, group, and home were then removed. Final Restic snapshot `e886c55d` completed successfully at `2026-09-21 02:47:55 PDT`; normal retention removed intermediate same-day snapshot `2c22607e`. No NPM, DNS, TLS, firewall, published-port, or NAS deployment changes were made.

## 12B. Service-Account Identity Migration Verification — 2026-09-21

At this **intermediate pre-JAR-6 checkpoint**, live verification established `infra 997:986`, `smarthome 126:128`, and new service identities at same-number UID/GID `900`–`905`; JAR-6 later superseded those primary GIDs with `5002`–`5009`, and JAR-19 separately retired `web-host`. All service accounts retained Docker-group membership and the built-in Linux `sync` identity remained unchanged at UID `4`.

The eight affected Compose projects validate and run from `/home/infra/docker/...` or `/home/smarthome/docker/...`; Docker metadata and bind mounts show no old `/home/rotom` or `/home/smart-home` paths in the checked active configuration. The five application probes returned HTTP 200, qBittorrentVPN reported WireGuard `wg0`, and post-change Restic snapshot `b67540b0` completed successfully.

## 12A. JAR-9 Rollback Verification — 2026-09-21

At JAR-9 rollback closeout, active architecture again used `game` (`995:5001`) at `/home/game`; Palworld and Gamarr were restored under `/home/game/docker`, qBittorrentVPN again used Media/Game torrent and sentinel binds, and `/mnt/nas-game` was the active Game NFS share. `/mnt/nas-game-servers` was removed from `fstab` and unmounted. At rollback time the UNAS `Game_Servers` share was retained; a later JAR-6 UNAS inspection found the former `Game_Servers/.data` path absent, and JAR-6 did not delete it. Snapshot `b5f77aeb` remains historical recovery evidence for the pre-rollback JAR-9 state.

## 13. Canonical Fact Ownership

Use the detailed subject file as the authoritative first-update location for a fact. `01` should carry only the summary needed to understand Rotom as a whole.

| Fact category | Canonical detailed owner |
|---|---|
| Docker services, Compose paths, images, runtime deployment | `02-Docker-Services.md` |
| LAN, DNS, ports, Docker networks, Cloudflare, NPM routes | `03-Network-and-Domains.md` |
| NAS exports, mountpoints, fstab/systemd automounts, storage layout | `04-NAS-and-Storage.md` |
| Restic, backup scope, retention, verification, restore boundaries | `05-Backup-and-Restore.md` |
| Scheduled/routine operations and documentation-maintenance workflow | `06-Maintenance-and-Automation.md` |
| Linux accounts, UID/GID, groups, sudo/Docker privilege, ACL/access model | `07-Users-and-Permissions.md` |
| Observed filesystem topology and important paths | `08-Rotom-Directory-Tree.txt` |