# 07 - Users and Permissions

**Documentation set:** legacy documentation set  
**Document role:** Canonical source for Linux accounts, UID/GID identities, groups, privileges, ACLs, and access policy  
**Hosts:** PVE hypervisor `pve` and Debian VM `rotom`  
**Baseline verified:** historical workload identity evidence through 2026-09-25; Phase B guest admin/backup-access state verified through 2026-09-27  
**Documentation updated:** 2026-10-15 — Improve navigation and safe-use guidance
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`, `06-Maintenance-and-Automation.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1. Purpose and Scope** and **2. Current Identity and Access Model** for supported accounts, UID/GID contracts, groups, ACLs, and privilege boundaries.
- Treat **Preserved Pre-Migration** and **Historical** sections as evidence, not as instructions to recreate retired users, groups, homes, memberships, or permissions.
- Use **Reconstruction-Critical Information** only after reconciling it with section 2 and read-only NSS, ownership, ACL, sudo, and Docker-group evidence.
- Review **Outstanding / Needs Verification** before granting access or changing an identity boundary.
- This file owns identities and access policy; storage mount semantics, Docker deployment, and operational workflow remain canonical in documents 04, 02, and 06.

## 1. Purpose and Scope

### Current identity and Smart Home ownership audit — 2026-10-14

Jared explicitly authorized removal of the verified-unused `/home/game-servers`, `/home/hosted-websites`, `/home/libraries`, `/home/platform`, and `/home/smart-home` trees. The five accounts remain password-locked with their existing UID:GID contracts and configured NSS home fields; those fields now point to absent directories. No account-home migration was performed. Active service Compose, state, and protected-input roots remain under `/srv/rotom`; pre-removal checks found no nested mounts, open users, or Docker bind mounts, and the system plus listed containers remained healthy afterward.

Live NSS data now confirms `platform 997:5002`, `libraries 127:5000`, `hosted-websites 902:5004`, `game-servers 995:5001`, `smart-home 126:5003`, `jared 1000:1000`, `desktopcmd 1001:1001`, and human administrator `fran 1002:1002`. Rotom Fran belongs to `sudo`, `users`, and Docker GID `989`; she is the Docker group's sole listed member. Service identities retain only their primary service groups. Privileged read-only `passwd -S` verification on 2026-10-14 found `platform`, `libraries`, `hosted-websites`, `game-servers`, `smart-home`, and `desktopcmd` each password-locked (`L`); no unlock or password-setting command was run.

Fran and the selected service identities use `/usr/bin/zsh` with Oh My Zsh. `desktopcmd` changed from `/bin/bash` to `/usr/bin/zsh` but remains a non-human account without sudo, Docker, SSH keys, service-group, or NAS-specific access. Existing non-Oh-My-Zsh service `.zshrc` files were backed up before replacement. Fran's prompt and the selected account prompts were exercised successfully; direct local inspection reconfirmed the account shells and group boundaries.

A read-only 2026-10-14 follow-up executed `docker ps` directly as Fran and listed the 20 running containers, confirming the documented Docker-group access works in practice. `desktop-commander.service` was enabled and active/running after the `desktopcmd` shell change. These checks do not alter Fran's root-equivalent Docker boundary or `desktopcmd`'s intentionally restricted account policy.

The existing `desktopcmd` identity was then migrated live from `/home/desktopcmd` to configured home `/opt/desktop-commander` without changing UID:GID `1001:1001`, the locked password state, Zsh/Oh My Zsh configuration, or its sole primary-group membership. The new home preserves the private base ACL and named Jared read/traverse/default ACL, so its displayed mode is `0750`; no base group or other access was granted. Jared subsequently authorized removal of the verified-unused old-home contents; the legacy directory itself is now absent. No `desktopcmd` sudoers entry or authorized SSH key exists.

The Smart Home stack directories are root-owned, but both live Compose files are owned by `smart-home:smart-home`. On 2026-10-14 Jared confirmed the cross-domain Game Servers grants were unintended, so only the named `game-servers:rw-` Home Assistant and `game-servers:rwx` Homebridge ACL entries were removed. Home Assistant remains `smart-home:smart-home` mode `0660`; Homebridge remains `smart-home:smart-home` mode `0670`; Jared retains read-only/read-traverse named access. Negative access tests confirmed `game-servers` cannot write either file, positive tests confirmed `smart-home` can read/write both, and Compose validation passed.

`/srv/rotom/backup-staging` is absent, so the earlier ACL observations for that root and its descendants are historical. The current `/srv/rotom` and `/home` roots retain the documented named Jared read/traverse/default ACL pattern; the former `/home/platform` service-home tree was retired on 2026-10-14.

### Current Paperless Libraries identity — 2026-10-13

Paperless application processes use the current `libraries` identity, UID:GID `127:5000`; the retired `documents` account was not recreated. Its newly created authoritative media subtree, `/mnt/nas-libraries/paperless`, is `127:5000`, mode `2770`. VM-local mutable state under `/srv/rotom/appdata/libraries/paperless` uses the required service/image ownership for each component, while Paperless web data, exports, and consume paths are reconciled to `127:5000`. Protected environment inputs remain root-owned under `/srv/rotom/secrets/libraries/paperless`; values are intentionally not recorded. Historical Documents ownership records remain recovery evidence only.

This is the canonical detailed owner of Rotom account identity and permission facts. Other documents may repeat a UID/GID or access consequence only as context and should refer back here for the complete account model.

### Evidence provenance

**Documentation updated:** 2026-10-14 — Add Fran as a Rotom and PVE administrator  

**Baseline method:** read-only inspection of the live host configuration, Docker metadata, systemd unit files, sudo policy, SSH metadata, filesystem modes, ACLs, NFS mount metadata, and non-mutating access tests. No passwords, password hashes, key contents, tokens, or private-key material were read or recorded.  
**Update scope:** JAR-29 restored the historical ten storage-facing identities and JAR-30 established the package-created Docker privilege boundary. Later migrations retired or renamed several of those identities. The current table in section 2 is authoritative; earlier full-account and NAS-shortcut tables are historical reconstruction evidence. Fran is now recreated independently on Rotom (`1002:1002`) and PVE (`1001:1001`) as a human administrator. Administrative Mac key-only access remains verified only for Jared; Fran SSH is intentionally deferred and no key contents are recorded.

## 2. Current Identity and Access Model

### Historical JAR-94 Platform identity and retired accounts — 2026-10-13

At this checkpoint `platform` owned UID:GID `997:5002` and `/home/platform`, replacing `infra` while preserving the numeric ownership contract for recovered shared-service state. The active legacy documentation checkout had moved with the service home to `/home/platform/documentation/rotom-runbook`; JAR-98 and the later service-home cleanup supersede both paths. `infra`, `documents`, `downloader`, and `filesync` accounts/groups were absent.

Their requested service homes and active local application trees were retired, together with the four local NFS contracts. The remaining empty `/srv/rotom/appdata/documents` and `/srv/rotom/secrets/documents` directories were removed on 2026-10-13 after no active reference was found. `/home/downloaders` later reappeared as an empty root-owned `0755` directory containing only empty `docker/`; it had no associated account and Arcane's active Compose file retained it as a read-only compatibility bind. Both the bind and home were removed on 2026-10-13; this did not restore or alter a Downloader service. Auth remains retired as recorded below.

