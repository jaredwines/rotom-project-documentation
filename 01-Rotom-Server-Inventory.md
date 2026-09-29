# 01 - Rotom Server Inventory

**Documentation set:** Rotom Project Documentation  
**Document role:** Architecture/index starting point and high-level Rotom summary  
**Hosts:** PVE hypervisor `pve` and portable Debian VM `rotom`
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-09-29 — JAR-45 Web v2 convergence verified
**Related canonical sources:** `02-Docker-Services.md`, `03-Network-and-Domains.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`, `08-Rotom-Directory-Tree.txt`  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

## 1. Purpose and Scope

This document is the front-page architecture/index for Rotom. It summarizes the current system and points to the canonical subject document for detailed facts; it is not the canonical home for full account, container, network, mount, backup, automation, or filesystem registers.

Purpose: front-page index and quick-reference architecture summary.

**RPD state labels:** **Current** means the latest supported recorded state; **Historical** preserves dated evidence that may no longer apply; **Retired** means intentionally no longer active; **Proposed** is not implemented; **Needs Verification** means the RPD lacks enough current evidence and should not be treated as settled.

## 2. Current Rotom Summary

The accepted Phase B architecture now uses **PVE** as the product abbreviation and `pve` as the physical Proxmox VE host name. JAR-68 normalized the host identity and all active PVE-only backup/monitoring names, removed temporary `proxmox` SSH/hosts compatibility aliases after successful post-reboot acceptance, and refreshed the whole-VM recovery point. Historical sections preserve the earlier names only as dated evidence.

