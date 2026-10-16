# 01 - Rotom Server Inventory

**Documentation set:** Rotom Runbook
**Document role:** Task-first starting page, symptom router, architecture overview, and source map
**Hosts:** PVE hypervisor `pve` and portable Debian VM `rotom`  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-15 — Migrate the private documentation portal to Rotom Support
**Related canonical sources:** `02-Docker-Services.md`, `03-Network-and-Domains.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `06-Maintenance-and-Automation.md`, `07-Users-and-Permissions.md`, `08-Rotom-Directory-Tree.txt`  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

## 1. Purpose and Scope

This document is the front-page architecture/index for Rotom. It summarizes the current system and points to the canonical subject document for detailed facts; it is not the canonical home for full account, container, network, mount, backup, automation, or filesystem registers.

Purpose: front-page index and quick-reference architecture summary.

**RR state labels:** **Current** means the latest supported recorded state; **Historical** preserves dated evidence that may no longer apply; **Retired** means intentionally no longer active; **Proposed** is not implemented; **Needs Verification** means the RR lacks enough current evidence and should not be treated as settled.

## Start by Task or Symptom

Procedures answer **what to do**. Current-state reference answers **what exists**. Reconstruction material supports disaster recovery. Historical and retired sections are evidence, not current instructions. Document `00` is the sole maintenance authority.

| Task or symptom | Canonical destination |
|---|---|
| Website, container, or application unavailable | [02 — Diagnose and Recover a Container or Docker-Backed Website](02-Docker-Services.md#diagnose-and-recover-a-container-or-docker-backed-website) |
| Proxy, listener, DNS, TLS, domain, routing, or external-access problem | [03 — Diagnose the Access Path](03-Network-and-Domains.md#diagnose-a-proxy-listener-dns-tls-routing-or-external-access-failure) |
| NAS path, NFS mount, or storage dependency unavailable | [04 — Diagnose and Recover a Configured NAS Mount](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount) |
| Backup execution, failure, repository verification, or restore | [05 — Backup and Restore Tasks](05-Backup-and-Restore.md#start-by-task-or-symptom) |
| Timer, scheduled automation, update, or maintenance problem | [06 — Automation and Maintenance Tasks](06-Maintenance-and-Automation.md#start-by-task-or-symptom) |
| Permission, ownership, user, group, identity, or access problem | [07 — Identity and Access Tasks](07-Users-and-Permissions.md#start-by-task-or-symptom) |
| Filesystem-path lookup | [08 — Current Topology](08-Rotom-Directory-Tree.txt) |
| RR maintenance or publication | [00 — Current Documentation Maintenance Rules](00-Rotom-Change-Log.md#current-documentation-maintenance-rules) |

### Maintaining the Runbook-First Model

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Supported tasks must remain reachable from this router within one or two links. Update this table whenever a procedure, heading, filename, destination, or canonical owner changes. This document routes tasks; it does not duplicate complete procedures.

### Recommended reading path

1. Read **Current Rotom Summary** for the newest system-wide orientation.
2. Use **How to use the RR** or **Current Source Map** to select the canonical subject document.
3. In that document, read its navigation/authority block and current-state sections before any procedure.
4. Use **Outstanding Decisions and Needs Verification** to identify uncertainty that must not be guessed through.
5. Consult preserved history only for provenance, rollback context, or reconstruction, and reconcile it with current sections plus read-only live evidence first.

The inventory summarizes cross-system facts but does not replace the detailed subject owners listed below. When a summary and a detailed owner appear inconsistent, preserve both, investigate the evidence dates, and record the discrepancy rather than silently choosing one.

### Jared-confirmed recovery follow-up closure — 2026-10-14

Jared confirmed completion and verification of the current Rotom Restic full-data-read plus restore rehearsal, current Rotom VM Backup VMA verification or disposable restore, Libraries/Paperless NAS snapshot retention plus isolated restore, NAS metadata persistence after a NAS/UniFi Drive event, controlled live Libraries-NFS-loss behavior, end-to-end Wake-on-LAN, VPN credential rotation, UniFi/router-policy review, full isolated Palworld disaster-recovery rehearsal, authenticated Paperless acceptance, Fran PVE browser login, legacy NPM-record review, certificate renewal, and dormant Docker-artifact review. Detailed transcripts, identifiers, and retain/remove decisions were not supplied in this chat; retain historical test evidence below as history, but do not present these items as current gaps.

### How to use the RR

Start here, then follow the canonical subject document instead of relying on duplicated detail:

| Need | Canonical source |
|---|---|
| Documentation governance and dated history | [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md) |
| Containers, Compose, images, and runtime deployment | [02-Docker-Services.md](02-Docker-Services.md) |
| Addresses, DNS, ports, proxying, TLS, and exposure | [03-Network-and-Domains.md](03-Network-and-Domains.md) |
| NAS exports, NFS mounts, storage ownership, and automounts | [04-NAS-and-Storage.md](04-NAS-and-Storage.md) |
| Backup scope, repositories, verification, and restore | [05-Backup-and-Restore.md](05-Backup-and-Restore.md) |
| Timers, maintenance, monitoring, and administration workflow | [06-Maintenance-and-Automation.md](06-Maintenance-and-Automation.md) |
| Accounts, UID/GID contracts, ACLs, sudo, and access policy | [07-Users-and-Permissions.md](07-Users-and-Permissions.md) |
| Filesystem topology and important paths | [08-Rotom-Directory-Tree.txt](08-Rotom-Directory-Tree.txt) |

Within each subject document, use the current-state sections first. Use historical or pre-migration evidence only for provenance, reconstruction, or recovery; do not replay it as current instruction without reconciling it against live read-only state.

## 2. Current Rotom Summary

**Evidence date:** 2026-10-15 unless a row states otherwise.

Rotom is Debian 13 VMID 100 on Proxmox VE host `pve`. The guest uses reserved address `192.168.1.69`; PVE uses `192.168.1.68`; the UniFi NAS uses `192.168.1.70`.

The verified application topology is **23 running Compose-managed containers, 18 Docker networks, two named volumes, and 18 Compose definitions**. The definitions are grouped under five local domains:

| Domain | Identity | Active roots | Current services |
|---|---:|---|---|
| Platform | `997:5002` | `/srv/rotom/{stacks,appdata,secrets}/platform` | Nginx Proxy Manager, Homepage with Glances, Arcane, Cloudflare DDNS, Rotom Support |
| Libraries | `127:5000` | `/srv/rotom/{stacks,appdata,secrets}/libraries` | Jellyfin, Radarr, Sonarr, Prowlarr, qBittorrentVPN, RomM, Paperless |
| Hosted Websites | `902:5004` | `/srv/rotom/{stacks,appdata}/hosted-websites` | Aloha Millworks, Jared Wines |
| Game Servers | `995:5001` | `/srv/rotom/{stacks,appdata,secrets}/game-servers` | Palworld Jared, Palworld Fran |
| Smart Home | `126:5003` | `/srv/rotom/{stacks,appdata}/smart-home` | Home Assistant, Homebridge |

The five service accounts are locked. Their NSS home fields still name absent `/home/<service>` paths; those paths are not deployment or recovery roots. Human administrators are Jared and Fran. Fran is a full administrator and the sole human member of Docker group GID `989`; Docker membership is root-equivalent.

Homepage and Glances share `/srv/rotom/stacks/platform/homepage/compose.yaml` and the private `homepage_default` network. Glances has no host listener. `jaredwines.com` is intentionally active and publishes only `127.0.0.1:7777:80` for NPM proxy host `8`. Aloha publishes `7778:80`.

Remote Desktop Commander runs from `/opt/desktop-commander` as locked identity `desktopcmd` (`1001:1001`). Its former home is absent.

## 3. Current Storage Summary

Rotom has five current guest NFSv3 systemd-automount contracts:

| NAS export | Guest path | Purpose |
|---|---|---|
| `Game_Servers/.data` | `/mnt/nas-game-servers` | Empty Game Servers service boundary; Palworld state remains VM-local |
| `Smart_Home/.data` | `/mnt/nas-smart-home` | Empty Smart Home service boundary; application state remains VM-local |
| `Libraries/.data` | `/mnt/nas-libraries` | Authoritative Libraries payload, including Paperless media |
| `Hosted_Websites/.data` | `/mnt/nas-hosted-websites` | Empty Hosted Websites service boundary; website content remains VM-local |
| `Rotom_Restic_Backup/.data` | `/mnt/nas-rotom-restic-backup` | Rotom Restic repository |

All five were observed mounted read/write on 2026-10-15. The Libraries recovery helper validates the exact source, re-arms the generated automount after boot-time start-limit failure, and conditionally recovers only eligible NAS-dependent containers.

Historical guest paths such as `/mnt/nas-media`, `/mnt/nas-library`, `/mnt/nas-downloaders`, `/mnt/nas-documents`, `/mnt/nas-filesync`, `/mnt/nas-web`, `/mnt/nas-gameserver`, `/mnt/nas-smarthome`, `/mnt/nas-customapps`, `/mnt/nas-apps`, `/mnt/nas-auth`, and `/mnt/nas-shared-drive` are retired locally. Their absence does not establish deletion of NAS-side exports or data. Do not recreate them from historical records.

## 4. Current Backup and Recovery Summary

- **Rotom Restic Backup:** repository `/mnt/nas-rotom-restic-backup/rotom-restic-backup`; worker `/usr/local/sbin/rotom-restic-backup`; timer around `03:00`; retention `7 daily / 4 weekly / 12 monthly`; protected local credential path `/etc/restic/restic-repository-password`.
- **PVE Restic Backup:** repository `/mnt/nas-pve-restic-backup/pve-restic-backup`; daily `04:00`; current credential path `/etc/restic/restic-repository-password` on PVE.
- **Rotom VM Backup:** Proxmox job `rotom-vm-daily`; VMID `100`; daily `05:00`; snapshot mode with zstd; retention `7 daily / 4 weekly / 6 monthly`.
- **NAS-resident application data:** outside Rotom Restic because `/mnt` is excluded. Confirm the applicable NAS snapshot and isolated-restore path before relying on it.

Current recovery procedures and limitations are owned by [05 — Backup and Restore](05-Backup-and-Restore.md). Backup layers are complementary and must not be treated as interchangeable.

## 5. Retired Infrastructure Warnings

Uptime Kuma and its Rotom/PVE alert-delivery layer, Syncthing, Gamarr, JDownloader, the ROM importer, Igir automation, and the browser-upload bridge are retired. Their former service ports `3002`, `8384`, `6767`, and `8090` had no listeners during the 2026-10-15 audit.

The former external Docker bridges `rotom-proxy`, `rotom-arr`, and `rotom-monitoring` are absent from active Compose definitions. Actual residual Docker objects were not independently enumerated during the documentation cleanup; do not infer authorization to delete any object.

The alerting worker, units, timers, scripts, Homepage integration, and delivery configuration are absent. An empty root-only `/var/lib/rotom-alerting` directory remains; its existence is not evidence that alerting is active.

## 6. Outstanding and Needs Verification

- Arcane currently reports `autoUpdate=true`, `autoHealEnabled=true`, `scheduledPruneEnabled=false`, and `followProjectSymlinks=true`. Exact exclusions and intended long-term policy remain unresolved.
- Root-only rollback archives for the retired shared Docker bridges and the pre-change NAS-recovery helper were not readable during this audit. Retain or remove them only after a separately authorized exact-path inspection.
- Palworld Jared and Fran were running and healthy on 2026-10-15; their intended persistence relative to the earlier deliberately stopped state remains an owner decision.
- Continue normal monitoring of the historical PVE NIC/reset and non-fatal iGPU option-ROM warning. Do not infer a single cause without new evidence.
- Select and verify an incident-appropriate recovery point at recovery time; retention changes available snapshot and archive sets.

## 7. Rotom Runbook Model

Rotom Runbook is every file intentionally kept in the project's Available Sources. The numbered files `00`–`08` are the complete current nine-file set. Repository support files such as `AGENTS.md`, `README.md`, and `support/tooling/*` are not RR members unless intentionally added to Available Sources.

The independently verified RR checkout is `/home/jared/documentation/rotom-runbook` on Rotom and `/Users/jared/Documents/ChatGPT/rotom-runbook` on the Mac. In each environment, `rr path` resolves the checkout through `~/.config/rr/repository`. Available Sources replacement remains manual by Jared.

The private **Rotom Support** portal presents the authoritative numbered set below at `/runbook/`, plus the explicitly selected supplementary `support/website/architecture/index.md` and `support/website/guides/index.md` inputs. Those supplementary sources do not change Runbook membership or Available Sources; they explain and route to the numbered sources rather than establish operational truth.

## 8. Current Source Map

| File | Canonical subject |
|---|---|
| `00-Rotom-Change-Log.md` | Maintenance contract and protected history |
| `01-Rotom-Server-Inventory.md` | Task router and current high-level inventory |
| `02-Docker-Services.md` | Applications, containers, Compose, and service recovery |
| `03-Network-and-Domains.md` | Addressing, listeners, DNS, TLS, proxying, and access |
| `04-NAS-and-Storage.md` | NAS exports, mounts, storage identity, and mount recovery |
| `05-Backup-and-Restore.md` | Backup systems, recovery evidence, and restore procedures |
| `06-Maintenance-and-Automation.md` | Timers, maintenance, monitoring, and RR tooling |
| `07-Users-and-Permissions.md` | Accounts, ownership, privilege, ACLs, and access |
| `08-Rotom-Directory-Tree.txt` | Concise current topology and recovery-critical paths |

## 9. Canonical Fact Ownership

Document 01 routes tasks and summarizes current state; it does not replace the detailed owning documents. Application facts belong to document 02, transport to 03, storage to 04, backup/recovery to 05, automation to 06, identities to 07, topology to 08, and governance/history to 00.

When a current fact changes, update its owner, affected runbook procedure, task-router destination if needed, revalidation triggers, structural tests, and the protected change log in the same authorized task.