**Historical identity state — 2026-10-13, superseded for physical home and runtime status:** The active Libraries service account was `libraries` (`127:5000`), preserving the former Library numeric identity. Its NSS home field was `/home/libraries`, with active local paths under `/srv/rotom/{stacks,appdata,secrets}/libraries`; `library` was absent from NSS. The active Palworld service account was `game-servers` (`995:5001`), preserving the former Game numeric identity; `game` and `gameserver` were absent from NSS. Its NSS home field was `/home/game-servers`, and active local paths were under `/srv/rotom/{stacks,appdata,secrets}/game-servers`. The active website account was `hosted-websites` (`902:5004`), with NSS home field `/home/hosted-websites` and local website paths under `/srv/rotom/{stacks,appdata}/hosted-websites`; `web` was absent from NSS. The active Smart Home account was `smart-home` (`126:5003`), with NSS home field `/home/smart-home`, Compose roots under `/srv/rotom/stacks/smart-home`, and mutable state under `/srv/rotom/appdata/smart-home`; `smarthome` was absent from NSS. The five physical service-home directories were removed on 2026-10-14 while the NSS fields remained. The former Documents primary GID `5004` was reassigned only after that retired account/group was absent. `downloader` is absent; `901:5005` is historical identity evidence only. No Docker-group change was performed.

JAR-94 above supersedes the `infra`, `documents`, `downloader`, and `filesync` portions of this dated account inventory; the paragraph is retained as historical evidence.

**Current Customapps retirement — 2026-10-12:** The unused `customapps` account/group (`904:5008`) was removed. Its configured `/home/apps` path was already absent; the actual default-only `/home/customapps` tree and dangling NAS shortcut were removed. The empty v2 Customapps stack, appdata, and secrets directories and its Rotom guest NFS mount contract were also retired. The NAS-side export was empty at final guest inspection but remains outside Rotom until deleted through UniFi management.

**Current Auth retirement — 2026-09-29:** The unused `auth` account/group (`905:5009`), default-profile `/home/auth`, `nas-auth` shortcut, and Auth NAS boundary were retired after no process, workload, Docker mount, or operational configuration consumer was found. Historical snapshots may contain this identity; do not recreate it by default.

Guest service-account identities remain unchanged. Docker group is package-created GID `989`; Fran is now its sole member and therefore has root-equivalent Docker control in addition to full sudo. Jared and all service identities remain outside Docker, so their established `sudo docker` or no-Docker boundaries are unchanged.

| Layer/account | Verified current state |
|---|---|
| PVE host | `jared` (`1000:1000`) and `fran` (`1001:1001`) are human administrators with `sudo`, `users`, `systemd-journal`, and enabled matching PAM identities holding propagated `Administrator` access at `/`. Canonical Mac aliases and `~/.ssh/id_ed25519_pve` still use Jared; Fran SSH is deferred. Root remains emergency recovery. |
| Debian VM `jared` | UID/GID `1000:1000`; supplementary `sudo`; key-only Mac login via `~/.ssh/id_ed25519_rotom` |
| Debian VM `fran` | UID/GID `1002:1002`; supplementary `sudo`, `users`, and root-equivalent Docker GID `989`; `/home/fran` mode `0700`; `/usr/bin/zsh` with Oh My Zsh; SSH, legacy documentation/GitHub/Codex entry point, and named documentation ACL not established |
| libraries | `127:5000`; NSS home field `/home/libraries`, physical directory absent; Jellyfin/Radarr/Sonarr/Prowlarr/RomM/qBittorrentVPN/Paperless. Paperless media is `/mnt/nas-libraries/paperless`. |
| game-servers | `995:5001`; NSS home field `/home/game-servers`, physical directory absent; Palworld Jared and Fran were running and healthy on 2026-10-15, but the reason and intended persistence remain unverified; JDownloader, Igir, qBittorrent importer, and browser-upload bridge were retired on 2026-10-05; Gamarr retired |
| platform | `997:5002`; NSS home field `/home/platform`, physical directory absent; Arcane/DDNS/Homepage/Glances/NPM/Rotom Docs |
| smart-home | `126:5003`; NSS home field `/home/smart-home`, physical directory absent; Home Assistant/Homebridge |
| documents | **Retired**; historical `900:5004` / `/home/documents` record only; numeric GID `5004` is now Hosted Websites |
| downloader | **Retired**; account, former home, and Arcane compatibility bind are absent |
| hosted-websites | `902:5004`; NSS home field `/home/hosted-websites`, physical directory absent; Aloha active; Jared Wines undeployed |
| filesync | **Retired**; historical `903:5007` / Syncthing record only |
| desktopcmd | `1001:1001`, locked non-human account with configured home `/opt/desktop-commander` (private base ACL; displayed mode `0750` due Jared read/traverse ACL); `/usr/bin/zsh` with Oh My Zsh; Remote Desktop Commander only; no sudo, SSH authorized keys, Docker/socket, service-group, or NAS-specific access. `/home/desktopcmd` and its former contents are absent. |
| Docker group | GID `989`, sole listed member `fran` |

### Current `/srv/rotom` Jared access — 2026-10-10

Root applied the named POSIX ACL `user:jared:r-X` recursively to the existing `/srv/rotom` tree, including `stacks/`, `scripts/`, `appdata/`, `secrets/`, and `backup-staging/`. Every existing directory carries the matching default named-user ACL, so ordinary newly created descendants inherit Jared read/traverse access. The access ACL and default ACL mask are `r-x`; ownership and the base owner/group ACL entries were not changed. POSIX presents that ACL mask through the conventional group-class mode display, but it does not grant the base group new access. Jared can read regular files and list/traverse directories but does not receive write access through this ACL, including at the `secrets/` and `backup-staging/` roots.

An application can subsequently set a restrictive mode or ACL that constrains or replaces inherited access; investigate such a case from the live ACL rather than assuming the default remains effective. One existing backup-staging descendant is owned by Jared and retains its pre-existing owner write permission; that condition was observed but not changed by this access-policy update.

### Current `/home` Jared access — 2026-10-12

Root applied the same named POSIX ACL `user:jared:r-X` recursively to local service-home content and placed the matching default named-user ACL on `/home` and the local service directories. Jared can read regular files and list/traverse all local home trees, including service homes and the otherwise locked `desktopcmd` home; service-home roots remain non-writable to Jared. Jared's own home remains readable and writable through its pre-existing owner permissions, so its redundant named ACL was removed to preserve owner-only SSH private-key protection.

The `nas-*` entries within service homes are shortcut symlinks to NFS storage. Their targets rejected the local ACL operation as unsupported and remain governed by their existing NAS permission boundaries; no NAS ownership, export, or data change was made. Ownership and base owner/group ACL entries throughout local `/home` remain unchanged. As with `/srv/rotom`, ACL mask `r-x` is reflected in the conventional group-class mode display without granting the base group new access. Future ordinary local descendants inherit the default ACL unless a creating application subsequently restricts or replaces it.

### JAR-87 through JAR-90 ROM-ingestion boundary — retired 2026-10-05

The JDownloader, Igir, qBittorrent importer, and browser-upload service identities and write boundaries are retired. Their dedicated configuration, state, scripts, and credentials are absent. qBittorrent's current Libraries sentinel and RomM's read-only Libraries access remain in place.

### Current PVE backup privilege boundary