| Item | Verified value |
|---|---|
| Physical host | `pve`; FQDN `pve.rotom.casa`; Proxmox VE `9.2.20`; kernel `7.0.14-19-pve`; management IP `192.168.1.68/24` |
| Physical hardware | Intel NUC / Intel Core i5-8259U; 16 GB class RAM; Intel Quick Sync capable iGPU; Samsung 970 EVO Plus 250GB NVMe |
| PVE network | `vmbr0` `192.168.1.68/24`; gateway/DNS `192.168.1.1`; physical `nic0` Intel I219-V MAC `1c:69:7a:0e:f2:f7`; WOL `g`; TSO/GSO mitigation retained |
| PVE administration | Routine identity `jared`; canonical Mac aliases `pve` / `pve.rotom.casa`; `HostName 192.168.1.68`; key `~/.ssh/id_ed25519_pve`; PVE identity `jared@pam` has propagated `Administrator` access; matching root-authorized key removed; retain root only for emergency recovery |
| Rotom VM | VMID `100`, name `rotom`; q35 + OVMF; `x86-64-v2-AES`; 8 vCPU; 12 GiB RAM; 100 GiB thin VirtIO SCSI; `onboot: 1` |
| Rotom guest | Debian 13.7; `rotom.casa`; MAC `BC:24:11:97:10:47`; UniFi-reserved `192.168.1.69/24`; gateway/DNS `192.168.1.1` |
| RPD / Codex integration | Canonical maintenance policy is in document 00. Repository support now includes shared `AGENTS.md` and canonical `tooling/rpd`; the identical helper is installed at `/usr/local/bin/rpd` on Mac and Rotom and resolves the expected checkout from the current RPD Git tree or `~/.config/rpd/repository`. `rpd path` is the supported checkout resolver. Current physical checkouts are `/Users/jared/Documents/ChatGPT/rotom-project-documentation` on Mac and `/home/infra/documentation/rotom-project-documentation` on Rotom. Mac global Codex instructions are limited to the editor preference; Rotom `/home/jared/.codex/AGENTS.md` carries machine/live-state safety and points to the shared repository instructions and document 00. Fresh Codex discovery tests and `rpd` checks passed on both systems. Available Sources remains the RPD membership boundary; repository support files are not RPD members unless intentionally added there. |
| Docker runtime | Docker Engine `29.8.1`; Compose `v5.5.1`; 17 container objects; **15 intended running** because both Palworld containers are intentionally stopped to save resources |
| v2 Infra / proxy | Homepage, Glances, Arcane, Cloudflare DDNS, and the private Rotom documentation portal use independent Compose files under `/srv/rotom/stacks/infra`; v2 mutable state is under `/srv/rotom/appdata/infra`, secrets under protected `/srv/rotom/secrets/infra`. Homepage/Glances share `rotom-monitoring`; Homepage/Arcane and the read-only `rotom-docs` web service attach `rotom-proxy`; DDNS is bridge/outbound-only. NPM is retained as the production reverse proxy. |
| JAR-5 documentation portal | `/srv/rotom/stacks/infra/rotom-docs` defines the static portal and its controlled-publish script. Its generated source and site output are local under `/srv/rotom/appdata/infra/rotom-docs`; it publishes only a deliberate derived copy of the approved current nine-file RPD set and never mounts the canonical RPD checkout. NPM serves `docs.rotom.casa` through loopback-only port `8082` and a trusted-LAN allowlist. |
| JAR-10 Media v2 | Jellyfin, Radarr, Sonarr, Prowlarr, Gamarr, RomM, and retained qBittorrentVPN use v2 Compose modules under `/srv/rotom/stacks`. RomM state is local under Media appdata and its game library bind is read-only; NPM serves `romm.rotom.casa`. |
| JAR-42 Gameserver v2 | Independent Palworld Jared and Fran Compose modules are current under `/srv/rotom/stacks/gameserver`, with local live state under `/srv/rotom/appdata/gameserver`; protected environment files remain under the root-only Gameserver secrets boundary. Both containers are intentionally stopped after controlled validation. The former `/home/game/docker/palworld-server-*` rollback trees were explicitly retired on 2026-09-29. Reserved `/mnt/nas-gameserver` is not a live-world dependency. |
| JAR-43 Smart Home v2 | Home Assistant and Homebridge run from independent modules under `/srv/rotom/stacks/smarthome`, with authoritative mutable state under `/srv/rotom/appdata/smarthome`. They retain host networking for discovery and the established NPM compatibility routes. The prior `/home/smarthome/docker/{home-assistant,homebridge}` state/Compose trees are untouched rollback material; `/mnt/nas-smarthome` remains an unused reserved boundary. |
| JAR-45 Web v2 | Aloha Millworks runs from `/srv/rotom/stacks/web` with site content under `/srv/rotom/appdata/web` and joins `rotom-proxy`; its `7778` listener remains the required compatibility upstream for retained host-networked NPM. Jared Wines is migrated to v2 state/Compose but remains intentionally unstarted and unpublished. The former legacy Web trees were explicitly retired on 2026-09-29. |
| JAR-44 Documents foundation | `/srv/rotom/stacks/documents/DOMAIN-CONTRACT.md` defines the future Paperless contract: Documents `900:5004`, VM-local appdata and secrets, `rotom-proxy` web expectations, and a reserved non-authoritative NAS boundary. No document app or data is deployed. |
| JAR-46 Future domains | Tracked `filesync`, `customapps`, and `auth` contracts define identities, local appdata/secrets, reserved NAS authority, proxy expectations, and recovery-review triggers; no workloads are deployed. |
| JAR-8 Paperless | Paperless-ngx v3.2.1, PostgreSQL 18, and Valkey 9 run from `/srv/rotom/stacks/documents/paperless`; all mutable state is VM-local under Documents appdata and the web service uses `rotom-proxy` plus loopback NPM compatibility routing. `paperless.rotom.casa` is externally verified through the retained NPM route. |
| JAR-71 Remote Desktop Commander | Native outbound-only `desktop-commander.service` runs as locked, dedicated `desktopcmd` (`1001:1001`). Its `0700` home holds pinned agent `0.2.52`, the sole writable workspace, and sensitive pairing state; it has no sudo, Docker, service-group, NAS, proxy, DNS, or inbound-listener access. |
| Rotom v2 filesystem foundation | JAR-34 created `/srv/rotom` as an empty, root-administered v2 namespace. `stacks/` and `scripts/` are locally Git-versioned declarative content; `appdata/` contains service-owned empty domain roots; `secrets/` and `backup-staging/` are root-only. Existing `/home/<service>/docker` workloads remain authoritative compatibility paths until their owning Phase D migrations. |
| Reserved v2 NAS boundaries | JAR-55 added empty `Gameserver`, `Downloads`, and `Customapps` exports as `/mnt/nas-gameserver`, `/mnt/nas-downloads`, and `/mnt/nas-customapps`. They use the established nofail automount/NFSv3 contract and do not replace active `Game`, `Downloader`, or `Apps` compatibility storage. |
| Palworld | `palworld-server-jared` and `palworld-server-fran` intentionally stopped, not dead/restarting/OOM; `unless-stopped`; active worlds/saves reverified and preserved |
| qBittorrentVPN | Running; fail-closed design retained; `wg0` verified `10.2.0.2/32`; Downloader sentinel and NAS mounts verified |
| Guest NAS | Twelve fstab-backed systemd automounts plus read-only Media/Game bindfs compatibility views; post-reboot verification passed |
| Guest Restic | Canonical guest layer unchanged: `/mnt/nas-rotom-restic-backup/rotom-restic-backup`, `rotom-restic-backup*`, manual `backup-restic-to-nas` |
| PVE host-config Restic | NAS `PVE_Restic_Backup/.data`; mount `/mnt/nas-pve-restic-backup`; repo `/mnt/nas-pve-restic-backup/pve-restic-backup`; repo ID `8ca0218c645dc2968c2b21d67dbda3840794d8bc9c157c0515a85af30eb3d0e3`; worker/unit/timer `pve-restic-backup*`; staging `/var/backups/pve-restic-recovery`; timer enabled/active daily `04:00`; retention `7 daily / 4 weekly / 12 monthly` |
| PVE Restic current recovery point | Snapshot `35d4b0c2`, host `pve`, path `/var/backups/pve-restic-recovery`; pre-rename `0cf1c62a` and `3db28d41` deliberately forgotten; subsequent `restic check` found no errors |
| Whole-VM backup | NAS `Rotom_VM_Backup/.data`; PVE storage `nas-rotom-vm-backup`; mount `/mnt/pve/nas-rotom-vm-backup`; launcher `backup-rotom-vm-to-nas`; service/worker `rotom-vm-vzdump-manual*`; job `rotom-vm-daily` enabled at `05:00`, `repeat-missed=0`, retention `7 daily / 4 weekly / 6 monthly` |
| Current whole-VM recovery point | Exactly one VMID 100 backup after deliberate cleanup: `vzdump-qemu-100-2026_09_28-00_09_19.vma.zst`, `47,415,540,796` bytes; zstd integrity PASS; full VMA verification PASS; VM100 remained running; currently unprotected |
| PVE monitoring | CPU: `/usr/local/sbin/pve-cpu-temp-api`; `pve-cpu-temp-api.service`; endpoint `192.168.1.68:8788`; Homepage label `PVE CPU Temperature`. Backup schedules: `/usr/local/sbin/pve-backup-status-api`; `pve-backup-status-api.service`; LAN-only `192.168.1.68:8789`; Homepage `Backup Schedule` section orders Rotom VM, guest Rotom Restic, then PVE Restic cards |
| Final JAR-68 acceptance | Physical reboot proven; PVE core services active; VM100 autostart passed; final PVE and Rotom post-reboot checks passed; zero failed PVE and Rotom systemd units at acceptance |