PVE Restic Backup is root-operated. `/usr/local/sbin/pve-restic-backup` is the privileged worker; root-owned `/usr/local/bin/backup-restic-to-nas` re-executes through `sudo` when a non-root user invokes it, so Jared or Fran can start the existing root workflow through normal sudo authentication without direct access to the worker or its protected credential. `/etc/restic/restic-repository-password` remains protected and its contents are never documented. The repository is `/mnt/nas-pve-restic-backup/pve-restic-backup` and staging is `/var/backups/pve-restic-recovery`. Rotom VM Backup uses the root-owned `/usr/local/bin/backup-rotom-vm-to-nas`, which re-executes through `sudo` for non-root invocation, plus `rotom-vm-vzdump-manual.service` and `/usr/local/sbin/rotom-vm-vzdump-manual`; VZDump remains the technical mechanism. Jared and Fran are PVE members of `systemd-journal`, allowing direct read-only service-log access such as `journalctl -fu rotom-vm-vzdump-manual.service`; this group does not itself grant permission to alter services.

### Current administrative public-key access

The Mac SSH config maps `pve` / `pve.rotom.casa` to `192.168.1.68` as `jared` using `~/.ssh/id_ed25519_pve`, `IdentitiesOnly yes`, `AddKeysToAgent yes`, and `UseKeychain yes`. Supplied output verified PVE Jared's `sudo` membership, enabled `jared@pam` identity, propagated `Administrator` ACL at `/`, and successful `sudo pveversion`; Jared confirmed both aliases now use this account. The matching public key was removed from PVE root. Fran's enabled `fran@pam` identity now has the same propagated `Administrator` ACL, but `/home/fran/.ssh` was verified absent and her SSH-key authentication is deferred. The old `proxmox` / `proxmox.rotom.casa` SSH aliases and old `~/.ssh/id_ed25519_proxmox{,.pub}` filenames remain absent. PVE `/etc/hosts` likewise contains only the canonical `pve.rotom.casa pve` names for `.68`. Password-auth policy was not changed.

### UniFi appliance administration — 2026-09-29

The UniFi Gateway and UNAS are appliance-managed systems, not general-purpose Linux hosts. Routine management uses the appropriate UniFi UI account and administrator role. If SSH is required for targeted troubleshooting, use the appliance's supported root-console access. Do not create unsupported local Linux `jared` accounts or rely on custom local account state surviving appliance updates.

The Rotom VM continues to use `rotom` / `rotom.casa`, user `jared`, and `~/.ssh/id_ed25519_rotom`. Private-key contents, passwords, and other authentication secrets are never recorded.

### Current legacy documentation / GitHub / Codex access — 2026-09-28

Historical `getfacl -p /home/infra` output showed the former Infra-home traversal model before the Platform rename. The Platform service home remains under `/home/platform`; Jared's current independent checkout is described below. Current root ACL behavior is summarized in section 1. Fran is now recreated, but has no legacy documentation checkout, GitHub identity, or Codex authentication yet.

Jared's current legacy documentation Git checkout is `/home/jared/documentation/rotom-runbook`; the former `/home/infra/documentation/retired-helper` path is historical. On 2026-10-14 it was cloned cleanly from the shared GitHub `main` branch as `jared:jared` mode `0700`, and Jared's private guarded-`retired-helper` fallback was changed to it. The former clean `/home/platform/documentation/rotom-runbook` checkout was removed at Jared's explicit authorization after final verification; it is neither a rollback path nor a Rotom Docs input. Fran has private non-credential legacy documentation/Codex setup only; no credentials were copied or changed, and her GitHub/Codex authentication and checkout were explicitly waived for JAR-98.

Jared's general GitHub SSH identity is `~/.ssh/d_ed25519_github`. The private key is kept under Jared's account and must not be copied to `infra` or another service account; key contents and passphrases are never documented. Supplied output verified `ssh -T git@github.com` authenticated as GitHub account `jaredwines`, and the legacy documentation checkout uses the SSH remote `git@github.com:jaredwines/rotom-runbook.git`. The user also set `~/.ssh` mode `0700`, the GitHub private-key file mode `0600`, public-key mode `0644`, and SSH config mode `0600`.

Jared's Codex directory `/home/jared/.codex` was set to mode `0700` and `/home/jared/.codex/AGENTS.md` to mode `0600`. A fresh Codex session after those changes successfully loaded the file and followed its legacy documentation path/discrepancy rules.

On 2026-10-14, Fran received private non-credential setup only: `/home/fran/.config/retired-helper` and `/home/fran/.codex` are `fran:fran` mode `0700`, their `repository` and `AGENTS.md` files are mode `0600`, and the configured legacy documentation destination is `/home/fran/documentation/rotom-runbook`. Her AGENTS instructions match Jared's Rotom safety policy. No Fran SSH key, GitHub credential, Codex authentication, or checkout was created or copied; a no-prompt GitHub read test failed as expected. The installed helper accepts either human account but still rejects a missing/non-repository checkout.

Later on 2026-10-14, Fran owner-controlled setup attempted a separate GitHub/Codex path. A local private SSH key now exists at the configured GitHub identity path as `fran:fran` mode `0600`, and the Codex CLI is installed under Fran's private local package directory. GitHub rejected that key, no Codex authentication file exists, and `/home/fran/documentation/rotom-runbook` remains absent; no key, token, passphrase, or authentication content was inspected or recorded. These incomplete local artifacts do not provide GitHub/legacy documentation access and remain deferred rather than satisfying the waived JAR-98 Fran criteria.

## 2A. Preserved Pre-Migration Identity and Access Model

### Account model

The named operational identities at the final pre-migration JAR-6/JAR-21 baseline are below. UIDs are unchanged. Media and Game retain storage GIDs `5000` and `5001`; JAR-6 migrated the eight remaining service primary groups to `5002`–`5009`.

| Account | UID : primary GID | Home | Login shell | Password state | Role evidenced by configuration |
|---|---:|---|---|---|---|
| jared | `1000:1000` | `/home/jared` | `/usr/bin/zsh` | password set | Interactive administrator |
| fran | `1001:1001` | `/home/fran` | `/usr/bin/zsh` | password set | Interactive administrator |
| infra | `997:5002` | `/home/infra` | `/usr/bin/zsh` | locked | Core infrastructure stack |
| game | `995:5001` | `/home/game` | `/usr/bin/zsh` | locked | Palworld services and Gamarr |
| smarthome | `126:5003` | `/home/smarthome` | `/usr/bin/zsh` | locked | Home Assistant and Homebridge |
| media | `127:5000` | `/home/media` | `/usr/bin/zsh` | locked | Jellyfin, Sonarr, Radarr |
| documents | `900:5004` | `/home/documents` | `/usr/bin/zsh` | locked | Service-account skeleton |
| downloads | `901:5005` | `/home/downloads` | `/usr/bin/zsh` | locked | Active Prowlarr/qBittorrentVPN download-stack owner |
| web | `902:5006` | `/home/web` | `/usr/bin/zsh` | locked | Website service account |
| filesync | `903:5007` | `/home/filesync` | `/usr/bin/zsh` | locked | Synchronization service-account skeleton; built-in `sync` remains separate |
| apps | `904:5008` | `/home/apps` | `/usr/bin/zsh` | locked | Application service-account skeleton |
| auth | `905:5009` | `/home/auth` | `/usr/bin/zsh` | locked | Authentication service-account skeleton |

JAR-6 reconciled ownership beneath the eight changed service homes so no object in those home trees retained the old primary GID. Arcane live named-volume data was also reconciled from old Infra GID `986` to `5002`. Historical backup files and containerd snapshot-layer objects retaining old numeric GIDs were intentionally preserved rather than normalized.