## 3. Current Workload Ownership — Restored Through JAR-31

JAR-31 restored the production Compose/application layer into the Debian VM while retaining the Phase B service-account split. The Linux-side download identity is **`downloaders`** at UID:GID `901:5005`; current active Prowlarr/qBittorrent project discovery uses `/home/downloaders/docker`. Historical sections retain the old `downloads` name where they describe pre-migration state.

ChatGPT-controlled browser work is not part of the Rotom administration or documentation workflow. Available Sources replacements are handled manually by Jared when needed. The canonical RPD maintenance contract selects behavior by execution environment and uses `rpd path` to resolve the active checkout rather than making shared workflow instructions depend on a physical path. ChatGPT on Mac delivers changed replacement files plus the complete RPD ZIP and instructs Jared to copy the changed files into the checkout returned by `rpd path`, manually replace Available Sources, then publish with a relevant `rpd commit`, `rpd check`, and `rpd push`; ChatGPT itself performs no Git publication. Codex on Mac or Rotom resolves its local checkout with `rpd path`, edits that checkout directly when RPD maintenance is authorized, and completes the final state through the guarded `rpd` commit/check/push flow. The Git checkouts remain operational mirrors/references and do not replace the Available Sources membership boundary or the maintenance contract in document 00. Apple Passwords/iCloud Passwords remains the credential source of truth; never record credential values or authentication material in the project documentation. See document 06 for the standing administration/Codex workflow and document 00 for RPD maintenance governance.