The 2026-09-18 Palworld rename used a group rename preserving GID `985` followed by a user rename preserving UID `995` and a home move to `/home/game`. The 2026-09-19 storage-identity migration then changed group `game` from GID `985` to `5001`, leaving UID `995` unchanged. Before that change 10,732 objects beneath `/home/game` used GID `985`; a targeted group migration left zero old-GID objects beneath `/home/game`. The 1,014 old-GID objects found outside the home were all under `/var/lib/containerd` and were intentionally left untouched. Checks at that checkpoint verified `game:x:995:5001::/home/game:/usr/bin/zsh`, group `game:x:5001`, Docker supplementary GID `984`, and `/home/game` plus both Palworld project directories owned `995:5001`. JAR-88 later retained `995:5001` while replacing `game` with `gameserver`.

Media's UID remains **127**. Its primary `media` group changed from historical GID **129** to **5000**. Recorded cleanup checks under `/home/media`, including the final old-GID symlink check, found no remaining GID 129 entries in the checked scope. This is not a claim that GID 129 is absent from every filesystem on Rotom.

The service identities have interactive shells but locked local-password entries. Jared and Fran are the only named accounts with password-set entries. A locked password does not prevent a future SSH public-key login if a valid authorized-keys file is added and SSH policy permits it.

The complete local system-account inventory is present in the host account database. All non-human accounts had locked password entries at the baseline audit. The recorded UID, primary GID, home, and shell inventory below preserves rebuild information, with the later Media GID update applied and the three dedicated service identities included explicitly.

| Account | UID:GID | Home | Shell |
|---|---:|---|---|
| root | 0:0 | /root | /bin/bash |
| daemon | 1:1 | /usr/sbin | /usr/sbin/nologin |
| bin | 2:2 | /bin | /usr/sbin/nologin |
| sys | 3:3 | /dev | /usr/sbin/nologin |
| sync | 4:65534 | /bin | /bin/sync |
| games | 5:60 | /usr/games | /usr/sbin/nologin |
| man | 6:12 | /var/cache/man | /usr/sbin/nologin |
| lp | 7:7 | /var/spool/lpd | /usr/sbin/nologin |
| mail | 8:8 | /var/mail | /usr/sbin/nologin |
| news | 9:9 | /var/spool/news | /usr/sbin/nologin |
| uucp | 10:10 | /var/spool/uucp | /usr/sbin/nologin |
| proxy | 13:13 | /bin | /usr/sbin/nologin |
| www-data | 33:33 | /var/www | /usr/sbin/nologin |
| backup | 34:34 | /var/backups | /usr/sbin/nologin |
| list | 38:38 | /var/list | /usr/sbin/nologin |
| irc | 39:39 | /run/ircd | /usr/sbin/nologin |
| _apt | 42:65534 | /nonexistent | /usr/sbin/nologin |
| dhcpcd | 100:65534 | /usr/lib/dhcpcd | /bin/false |
| messagebus | 101:101 | /nonexistent | /usr/sbin/nologin |
| syslog | 102:102 | /nonexistent | /usr/sbin/nologin |
| usbmux | 103:46 | /var/lib/usbmux | /usr/sbin/nologin |
| tss | 104:103 | /var/lib/tpm | /bin/false |
| rtkit | 105:104 | /proc | /usr/sbin/nologin |
| uuidd | 106:107 | /run/uuidd | /usr/sbin/nologin |
| cups-pk-helper | 107:105 | /nonexistent | /usr/sbin/nologin |
| avahi-autoipd | 108:111 | /var/lib/avahi-autoipd | /usr/sbin/nologin |
| kernoops | 109:65534 | / | /usr/sbin/nologin |
| avahi | 110:112 | /run/avahi-daemon | /usr/sbin/nologin |
| _flatpak | 111:115 | /nonexistent | /usr/sbin/nologin |
| nm-openvpn | 112:116 | /var/lib/openvpn/chroot | /usr/sbin/nologin |
| lightdm | 113:117 | /var/lib/lightdm | /bin/false |
| tcpdump | 114:119 | /nonexistent | /usr/sbin/nologin |
| speech-dispatcher | 115:29 | /run/speech-dispatcher | /bin/false |
| geoclue | 116:120 | /var/lib/geoclue | /usr/sbin/nologin |
| cups-browsed | 117:105 | /nonexistent | /usr/sbin/nologin |
| saned | 118:123 | /var/lib/saned | /usr/sbin/nologin |
| hplip | 119:7 | /run/hplip | /bin/false |
| colord | 120:124 | /var/lib/colord | /usr/sbin/nologin |
| sssd | 121:126 | /var/lib/sss | /usr/sbin/nologin |
| sshd | 122:65534 | /run/sshd | /usr/sbin/nologin |
| _rpc | 123:65534 | /run/rpcbind | /usr/sbin/nologin |
| statd | 124:65534 | /var/lib/nfs | /usr/sbin/nologin |
| xrdp | 125:127 | /run/xrdp | /usr/sbin/nologin |
| smarthome | 126:5003 | /home/smarthome | /usr/bin/zsh |
| media | 127:5000 | /home/media | /usr/bin/zsh |
| documents | 900:5004 | /home/documents | /usr/bin/zsh |
| downloads | 901:5005 | /home/downloads | /usr/bin/zsh |
| web | 902:5006 | /home/web | /usr/bin/zsh |
| filesync | 903:5007 | /home/filesync | /usr/bin/zsh |
| apps | 904:5008 | /home/apps | /usr/bin/zsh |
| auth | 905:5009 | /home/auth | /usr/bin/zsh |
| fwupd-refresh | 988:988 | /var/lib/fwupd | /usr/sbin/nologin |
| systemd-coredump | 989:989 | / | /usr/sbin/nologin |
| polkitd | 990:990 | / | /usr/sbin/nologin |
| systemd-resolve | 991:991 | / | /usr/sbin/nologin |
| game | **995:5001** | /home/game | /usr/bin/zsh |
| systemd-timesync | 996:996 | / | /usr/sbin/nologin |
| infra | 997:5002 | /home/infra | /usr/bin/zsh |
| systemd-network | 998:998 | / | /usr/sbin/nologin |
| dnsmasq | 999:65534 | /var/lib/misc | /usr/sbin/nologin |
| jared | 1000:1000 | /home/jared | /usr/bin/zsh |
| fran | 1001:1001 | /home/fran | /usr/bin/zsh |
| nobody | 65534:65534 | /nonexistent | /usr/sbin/nologin |

No duplicate UID or GID values were found in the baseline audit. The former smart-hub account and group were absent. GID `5001` was later verified unused on Rotom and is now assigned to `game`; GIDs `5002` through `5009` are now assigned to the eight JAR-6 service groups in the order recorded above.



### JAR-22 identity revalidation — 2026-09-22

JAR-22 re-ran `id` for every operational service identity plus Jared and Fran. The expected primary identities matched exactly: Media `127:5000`, Game `995:5001`, Infra `997:5002`, Smarthome `126:5003`, Documents `900:5004`, Downloads `901:5005`, Web `902:5006`, Filesync `903:5007`, Apps `904:5008`, and Auth `905:5009`. Jared remained `1000:1000`; Fran remained `1001:1001`.

`getent group docker` returned GID `984` with Jared, Fran, and all ten operational service accounts as members. JAR-22 did not modify any account, group, home ownership, ACL, sudo rule, or NAS permission. The verification confirms the current identity assumptions used by Docker and NFS before the Rotom v2 migration.


### JAR-23 identity and permission preservation verification — 2026-09-25

JAR-23 made no UID/GID, group-membership, sudo-policy, home-ownership, Docker-socket, or NAS share-permission changes. Preservation checks deliberately used the owning service identities where direct Jared access is not part of the service contract: Palworld Compose validation ran as `game`, website Compose validation ran as `web`, and the NAS recovery helper's read-only traversal checks run as `media` and `game`. The earlier Jared Palworld Compose false negative was confirmed to be a normal `/home/game` traversal/read permission boundary, not a missing Compose file.