| Account | Current responsibility |
|---|---|
| infra | Arcane, Cloudflare DDNS, Homepage, Glances, Nginx Proxy Manager |
| media | Jellyfin, Radarr, Sonarr |
| gameserver | Palworld servers for Fran and Jared at UID:GID `995:5001`; Gamarr retains UID 995 but uses Media storage GID 5000 for its Media-library bind; compatibility home `/home/game` retained |
| smarthome | Home Assistant and Homebridge |
| documents | Service-account skeleton reserved for future document workloads; no application deployed |
| downloader | Active Prowlarr/qBittorrentVPN domain; UID:GID `901:5005`; compatibility home and mount remain `/home/downloaders` / `/mnt/nas-downloaders` |
| web | Aloha Millworks active; Jared Wines project retained but intentionally undeployed |
| filesync | Service-account skeleton reserved for future synchronization workloads; built-in Linux `sync` remains untouched |
| customapps | Service-account skeleton reserved for future application workloads; compatibility home `/home/apps` retained |
| auth | Service-account skeleton reserved for future authentication workloads |
| desktopcmd | Dedicated non-human Remote Desktop Commander identity; only `/home/desktopcmd/workspace` is its intended writable work area; no service-storage role or NAS mount |
| jared | Interactive administrator with full sudo; Fran's pre-migration administrator identity is preserved but not yet recreated in the VM |

The pre-migration system granted Docker group GID `984` to all named operational accounts. JAR-30 deliberately did not recreate that root-equivalent privilege model. A fresh 2026-09-27 post-boot read again returned `docker:x:989:` with no members, confirming the current VM still uses `sudo docker` rather than broad Docker-group membership. Historical GID `984` membership must not be inferred current.

### Current service identities, storage, and application state

The ten storage-facing service accounts retain their JAR-29 numeric identity contract. Current workload restoration did not require broadening ordinary NFS permissions.

| Account | UID : primary GID | Current NAS mount | Current state |
|---|---:|---|---|
| media | 127 : 5000 | `/mnt/nas-media` | Active Jellyfin/Radarr/Sonarr owner; NFSv3 on demand; ordinary storage boundary retained |
| gameserver | 995 : 5001 | `/mnt/nas-game` | Active Jared/Fran Palworld identity; the retained Game library is JAR-52 rollback material, while Gamarr uses UID 995 with Media GID 5000 against `/mnt/nas-media/library/games` |
| infra | 997 : 5002 | `/mnt/nas-infra` | Active core infrastructure owner; Arcane restored with preserved named volume |
| smarthome | 126 : 5003 | `/mnt/nas-smarthome` | Active Home Assistant/Homebridge owner; authoritative application state is local under `/srv/rotom/appdata/smarthome` |
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
| `/mnt/nas-rotom-restic-backup` | NFSv3 `Rotom_Restic_Backup/.data`; mounted share root `988:988` mode `0700`; canonical repository `/mnt/nas-rotom-restic-backup/rotom-restic-backup`; current verified snapshot `f666d63c`; `rotom-restic-backup.timer` enabled/active |
| `/mnt/nas-pve-restic-backup` *(PVE host only)* | NFSv3 `PVE_Restic_Backup/.data`; systemd automount from PVE `/etc/fstab`; canonical repository `/mnt/nas-pve-restic-backup/pve-restic-backup` (repo ID `8ca0218c645dc2968c2b21d67dbda3840794d8bc9c157c0515a85af30eb3d0e3`); staging `/var/backups/pve-restic-recovery`; current snapshot `35d4b0c2`; `pve-restic-backup.timer` enabled/active for `04:00` |
| `/mnt/nas-shared-drive` | NFSv3 Shared Drive; contains JAR-25 raw NVMe rollback image; outside host Restic because `/mnt` is excluded |
| `/home/infra/docker` | Active core infrastructure Compose trees: Arcane, DDNS, Homepage/Glances, NPM |
| `/home/media/docker` | Active Jellyfin/Radarr/Sonarr Compose/config trees |
| `/home/downloaders/docker` | Active Prowlarr/qBittorrentVPN Compose/config trees; Arcane mounts this root read-only for discovery |
| `/home/smarthome/docker` | Active Home Assistant/Homebridge Compose/config trees |
| `/home/game/docker` | Both Palworld projects present with saves preserved; both Palworld containers intentionally stopped; Gamarr active |
| `/home/web/docker` | Aloha active; Jared Wines project retained but intentionally undeployed |
| `/var/lib/docker` | Current Docker runtime root on VM-local ext4; not NFS-backed |
| `/var/lib/docker/volumes` | Current Docker named-volume store; includes restored `arcane_arcane-data` |
| `/var/backups/system-info` | Active backup staging/recovery inventory path; all three required SQLite sources remain part of the normal backup path; current verified snapshot is `f666d63c` |

All twelve canonical **guest** NFS mountpoints remain guest fstab/systemd automounts. The PVE-only `/mnt/nas-pve-restic-backup` automount is an additional host backup path and does not change the guest twelve-mount count. The 2026-09-27 post-boot audit freshly verified all twelve generated `.automount` units active and the current NFS sources, including `Downloader/.data` and `Rotom_Restic_Backup/.data`. JAR-31 additionally restored the two bindfs compatibility views and the targeted NAS/Docker recovery helper; fresh Media/Game traversal checks again passed. The former `/mnt/nas-rotom-backup` export is no longer part of current Rotom configuration: it is unmounted and the old NAS `Rotom_Home_Server_Backup` share is retained only as rollback data. The mounted `/mnt/nas-rotom-restic-backup` NFS root freshly verified as numeric `988:988` mode `0700`. The audit did not isolate the underlying local directory while fully unmounted, so its local-directory mode remains non-authoritative unless separately checked.

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

The current recovery design has three independent logical layers on the same UniFi UNAS 2 appliance: guest Restic, PVE host-configuration Restic, and whole-VM VZDump.

- **Guest Restic:** unchanged canonical guest repository `/mnt/nas-rotom-restic-backup/rotom-restic-backup`, manual command `/usr/local/bin/backup-restic-to-nas`, worker `rotom-restic-backup`, timer around `03:00`, retention `7 daily / 4 weekly / 12 monthly`.
- **PVE host-config Restic:** `PVE_Restic_Backup/.data` -> `/mnt/nas-pve-restic-backup` -> `pve-restic-backup`; repository ID `8ca0218c645dc2968c2b21d67dbda3840794d8bc9c157c0515a85af30eb3d0e3`. Worker `/usr/local/sbin/pve-restic-backup`; service/timer `pve-restic-backup.service` / `.timer`; staging `/var/backups/pve-restic-recovery`; manual launcher `/usr/local/bin/backup-restic-to-nas`; password path `/etc/restic/nas-password`. Snapshot `35d4b0c2` is the surviving canonical point after deliberate removal of the two pre-rename snapshots; `restic check` passed.
- **Whole-VM VZDump:** `Rotom_VM_Backup/.data`, storage `nas-rotom-vm-backup`, mount `/mnt/pve/nas-rotom-vm-backup`; manual launcher `/usr/local/bin/backup-rotom-vm-to-nas`; oneshot `rotom-vm-vzdump-manual.service`; worker `/usr/local/sbin/rotom-vm-vzdump-manual`; lock `/run/lock/rotom-vm-vzdump-manual.lock`. Automatic job `rotom-vm-daily` is enabled at `05:00`, snapshot + zstd, `repeat-missed=0`, retention `7 daily / 4 weekly / 6 monthly`. Six older VM100 backups were deliberately removed and replaced by one fully verified archive `vzdump-qemu-100-2026_09_28-00_09_19.vma.zst` (`47,415,540,796` bytes). It is currently unprotected and can eventually be pruned normally.
- **Maintenance:** PVE CPU bridge uses `pve-cpu-temp-api`; read-only backup schedule bridge uses `pve-backup-status-api` on `192.168.1.68:8789`, serving the PVE Restic and native Rotom VM Homepage cards. The NIC WOL/offload configuration remains as documented. The historical local `backup-proxmox-to-nas.pre-systemd.*` launcher was deliberately removed after final acceptance.
- **Operational evidence:** physical PVE reboot, VM100 autostart, post-reboot PVE/Rotom acceptance, canonical mount/source checks, qBittorrent VPN state, and zero failed units all passed.

## 7. Current Startup and Dependency Summary