The new `/usr/local/sbin/rotom-nas-docker-recovery` and `rotom-nas-docker-recovery.service` run as root because they must start systemd mount/bindfs units, inspect Docker state, and perform service-account traversal tests. They do not grant new login rights or group membership. JAR-23 also reverified that secret contents remain in their existing protected locations; the audit and Linear notes do not contain credential/token/private-key/VPN-secret/Home Assistant secret values.

### JAR-24 backup privilege and permission verification — 2026-09-25

JAR-24 made no UID/GID, group-membership, sudo-policy, home ownership, ACL, Docker-socket, SSH, NAS-share permission, or service-account change. The canonical `backup-to-nas` launcher continued to hand off to the protected root backup path; the Restic password file was referenced only by its existing protected path and its contents were never displayed.

A stale Restic lock was removed with `restic unlock` only after confirming no Restic process and no active backup service. This was a repository-level maintenance action under existing root privileges, not a permission broadening. Final snapshot `fbe1838e` and all verification operations used the existing root backup model. Ordinary Jared/Fran direct access boundaries to the backup share and all service-account storage identities remain unchanged.


### JAR-25 whole-disk image permission verification — 2026-09-25

JAR-25 made no Linux account, UID/GID, supplementary-group, sudo, Docker-socket, SSH, service-home, or service-share permission change. The whole-disk image was written to the existing Shared Drive using a restricted client-side numeric identity during the acquisition workflow. A functional dry run verified that the selected NAS client identity could create, read, and delete inside the protected JAR-25 directory before the full image began.

Post-closeout numeric inspection showed `/mnt/nas-shared-drive/rotom-bare-metal/jar-25-2026-09-25` as UID:GID `977:988`, mode `0700`, with ACL `user::rwx`, `group::---`, `other::---`. The raw image itself presents as UID:GID `977:988`, mode `0660`, with ACL `user::rw-`, `group::rw-`, `other::---`. The acquisition attempted to restrict image files further, but the UNAS/NFS presentation retained group-write on the image. The `0700` parent directory prevents group/other pathname traversal, so effective access to the image remains restricted through that directory.

Rotom locally maps numeric GID `988` to the name `fwupd-refresh`; this display name does not establish application ownership of the NAS object. Numeric NFS identities and functional access tests are authoritative. The observed UID `977` presentation for objects created through the tested client identity is recorded as an UNAS/NFS mapping observation rather than a new Rotom account contract. Because the raw image contains the entire local disk, including protected configuration and secrets, do not broaden the parent directory ACL/mode merely to make the file's group bits look conventional.

### JAR-9 identity rollback — 2026-09-21

The temporary JAR-9 `game-servers` rename was fully reversed. At that checkpoint the identity was again `game` with UID `995`, primary GID `5001`, home `/home/game`, shell `/usr/bin/zsh`, and Docker supplementary membership. Final rollback output verified `game-servers` absent. Palworld and Gamarr were then owned/operated under the `game` domain. JAR-88 later replaced it with `gameserver`; the unused UNAS `Game_Servers` share was never an active identity/storage contract.

### Infrastructure / smart-home identity migration — 2026-09-21

The `rotom` user/group was renamed to `infra` while preserving UID/GID `997:986`; its home moved to `/home/infra`. The `smart-home` user/group was renamed to `smarthome` while preserving `126:128`; its home moved to `/home/smarthome`. Docker supplementary membership was preserved. The documentation ACLs moved with `/home/infra/documentation` and retained Jared/Fran traversal/read access.

At the intermediate account-creation checkpoint, new service accounts were created with same-number primary groups and Docker membership: `documents 900:900`, `downloads 901:901`, `web 902:902`, `filesync 903:903`, `apps 904:904`, and `auth 905:905`; JAR-6 later superseded those primary GIDs with `5004`–`5009`. Each has a locked local password, `/usr/bin/zsh`, a service-owned home mode `0750`, and a service-owned `/home/<name>/docker` root mode `0775`. No application, NAS share, proxy route, or recurring job was created for these identities by this migration. JAR-19 later migrated the website workloads to `web` and removed the legacy `web-host 128:130` user/group/home after verification. The built-in `sync:x:4:65534:sync:/bin:/bin/sync` identity was not modified.

### JAR-6 service storage-GID migration — 2026-09-21

JAR-6 changed primary group numbers only; service UIDs remained stable. Final identities are Infra `997:5002`, Smarthome `126:5003`, Documents `900:5004`, Downloads `901:5005`, Web `902:5006`, Filesync `903:5007`, Apps `904:5008`, and Auth `905:5009`. Docker supplementary GID `984` was preserved. The eight service home trees were reconciled to the new primary groups, including symlink inode ownership where required; final checks found no old primary-GID objects remaining beneath those homes.

At the JAR-6 checkpoint, Arcane live-volume data was changed from GID `986` to `5002`; Arcane used `PGID=5002`, Cloudflare DDNS used `997:5002`, Prowlarr used `997:5002`, and qBittorrent remained `997:5000`. JAR-21 later moved Prowlarr/qBittorrent to Downloads `901:5005`. Historical rollback/backups and containerd snapshot-layer objects with old GIDs were left unchanged. Migration rollback material remains under `/root/jar6-gid-migration-20260921-162313`.

### JAR-19 website identity retirement — 2026-09-21

JAR-19 migrated both website project trees to `/home/web/docker` and reconciled ownership from legacy `web-host 128:130` to `web 902:902` where service-account ownership was intended. Aloha was verified running from the new path; Jared Wines remained stopped by explicit deployment intent. A root-only rollback copy was verified at `/root/jar-19-20260921-024634`. Final guards found no UID 128 process, running Docker bind, active operational path dependency, or required persistent UID/GID `128:130` ownership outside the retiring home. The `web-host` user, group, and `/home/web-host` were then deleted.

### Preserved pre-migration privileged groups and sudo

| Group | GID | Members | Effect |
|---|---:|---|---|
| sudo | 27 | jared, fran | Full sudo access, subject to normal authentication |
| docker | 984 | jared, fran, infra, smarthome, media, game, documents, downloads, web, filesync, apps, auth | Control of the root-owned Docker daemon |
| adm | 4 | syslog, jared | Administrative log access |
| sambashare | 125 | jared | Samba-share access |
| users | 100 | jared, fran | General local-user group |

Jared and Fran can run any command as any user through sudo. Every named operational account, including both human administrators, belongs to docker. The Docker socket is root:docker with mode 0660. Docker group membership is effectively privileged access because its members can create privileged containers, mount host paths, and control other containers.

All seven named accounts also receive four passwordless Linux Mint maintenance commands. Jared and Fran each have a separate exact-command passwordless rule for `/usr/local/sbin/backup-to-unas`: Jared's is in `/etc/sudoers.d/update-to-nas`, and Fran's is in `/etc/sudoers.d/fran-backup-to-nas`. The Fran file was confirmed as `root:root` mode `0440`; the complete sudo configuration and every sudoers.d fragment passed validation. A noninteractive permission check for Fran passed, and the user confirmed the shared `backup-to-nas` command worked. These rules authorize the existing root backup operation; they do not grant direct access to the NAS backup mount.

### Homes, service directories, and Compose files

Every named home is owner-private at mode 0750: owner read/write/execute, primary group read/execute, and no access for others. The September 15 baseline found no extended ACL entries on the named homes, service roots, NAS mount roots, or Docker socket. The targeted September 16 documentation ACLs are recorded separately below and supersede that baseline only for the named documentation paths.