1. The Debian VM still boots independently of NAS availability through the fstab/systemd automount design.
2. Docker and containerd are enabled and returned active after the JAR-31 acceptance reboot. Docker runtime state remains local at `/var/lib/docker`.
3. The adapted `rotom-nas-docker-recovery.service` is enabled with retry-on-failure behavior. On the JAR-31 reboot its first invocation reached `/mnt/nas-downloaders` but failed when `/mnt/nas-media` was not ready; systemd retried about 30 seconds later, then the helper mounted/verified Downloader, Media, and Game storage plus both read-only bindfs views and automatically recovered qBittorrentVPN, Radarr, Sonarr, Gamarr, and Jellyfin. Final unit state was `active (exited)`, `Result=success`, `ExecMainStatus=0`.
4. The two compatibility-view services are active: `/mnt/nas-downloads-media-ro` and `/mnt/nas-downloads-game-ro` are read-only bindfs views sourced from current `/mnt/nas-downloaders`.
5. The earlier reboot restored the then-intended 16-running set. Final JAR-68 acceptance later recorded 16 total container objects / 14 intended running because both Palworld containers were deliberately stopped; qBittorrent `wg0` and core checks passed and zero systemd units were failed.
6. The Proxmox host remains independent of Rotom application NFS and application Docker; no application share is mounted there.

## 8. Current Architecture-Level Findings and Limitations

- The Phase B application layer is restored. The final running set is 16 containers; Jared Wines remains intentionally undeployed.
- UID/GID `901:5005` is named `downloaders`, with `/home/downloaders` and `/mnt/nas-downloaders`. **Current live evidence identifies the backing NAS export as `Downloader/.data`, not the earlier JAR-29/JAR-30 `Downloads/.data` claim.** Current fstab, generated mount unit, active NFS source, and `showmount` output agree.
- `/mnt/nas-downloaders` root metadata is `988:5005` mode `2770`; `.rotom-qbt-nas-ready` and `torrents/{games,incomplete,movies,shows}` were observed. Historical `Downloads/.data` references remain dated historical evidence.
- Docker's root `/var/lib/docker` remains VM-local ext4. The application networks now occupy the verified bridge map recorded in documents 02/03; Homepage is `172.24.0.0/16`, Jared Palworld was deliberately moved to `172.27.0.0/16`, and Arcane is `172.28.0.0/16`.
- Jared Palworld was stopped cleanly for the network relocation, a stopped-state archive was made, five active-save paths matched the pre-move gate, and the server returned healthy with the preserved image and zero restarts.
- A fresh 2026-09-27 post-boot read verified the package-created Docker group remains GID `989` with no members; the historical pre-migration GID `984` broad-membership model remains retired.
- Backup controls are restored: guest Restic is enabled/active under `rotom-restic-backup`, Proxmox host-configuration Restic is enabled/active at `04:00`, and native PVE VZDump remains enabled at `05:00`. Restic repositories must never be manually reorganized.
- Homepage's local backend intentionally enforces `HOMEPAGE_ALLOWED_HOSTS=rotom.casa`; `curl http://127.0.0.1:3001/` returning 400 is expected host validation, while the same request with `Host: rotom.casa` and the HTTPS apex both returned 200.
- Mac key-only SSH remains verified for Rotom and Proxmox; password-authentication policy is unchanged and no key contents/passphrases are recorded.

### 2026-09-27 live post-boot verification refresh

A read-only audit after the 02:20 PDT reboot refreshed the current Phase B evidence: zero failed units; Docker `29.8.1` / Compose `v5.5.1`; 16/16 containers running with zero restart counts; current bridge subnets `172.18.0.0/16` through `172.28.0.0/16`; qBittorrent `wg0` present; all ten service accounts and `/home/<service>/nas-<service>` symlinks correct; Docker group `989` empty; all twelve NFS automounts active; both bindfs compatibility traversals passing; mounted Restic root `988:988` mode `0700`; later that day the renamed Restic timer was recommissioned enabled/active; weekly-update cron absent; core HTTP probes successful; and the Proxmox temperature API reachable. Arcane automation settings were also freshly reread and are current as documented in `06-Maintenance-and-Automation.md`.

## 9. Outstanding Decisions and Needs Verification

JAR-31 compatibility/application restoration and the JAR-33 virtualization baseline are complete. The remaining items are either intentional commissioning decisions or facts that still need current evidence:

- **Intentional / not yet commissioned:** Fran is not recreated and her historical backup sudoers rule remains absent; the reboot-capable weekly updater script is present but its cron schedule is absent; broader JAR-47/off-site failure-domain recovery work remains unfinished even though guest Restic, Proxmox host-config Restic, and Proxmox whole-VM schedules are now active; Rotom VM iGPU passthrough remains uncommissioned.
- **Needs Verification — NAS metadata persistence:** service-share root UID/GID/mode persistence across a future NAS/UniFi Drive restart/update has not been observed. Resolve after such an event with read-only `stat`/`findmnt` checks from Rotom plus NAS export inspection; do not change ownership merely to make the result match documentation.
- **Needs Verification — first post-rename Restic scheduled run:** `rotom-restic-backup.timer` is enabled/active and has a populated next trigger, but the first timer-triggered run under the renamed unit has not yet been observed in this evidence set. The same implementation has been verified through the manual launcher.
- **Needs Verification — first final-name PVE host-config Restic scheduled run:** `pve-restic-backup.timer` is enabled/active for `04:00`; manual/service execution and repository integrity are verified, but a future unattended timer-triggered run under the final name may still be observed as operational evidence.
- **Needs Verification — first final-name native PVE scheduled run:** `rotom-vm-daily` is enabled for `05:00`; the disconnect-safe manual path and fresh archive are fully verified, but a future unattended scheduler-triggered run under the final name may still be observed.
- **Retention decision:** the canonical whole-VM storage currently has exactly one fresh verified VMID 100 archive, `vzdump-qemu-100-2026_09_28-00_09_19.vma.zst`. It is unprotected and therefore may be pruned normally under `rotom-vm-daily` retention; protect it only if a permanent JAR-68 baseline is desired.
- **Needs Verification — end-to-end Wake-on-LAN trigger:** Proxmox `nic0` WOL capability, persistent `wol g` configuration, PCI wake enablement, and post-power-cycle retention are verified, but the supplied transcript does not capture a magic-packet sender invocation or otherwise prove WOL caused the NUC to power on.
- **Needs Verification — prior Proxmox resets:** the ConBee II remains intentionally disconnected and the RPD does not isolate one causal explanation for the earlier resets. Read-only logs can narrow evidence (`journalctl`, Proxmox task/system logs), but a single cause should not be claimed without new evidence.
- **Recovery/operations follow-up:** a full Palworld disaster-recovery rehearsal, alert-delivery test, certificate-renewal observation, and final public-exposure review remain unperformed or incompletely evidenced. These are tests/decisions, not contradictory current-state facts.

## 10. Rotom Project Documentation Model

**Rotom Project Documentation** is the proper name for **every file intentionally kept in this project's Available Sources**. Available Sources defines the complete maintained documentation set and is the membership boundary regardless of filename, extension, file format, document type, or purpose. A file does not need to be Markdown, a guide, a standard, or a numbered reference file to belong to the set.

The numbered files **00–08** form the **Core Numbered Reference Set**. A complete Available Sources inventory on 2026-09-27 confirms they are also the entire current Rotom Project Documentation set: nine files total, with no additional current non-numbered source. They remain the primary current-state references for change history, server inventory, Docker services, networking, storage, backup/recovery, maintenance/automation, users/permissions, and the directory tree. `Rotom-Architecture-and-Deployment-Standard.md`, `Rotom-Home-Server-Guide-Maintenance.md`, and `Rotom-Home-Server-Manual-Maintenance.md` were removed from Available Sources by user decision on 2026-09-20 and therefore are no longer current Rotom Project Documentation. The same membership rule applies automatically to any file intentionally added to or removed from Available Sources in the future.

Within the Rotom project, **RPD** is the accepted acronym for **Rotom Project Documentation**. “Project docs,” “rotom project docs,” “project documentation,” “project documents,” and “project reference files” are also informal aliases unless a narrower subset is specified. Capitalization does not change the meaning. Raw command output, audit attachments, chat exports, screenshots, generated packages, local working files, and other material that exists **outside** Available Sources remains supporting or temporary material rather than Rotom Project Documentation. If such a file is intentionally placed in Available Sources, it becomes part of Rotom Project Documentation by membership.

Consult the relevant available documents before giving Rotom-specific advice or documenting changes. Respect their evidence dates and distinguish current recorded configuration, historical findings, proposals, and unverified details. If a required document is unavailable, identify the gap rather than assuming its contents.

RPD maintenance is governed exclusively by [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md). The sole canonical update command is **`Update the RPD`**; its scope, evidence rules, consistency-check requirements, delivery behavior, and documentation-only authorization boundary are defined there. This file defines RPD membership and the source map rather than duplicating the maintenance procedure.

The September 16 update also incorporates “Change Homepage URL” plus the user's confirmation that Ghostty was removed. Homepage routing is verified by the supplied chat output. Earlier browser-workflow decisions are historical and are superseded by the current no-browser-automation policy.