| Path | Owner | Mode |
|---|---|---:|
| /home/infra | infra:infra | 0750 |
| /home/media | media:media | 0750 |
| /home/game | game:game | 0750 |
| /home/smarthome | smarthome:smarthome | 0750 |
| /home/jared | jared:jared | 0750 |
| /home/fran | fran:fran | 0750 |
| /home/infra/docker | infra:infra | 0770 |
| /home/media/docker | media:media | 0775 |
| /home/game/docker | game:game | 0775 |
| /home/smarthome/docker | smarthome:smarthome | 0755 |
| /home/jared/docker | jared:jared | 0775 |
| /home/documents | documents:documents | 0750 |
| /home/documents/docker | documents:documents | 0775 |
| /home/downloads | downloads:downloads | 0750 |
| /home/downloads/docker | downloads:downloads | 0775 |
| /home/web | web:web | 0750 |
| /home/web/docker | web:web | 0775 |
| /home/filesync | filesync:filesync | 0750 |
| /home/filesync/docker | filesync:filesync | 0775 |
| /home/apps | apps:apps | 0750 |
| /home/apps/docker | apps:apps | 0775 |
| /home/auth | auth:auth | 0750 |
| /home/auth/docker | auth:auth | 0775 |

Each service home also contains a verified local `nas-<service>` symbolic link to the matching canonical mount: Media uses `/home/media/nas-media -> /mnt/nas-media`, Game uses `/home/game/nas-game -> /mnt/nas-game`, and Infra through Auth follow `/home/<service>/nas-<service> -> /mnt/nas-<service>`. The links were initially created as generic `nas` names and then renamed in place. The supplied final verification checked all ten new link targets and confirmed each intended service account could traverse its own shortcut, ending with `ALL NAS SHORTCUT RENAMES: PASS`. The verification did not inspect or redefine symlink ownership metadata and did not rerun the full negative cross-account access matrix. Existing target-side UID/GID and mode rules remain the authoritative access controls; use `/mnt/nas-*` as the canonical operational path.

The Docker service directories and their Compose files have matching service ownership:

| Owner | Service directory and Compose mode |
|---|---|
| infra | arcane, cloudflare-ddns, homepage, nginx-proxy-manager retain their recorded modes; JAR-85 retired the inactive `/home/infra/docker/downloads` symlink after verifying Arcane directly mounts the Downloads Docker root and its current database has no reference to the symlink |
| media | jellyfin, radarr, sonarr remain active; no retained Prowlarr/qBittorrent rollback project remains |
| downloads | at that checkpoint `prowlarr` and `qbittorrentvpn` were under `/home/downloads/docker`, owned by Downloads `901:5005`; qBittorrent `.env` was mode `0600` with a narrow Infra read ACL for Arcane discovery |
| game | at that checkpoint palworld-server-jared and palworld-server-fran retained their recorded directory/Compose modes; `gamarr` was under `/home/game/docker/gamarr` with local config owned for `game` (`995:5001`) |
| smarthome | home-assistant and homebridge: directories 0770; compose.yaml 0660 |
| web | alohamillworks.com: directory 0755, compose.yaml 0644; jaredwines.com: directory 0775, compose.yaml 0664; JAR-19 verified `web:web` ownership |
| jared | docker directory exists, but no Compose service directory was found beneath it |

During the earlier Prowlarr migrations, direct access through private service homes required `sudo` or the service identity; those migrations did not broaden `/home/media` permissions. JAR-21 final state is `/home/downloads/docker/prowlarr`, and the old Media/Infra rollback trees are removed.

The post-migration audit at that checkpoint found no active `game-server` references in the inspected systemd unit overrides, `/etc/cron.d`, sudoers fragments, `/usr/local/bin`, `/usr/local/sbin`, shell profile files, symlink targets, `/etc/subuid`, or `/etc/subgid`; no `game` user crontab or old-name linger/mail state was present. The one then-active match was `/home/game/.codex/config.toml`, which was updated to `/home/game/docker/palworld-server-jared`. JAR-88 later superseded the active domain; old-name strings remain in historical records rather than being rewritten.

The parent home mode prevented unrelated local accounts from traversing into these otherwise group-readable Docker trees unless a targeted ACL granted access. At the baseline audit, direct traversal tests confirmed that each named account could read and traverse its own home but could not read or traverse the other six named homes. Media's later group migration changed its numeric group to 5000 while preserving this service-account organization. The Palworld account/home migration initially preserved `995:985`; the later Game GID migration changed group ownership to `5001`. At that checkpoint `/home/game` was mode `0750`; after the targeted migration, zero GID-985 objects remained beneath it and the key roots `/home/game`, `/home/game/docker`, and both Palworld project directories were verified `995:5001`. JAR-88 later retained the numeric identity under `/home/gameserver` and `/srv/rotom/.../gameserver`. The baseline full cross-account traversal matrix was not repeated after these changes.

### Historical documentation permissions — 2026-09-16

At the 2026-09-16 pre-migration checkpoint, the documentation directory and confirmed control files used this ownership and base-mode policy. The current 2026-09-28 paths are documented above and supersede this table for present operations:

| Path | Owner | Base mode | Recorded purpose |
|---|---|---:|---|
| `/home/infra/documentation` | `infra:infra` | `0750` | Authoritative Rotom documentation directory |
| `/home/infra/documentation/AGENTS.md` | `infra:infra` | `0640` | Codex operating, safety, and formatting instructions |
| `/home/infra/documentation/README.md` | `infra:infra` | `0640` | Directory purpose, expected document set, and synchronization policy |

Targeted POSIX ACLs preserve `/home/infra` as a private service home while permitting documentation access:

- `/home/infra` grants named users `jared` and `fran` execute-only traversal (`--x`).
- `/home/infra/documentation` grants both named users read and traversal access (`r-x`).
- `AGENTS.md` and `README.md` grant both named users read-only access (`r--`).
- Default ACLs on `/home/infra/documentation` grant `jared` and `fran` `r-x` on new child directories and effective read access on newly created ordinary files, subject to the inherited ACL mask and the file's creation mode.
- Supplied tests confirm both accounts can read `AGENTS.md` and cannot write it.

This is an intentional exception to the older statement that no extended ACLs existed on the inspected homes: that statement remains a 2026-09-15 baseline, while these named ACLs were added on 2026-09-16. The ACLs do not add either user to the `infra` group and do not grant general listing or read access to unrelated content under `/home/infra`. The `infra` account remains the intended writer of authoritative documentation.

### Docker runtime identity and data access

Most containers leave the Docker user field unset. This typically starts the entrypoint as root, but it does not prove that every application process remains root: images can drop privileges. Arcane explicitly configures 0:0 and Homepage root. Cloudflare DDNS explicitly configures `997:5002` after JAR-6. The other containers inspected had no explicit Docker-configured runtime user.

The current media-owned Compose services—Jellyfin, Radarr, and Sonarr—use `PUID=127`, `PGID=5000`, and `UMASK=002`. Gamarr uses `995:5001`. JAR-21 moved Prowlarr and qBittorrentVPN to Downloads identity `901:5005`; qBittorrent's active config tree and NAS writes now follow Downloads ownership. Historical files/snapshots with earlier Prowlarr/qBittorrent identities remain historical evidence and should not be blanket-reowned.

Runtime mounts create these notable access paths:

- Arcane has a read/write Docker socket and a read/write bind of `/home/infra/docker`; JAR-21 also gives it a read-only same-path bind of `/home/downloads/docker` for project discovery.
- Homepage has a read-only Docker socket.
- Glances has a read-only bind of the host root and a read-only Docker socket. Its former read-only `/mnt/nas-rotom-backup` bind was removed on 2026-09-18.
- Media storage: Sonarr and Radarr read/write `/mnt/nas-media` at `/media` and additionally bind `/mnt/nas-media/torrents` read-only at `/media/torrents`; Jellyfin mounts the library subtree read-only. qBittorrentVPN binds that torrent directory read/write. No Downloader bindfs compatibility view remains in the current topology; Prowlarr has no NAS bind.
- Home Assistant has read-only access to /run/dbus.
- Game Servers storage: Palworld uses local `/srv/rotom/appdata/game-servers` data and no NAS world bind. JDownloader, Igir, the qBittorrent importer, and the browser-upload bridge were retired on 2026-10-05, so no current ROM-inbox write boundary exists. Original Game NAS content remains rollback material and qBittorrent has no Game Servers payload bind.
- Cloudflare DDNS receives a read-only secret-file mount. Its contents were not inspected.


JAR-21 added narrowly scoped ACLs so the Infra-owned Arcane process can discover Downloads projects without making the Downloads home broadly readable: `/home/downloads` grants `infra` execute-only traversal, `/home/downloads/docker/qbittorrentvpn` grants `infra` read/traverse, and the qBittorrent project `.env` grants `infra` read access. Arcane mounts `/home/downloads/docker` read-only. These ACLs are for Arcane Compose discovery; they do not make Infra the application owner and do not grant write access to Downloads configuration.

Filesystem ownership separates Compose files, but Docker-group membership and the Arcane Docker socket bypass that boundary. A Docker-group member can control the daemon and, through it, access host data or other containers. The named service accounts therefore do not have least-privilege separation at the Docker-daemon level.

### Preserved Pre-Migration NAS Identities and Access

Numeric IDs are authoritative for NFS. The implemented service-share mapping is:

| Account | UID | Primary GID | NAS mount | Share-root numeric owner/mode |
|---|---:|---:|---|---|
| media | 127 | 5000 | `/mnt/nas-media` | `988:5000`, `2770` |
| game | 995 | 5001 | `/mnt/nas-game` | `988:5001`, `2770` |
| infra | 997 | 5002 | `/mnt/nas-infra` | `988:5002`, `2770` |
| smarthome | 126 | 5003 | `/mnt/nas-smarthome` | `988:5003`, `2770` |
| documents | 900 | 5004 | `/mnt/nas-documents` | `988:5004`, `2770` |
| downloads | 901 | 5005 | `/mnt/nas-downloads` | `988:5005`, `2770` |
| web | 902 | 5006 | `/mnt/nas-web` | `988:5006`, `2770` |
| filesync | 903 | 5007 | `/mnt/nas-filesync` | `988:5007`, `2770` |
| apps | 904 | 5008 | `/mnt/nas-apps` | `988:5008`, `2770` |
| auth | 905 | 5009 | `/mnt/nas-auth` | `988:5009`, `2770` |

**Current backup correction:** Restic uses `Rotom_Restic_Backup` at `/mnt/nas-rotom-restic-backup`, verified mounted as numeric `988:988` mode `0700`; the underlying local directory was not isolated while fully unmounted in the 2026-09-27 live audit and should not be assigned a definitive mode without a separate check. The old `Rotom_Home_Server_Backup` share is rollback-only and not mounted/used by Rotom. The surrounding table intentionally preserves pre-migration Downloader names and paths; all Downloader guest contracts are now retired. The Shared Drive/JAR-25 ownership observations remain historical evidence.

#### UNAS Isolated Mode and `--manage-gids`

The observed UNAS `rpc.mountd --manage-gids` behavior rebuilds supplementary groups server-side for the requesting UID while retaining the request primary GID. This makes primary service GIDs the reliable ordinary-access contract. Do not assume adding a client-side supplementary storage group will grant access.

#### JAR-6 functional isolation verification

For every new JAR-6 share, the intended service account could traverse/list/create; new files used the account UID and matching GID `5002`–`5009`; child directories inherited setgid; and disposable objects were removed. A different unrelated service account, verified not to belong to the target GID, could not traverse, list, or write. The same positive/negative behavior was repeated successfully after a normal Rotom reboot.

Jared remains outside these service storage groups for ordinary work. Because the roots are `2770`, ordinary Jared denial is expected. For service-specific maintenance, use `sudo -iu <service-user>` rather than broadening group membership. Root/sudo and Docker-daemon control remain privileged routes across these ordinary filesystem boundaries.

### Historical JAR-91 Paperless Documents identity mapping — 2026-10-05 (superseded by JAR-92)

### Historical JAR-92 Library identity and Paperless storage mapping — 2026-10-13

The former `media` user/group is `library`, retaining `127:5000`, a locked password, and home `/home/library`. The active Library application set includes Jellyfin, Radarr, Sonarr, Prowlarr, RomM, qBittorrentVPN, and Paperless. qBittorrent uses `127:5000` with only its Media payload/guard bind.

Paperless uses `USERMAP_UID=127` and `USERMAP_GID=5000`; its live web, consumer, scheduler, and worker processes run as Library. The Documents NFS export remains a distinct document boundary, but its root is `988:5000` mode `2770` and its sole Paperless subtree is `127:5000`; no other Documents export content was present or changed. POSIX ACL mutation is unsupported by this NFSv3 export, and a supplementary Documents-GID test was denied, so this exact ownership migration is the supported least-scope access design. Database/broker state retains image ownership and remains VM-local.

Before JAR-92, Paperless's Compose web service used `USERMAP_UID=900` and `USERMAP_GID=5004`, and its live workers ran as `documents`. This preserved JAR-91 evidence is superseded by the current Libraries mapping and `/mnt/nas-libraries/paperless -> /usr/src/paperless/media` bind.

The historical JAR-91 Downloads assertion is superseded: qBittorrent runs as Library and uses only the retained Media payload/guard. The Downloader share has no qBittorrent payload or recovery-helper role.

### Pre-migration SSH access — historical baseline

The following findings describe the retired pre-migration bare-metal host and are preserved only as historical evidence. They are superseded for the current Proxmox/Debian Phase B environment by **Current administrative public-key access** above.

At that baseline, the active SSH policy permitted password authentication and public-key authentication. It used the standard authorized-keys locations `.ssh/authorized_keys` and `.ssh/authorized_keys2`. Root password login was disabled by the then-recorded `PermitRootLogin without-password` policy, while root public-key login remained allowed if a valid authorized key was configured.

At that historical baseline, only `/home/jared/.ssh/authorized_keys` was found among the checked root and named-home locations. Its directory was mode `0700` and its file mode `0600`, both owned by Jared. Key material was not read. `/root/.ssh` existed at root:root mode `0700`, but no root authorized-keys file was found at the standard locations at that time. No authorized-keys files were found in the other named homes. This is not the current Proxmox state: the current Proxmox `/root/.ssh/authorized_keys` now contains the preserved pre-existing RSA key plus the verified Mac ED25519 administration key documented above.

### Current backup system services and local scripts

The current custom Restic units are `rotom-restic-backup.service` and `rotom-restic-backup-status-api.service`. Neither declares `User=` or `Group=`, so systemd runs them as root by default. The backup service invokes root-protected `/usr/local/sbin/rotom-restic-backup`; the status API invokes `/usr/local/sbin/rotom-restic-backup-status-api`. The backup timer is `rotom-restic-backup.timer` and is enabled/active on the daily `03:00` schedule with persistent catch-up and up to ten minutes randomized delay.

The human-facing `/usr/local/bin/backup-restic-to-nas` launcher is `root:root` mode `0755` and contains only the sudo handoff to `/usr/local/sbin/rotom-restic-backup`, which is `root:root` mode `0700`. The exact-command sudoers fragment is `/etc/sudoers.d/rotom-restic-backup` mode `0440`; the protected credential path is `/etc/restic/restic-repository-password` mode `0600`. Old active `backup-to-unas`, `backup-to-nas`, `unas-backup*`, `update-to-nas`, and `unas-password` control paths were verified absent; dated rollback/history material may retain those old names. No credential contents are recorded here.

## 3. Preserved Pre-Migration Operational and Follow-up Findings

1. **Pre-migration:** Docker was the main cross-service privilege bypass because all named service accounts were Docker-group members. That historical GID `984` membership is not current authority. JAR-30 later verified an empty package-created GID `989`; the 2026-10-14 current overlay supersedes that checkpoint with Fran as its sole member.
2. JAR-6 implemented Infra through Auth storage GIDs `5002`–`5009` and verified intended-account versus unrelated-account NFS behavior for each new share.
3. **Current:** Restic backup storage remains access-protected at mounted NAS metadata `988:988` mode `0700`; JAR-32 moved the active mount to `/mnt/nas-rotom-restic-backup` without broadening ordinary-user access. The 2026-09-27 audit refreshed the mounted metadata but did not isolate the underlying local directory while fully unmounted.
4. SSH password/public-key policy and the existing sudo/docker privilege model were not changed by JAR-6.
5. JAR-21 reconciled Arcane Prowlarr/qBittorrent project paths through the Downloads symlink and enabled `followProjectSymlinks`; unrelated historical Smart Hub/Smarthome records remain technical debt.

### 2026-09-27 live identity/access verification

Read-only post-boot checks refreshed all ten service-account UID:GID pairs and confirmed every service home remains mode `0750`. Every convenience link `/home/<service>/nas-<service> -> /mnt/nas-<service>` resolved correctly, including Downloaders. `docker:x:989:` still has no members; Jared remains in `sudo` but not Docker; Fran remains absent. The Media/Game read-only bindfs compatibility traversals passed under their respective service identities.

### JAR-37 Docker-access exception verification — 2026-09-28

The current Docker-socket audit found exactly three consumers: Arcane has the raw read-write socket as the explicit manual Docker-administration exception; Homepage has a read-only socket for dashboard metadata; and Glances has a read-only socket plus a read-only host-root bind for monitoring. Read-only socket flags do not establish read-only Docker API behavior. No ordinary application container has a Docker socket. Arcane's unattended update and auto-heal controls are disabled; its protected named-volume database has a root-only pre-change rollback copy.

Current non-socket exception fields are Home Assistant's host networking, privileged mode, and read-only D-Bus bind; Homebridge, Nginx Proxy Manager, and Cloudflare DDNS host networking; and qBittorrentVPN's `CAP_NET_ADMIN` for WireGuard. These are current workload-specific exceptions, not defaults. The Phase D owning ticket must justify and verify every retained or new privileged field.

## 4. Outstanding / Needs Verification

- **Intentional:** Fran SSH-key authentication remains deferred on Rotom and PVE. No separate Rotom documentation ACL, legacy documentation/GitHub identity, or Codex entry point is established for her. The historical exact-command backup sudoers fragment remains absent; Fran's current full sudo access makes it unnecessary for authorization, but no manual backup run was performed in this change.
- **Verified by Jared confirmation — account acceptance:** Fran's PVE PAM identity, effective Administrator permissions, direct browser login, Rotom Docker access, and Remote Desktop Commander health are verified.
- **Verified by Jared confirmation — UNAS identity persistence:** required service-share metadata persistence was observed after a NAS/UniFi Drive event; detailed output was not supplied in this chat.
- **Historical by design:** backup/containerd objects may retain old primary GIDs; do not normalize them recursively without a specific verified need.
- **Verified by Jared confirmation — credential rotation:** VPN credential rotation was completed and verified through the appropriate provider/administrator record. No credential values are recorded.
- **Verified by Jared confirmation — live Libraries-NFS loss:** the planned non-disruptive running-workload test completed; detailed observations were not supplied in this chat.

## 5. Preserved Pre-Migration Consolidated Verification — through 2026-09-21

- Final pre-migration named identities: `infra 997:5002`, `media 127:5000`, `game 995:5001`, `smarthome 126:5003`, `documents 900:5004`, `downloads 901:5005`, `web 902:5006`, `filesync 903:5007`, `apps 904:5008`, `auth 905:5009`, `jared 1000:1000`, `fran 1001:1001`.
- At the final pre-migration baseline, all service identities were Docker group `984` members; Jared/Fran also had sudo. This is not current JAR-30 guest membership.
- Locked-password state remains for service accounts; Jared/Fran remain interactive password-set administrators. Built-in Linux `sync` UID `4` is unchanged and distinct from `filesync`.
- No old JAR-6 primary GID remains in the eight changed service home trees. `/home/infra/documentation` now follows Infra group `5002`; its named Jared/Fran ACL model was preserved.
- Arcane and Cloudflare DDNS use Infra identity where configured. Prowlarr and qBittorrentVPN are Downloads-owned at `901:5005`; Arcane has only read/traverse access needed for discovery.
- All eight new service NAS roots are `988:<service GID>` mode `2770`. Positive intended-account and negative unrelated-account tests passed before and after a normal Rotom reboot.
- Media/Game/Backup/Shared Drive retained their ownership/mount roots. JAR-21 removed the old Media/Game torrent subtrees and activated Downloads torrent storage without changing those root identities.

## 6. Historical Identity Verification — 2026-09-15

Read-only checks then reconfirmed the named-account IDs, Docker-group membership, home and NAS modes, Docker socket mode, and the absence of extended ACL entries on the inspected paths. No factual correction was required at that time. The later Media GID and NAS permission changes now supersede the Media portions of that baseline; unchanged security, SSH, backup, and service-account findings retain their original evidence date.

## 7. Reconstruction-Critical Information

- The **Current Identity and Access Model** table in section 2 is the canonical rebuild reference for current UID/GID, home, and workload ownership. The **Account model** table in section 2A is preserved pre-migration evidence and must not be replayed where names or privileges differ.
- The current identity table in section 2 is the canonical reference for `sudo`, `docker`, and other named privilege-bearing memberships. The **Preserved pre-migration privileged groups and sudo** table is historical evidence only and must not be replayed; in particular, do not restore its broad Docker-group membership.
- Home ownership/modes, documentation ACLs, NAS access behavior, and account-switching policy are preserved in the current as-built sections above.
- Numeric UID/GID values are authoritative for NFS behavior; names displayed by a different system must not be used to infer ownership.

## 8. Future Identity / Storage Changes

The JAR-6 service storage-GID plan is no longer proposed; it is current as-built state. Future changes should preserve UIDs and the current service-GID sequence unless a separately verified migration is approved. Commissioned NAS shares do not imply workloads should automatically move to them. Before moving critical data or binding a container to one, inspect the current Compose configuration, define missing-mount behavior, confirm backup/recovery coverage, and re-test ordinary-account isolation.

## 9. Related Documentation

- See `04-NAS-and-Storage.md` for the canonical NAS export, mountpoint, storage-layout, and NFS mount behavior details.
- See `02-Docker-Services.md` for container/runtime deployment context that uses these identities.
- See `06-Maintenance-and-Automation.md` for routine administration and automation workflows.
- See `00-Rotom-Change-Log.md` for the canonical legacy documentation maintenance contract and identity-migration history.