For project-wide Rotom command execution guidance, large or multi-step command blocks should be isolated in a child Bash process by default so errors and shell-control statements cannot terminate the interactive SSH session. Small single-purpose commands may run directly. The detailed convention is maintained in `06-Maintenance-and-Automation.md`.

## 11. Current Source Map

Available Sources determines the complete Rotom Project Documentation membership. The map below records the current Project-backed Available Sources set for navigation, but membership does not depend on a filename appearing in this table. When the Available Sources set changes, update this map as part of the next substantive documentation update. Follow the maintenance guide in [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md) and update its dated history in the same task.

### Core Numbered Reference Set

| Area | Project source |
|---|---|
| Documentation change history and sole RPD maintenance contract | [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md) |
| Documentation definition, index, and server overview | [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md) |
| Docker services, Compose paths, images, container/runtime deployment | [02-Docker-Services.md](02-Docker-Services.md) |
| Host network, DNS, ports, Docker networks, NPM, Cloudflare | [03-Network-and-Domains.md](03-Network-and-Domains.md) |
| NAS exports, mounts, automounts, storage layout and contracts | [04-NAS-and-Storage.md](04-NAS-and-Storage.md) |
| Restic, backup scope, retention, verification, restore boundaries | [05-Backup-and-Restore.md](05-Backup-and-Restore.md) |
| Timers, updates, monitoring, automation, and administration workflow | [06-Maintenance-and-Automation.md](06-Maintenance-and-Automation.md) |
| Accounts, UID/GID, groups, sudo/Docker privilege, ACL/access boundaries | [07-Users-and-Permissions.md](07-Users-and-Permissions.md) |
| Observed filesystem topology and important paths | [08-Rotom-Directory-Tree.txt](08-Rotom-Directory-Tree.txt) |

### Current membership

The current Project-backed Available Sources set is exactly the nine-file Core Numbered Reference Set (`00`–`08`) listed above. There are no additional current non-numbered Rotom Project Documentation files. Future files intentionally added to Available Sources become Rotom Project Documentation automatically, regardless of file type or extension; files removed from Available Sources cease to be current members without erasing their historical change-log references.

The current Git checkouts are verified at `/Users/jared/Documents/ChatGPT/rotom-project-documentation` on Mac and `/home/infra/documentation/rotom-project-documentation` on Rotom, both using origin `git@github.com:jaredwines/rotom-project-documentation.git`, branch `main`, and upstream `origin/main`. Historical verification at the former Mac `/Users/jared/Documents/ChatGPT/Rotom-Home-Server` and Rotom `/home/infra/documentation/rpd` paths remains dated evidence only. Repository support metadata now includes `.gitignore`, shared root `AGENTS.md`, and `tooling/rpd`; none is part of Rotom Project Documentation unless intentionally added to Available Sources. The identical universal helper is installed at `/usr/local/bin/rpd` on both systems, is versioned at `tooling/rpd`, and resolves the expected current checkout from the active Git working tree or from `~/.config/rpd/repository`; `rpd path` exposes the result. The per-machine fallback currently contains `/Users/jared/Documents/ChatGPT/rotom-project-documentation` on Mac and `/home/infra/documentation/rotom-project-documentation` on Rotom. Mac global `~/.codex/AGENTS.md` contains only the command-line editor preference. Rotom `/home/jared/.codex/AGENTS.md` remains the Jared-global machine entry point, contains Rotom live-state/safety guidance, and points to `rpd path`, the repository `AGENTS.md`, and document 00. Fresh read-only Codex sessions on both systems verified the intended discovery hierarchy. Under the adopted execution policy, Codex resolves its checkout with `rpd path` and uses guarded `rpd commit`, `rpd check`, and `rpd push` only when RPD maintenance is authorized; ChatGPT on Mac remains a replacement-file/ZIP delivery path and does not publish Git changes itself. The older 2026-09-19 statement that top-level `/home/infra/documentation/AGENTS.md` and `README.md` were present remains historical; those legacy top-level files were not re-inspected during this integration task, so their present state remains unverified.

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
| Scheduled/routine operations, automation, monitoring, and administration workflow | `06-Maintenance-and-Automation.md` |
| Linux accounts, UID/GID, groups, sudo/Docker privilege, ACL/access model | `07-Users-and-Permissions.md` |
| Observed filesystem topology and important paths | `08-Rotom-Directory-Tree.txt` |
