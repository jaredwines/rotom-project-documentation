# 03 - Rotom Network and Domains

**Documentation set:** legacy documentation set  
**Document role:** Canonical source for Rotom LAN, DNS, Docker networking, ports, Cloudflare, NPM, and domain routing  
**Hosts:** PVE hypervisor `pve` plus Debian VM `rotom`  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-15 — Restore Jared Wines and privatize Homepage monitoring
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1. Purpose and Scope**, **2. Current Host, VM, and NAS Network Baseline**, and **10. Current Nginx Proxy Manager** for supported current-state context.
- Sections explicitly titled **Pre-Migration**, **Historical**, or **Preserved** are evidence and reconstruction context, not current firewall, DNS, listener, or routing instructions.
- Use **Reconstruction-Critical Information** only after comparing it with the current sections and read-only live state.
- Review **Known Gaps / Needs Verification** before treating router, exposure, certificate, or application-path behavior as settled.
- This file owns network and routing facts; storage ownership, service deployment, backup, and account privileges remain canonical in documents 02 and 04–07.

## 1. Purpose and Scope

### Current Jared Wines route and private Glances access — 2026-10-15

`jaredwines.com` is active behind NPM proxy host `8`. Its container publishes only `127.0.0.1:7777:80`; NPM remains host-networked and forwards HTTP to `127.0.0.1:7777` with forced TLS. The host record already held that target, but its generated NPM configuration was stale and pointed at the LAN address; regenerating only host `8` corrected the active upstream. Loopback HTTP, local-SNI HTTPS, and public `https://jaredwines.com` each returned `200` with the expected portfolio page. Homepage has an internal container-host mapping for `jaredwines.com` to `192.168.1.69` so its existing HTTPS monitor reaches NPM; no public DNS, router, firewall, or certificate setting changed.

Glances no longer has a host listener, including no `127.0.0.1:61208` publication. It is a private service on the existing `homepage_default` Docker bridge and Homepage reaches it as `http://glances:61208`; all eight Glances-widget API categories, including containers, returned `200` from Homepage. The retired `glances_default` bridge is absent. This private migration created no new Docker network and does not make Glances LAN-reachable.

This overlay supersedes earlier same-day unavailable-Jared-Wines and loopback-`61208` observations.

### Post-cleanup Docker network and route audit — 2026-10-15

The final read-only snapshot contained 19 Docker networks: the three Docker built-ins, fifteen intended private project bridges, and unexpected `jaredwinescom_default`. The retired shared bridges `rotom-proxy`, `rotom-arr`, and `rotom-monitoring` remain absent, and no active container or current Compose definition references them. Without the unexpectedly deployed Jared Wines project, the topology is the documented eighteen networks.

All documented listeners and unauthenticated direct/local-SNI route probes returned their expected responses. Jared Wines is running but publishes no host port, so NPM's retained `127.0.0.1:7777` upstream still returns `502`; its runtime presence does not make the route operational. Any older current-facing sentence that says Paperless, RomM, Rotom Docs, Aloha, or another retained workload still joins `rotom-proxy` is superseded by the shared-network retirement and this audit. Historical dated topology remains preserved as history.

### Current Docker-network and Glances-access state — 2026-10-15

The former external Docker bridges `rotom-proxy`, `rotom-arr`, and `rotom-monitoring` are retired: active Compose definitions no longer declare them, no running container attaches to them, and Docker no longer lists them. NPM remains host-networked and all retained NPM routes continue to use their documented host or loopback upstream listeners.

Prowlarr's Radarr/Sonarr integrations use `http://192.168.1.69:7878` and `http://192.168.1.69:8989`; Radarr and Sonarr each use qBittorrentVPN at `192.168.1.69:8080`. Glances now publishes only `127.0.0.1:61208`; it is not LAN-published. Homepage retains `host.docker.internal:host-gateway` and its Glances widgets use `http://host.docker.internal:61208`, which passed a container-to-loopback API check. Historical bridge/subnet assignments below are not current reconstruction instructions.

### Current PVE network and exposure audit — 2026-10-14

PVE uses `nic0` as the physical Intel I219-V interface and `vmbr0` at `192.168.1.68/24` with gateway `192.168.1.1`; VM 100 attaches through `tap100i0` without a configured VLAN tag or bridge VLAN filtering. The physical link negotiated 1 Gbit/s full duplex, Wake-on-LAN mode `g` is enabled, and the documented TSO/GSO-off mitigation remains effective. PVE reached Rotom (`192.168.1.69`) and the UNAS (`192.168.1.70`) without packet loss during the audit.

The Proxmox firewall reported `Status: disabled/running`; cluster and node rule listings were empty. The PVE web UI on `8006` and read-only monitoring APIs on LAN-bound `8788` and `8789` therefore depend on LAN reachability and their application/access controls rather than an active PVE firewall policy. The NPM route `pve.rotom.casa -> https://192.168.1.68:8006` with access-list ID `1` remains active, but it is not a substitute for host firewall policy. Jared explicitly accepted this trusted-LAN design on 2026-10-14; no hardening or firewall-project work is planned from that decision.

### Current route, DNS, listener, and certificate audit — 2026-10-14

The live NPM SQLite database has sixteen undeleted, enabled proxy hosts: `home-assistant.rotom.casa -> 127.0.0.1:8123`, `homebridge.rotom.casa -> 127.0.0.1:8581`, `nginx.rotom.casa -> 127.0.0.1:81`, `arcane.rotom.casa -> 127.0.0.1:3552`, `jaredwines.com -> 127.0.0.1:7777`, `rotom.casa -> 127.0.0.1:3001`, `alohamillworks.com -> 127.0.0.1:7778`, `qBittorrent.rotom.casa -> 127.0.0.1:8080`, `radarr.rotom.casa -> 127.0.0.1:7878`, `sonarr.rotom.casa -> 127.0.0.1:8989`, `jellyfin.rotom.casa -> 127.0.0.1:8096`, `prowlarr.rotom.casa -> 127.0.0.1:9696`, `paperless.rotom.casa -> 127.0.0.1:8000`, `romm.rotom.casa -> 127.0.0.1:8081`, `docs.rotom.casa -> 127.0.0.1:8082`, and `pve.rotom.casa -> 192.168.1.68:8006` over HTTPS. The Docs and PVE routes use access-list ID `1`; the other current routes have no NPM access list. The database retains deleted historical rows for Smart Hub/Portainer/Homebridge legacy naming, Palworld, OliveTin, PalTools, Gamarr, Syncthing, Uptime Kuma, and ROM upload.

Local SNI probes returned expected application responses for all tested active routes: `200` for Rotom, Arcane, Docs, PVE, Homebridge, qBittorrent, RomM, and Aloha; `302` for Jellyfin, Radarr, Sonarr, Prowlarr, and Paperless. Jared Wines returned the expected `502` because its upstream remains intentionally undeployed. The canonical Home Assistant name is `home-assistant.rotom.casa`; the similarly spelled `homeassistant.rotom.casa` is not a configured route.

Certificate ID `24` covers `*.rotom.casa` and `rotom.casa` and expires 2026-12-10 UTC. Certificate ID `16` covers `alohamillworks.com` and expires 2026-11-21 UTC. Certificate ID `14` covers `jaredwines.com` and expires 2026-12-04 UTC. These are dated observations; renewal remains an operational dependency.

The LAN resolver currently overrides only `rotom.casa` to `192.168.1.69`. The tested `*.rotom.casa` service names resolve through the LAN resolver to public `68.8.40.225`, so ordinary LAN access to those names depends on public-IP hairpin/NAT behavior rather than a wildcard local override. Public resolver queries were unavailable from this execution context; the public-IP value above is the answer returned by the LAN resolver.

Current listener inspection shows that NPM is not the only LAN-reachable path. TCP `80`, `81`, and `443` are LAN-wide, as are Homepage `3001`, Arcane `3552`, Aloha `7778`, Jellyfin `8096`, Radarr `7878`, Sonarr `8989`, Prowlarr `9696`, Home Assistant `8123`, Homebridge `8581`, and the backup-status API `8787`. qBittorrent `8080`, RomM `8081`, Paperless `8000`, and Rotom Docs `8082` are loopback-only. The guest has no UFW installation per the current VM baseline. Jared explicitly accepted these direct listeners as the intended trusted-LAN design on 2026-10-14; no firewall change is planned.

### Current Paperless route — 2026-10-13

Paperless is active from the Libraries deployment and listens only at `127.0.0.1:8000`. The retained NPM proxy record for `paperless.rotom.casa` is operational: both a loopback request and an SNI request through NPM returned the expected `302` login redirect. Its web service, database, and broker use only `paperless_default`; the former `rotom-proxy` attachment is retired. Historical statements that Paperless is absent or that its retained route has no upstream are superseded by this overlay.

### Current Library routes — 2026-10-13

The supported current Library-facing routes are qBittorrentVPN through loopback-only `127.0.0.1:8080` and NPM host `14`, plus Prowlarr through NPM host `18`; 2026-10-14 local NPM/SNI probes returned `200` and `302` respectively. RomM retains its established loopback-only route and returned `200`; the internal-only game-library alias uses the active `/mnt/nas-libraries` bind. Paperless remains loopback-only at `127.0.0.1:8000` and returned the expected login `302` through NPM. Prowlarr remains LAN-published at `9696`; its NPM route is operational but direct LAN exposure remains subject to the current exposure-policy decision.

### Current Homepage Platform presentation — 2026-10-13

Homepage's former **Rotom Management** section is now **Platform**, with Rotom Docs moved into it and no standalone Documents section. The stale Uptime Kuma card/site monitor was removed, consistent with the retired Uptime Kuma NPM route, listener, and service mapping; it does not restore any route or service. The former **Library** service section and matching layout key are now **Libraries**. Homepage was `running healthy`; its API contained no Uptime Kuma entry; and local-host and public HTTPS checks returned `200`.

### JAR-95 current monitoring-route state — 2026-10-13

The Uptime Kuma NPM route and Homepage mapping are retired; no Uptime Kuma listener or Docker service remains. This is not evidence of a public status page, and JAR-95 makes no claim that one existed. Homepage itself remains healthy: local `Host: rotom.casa` and public HTTPS checks returned `200`. The PVE HTTPS UI route plus the read-only PVE CPU-temperature (`8788`) and backup-status (`8789`) services remain unrelated and were not changed.

qBittorrentVPN is active with a loopback-only `127.0.0.1:8080` listener; NPM proxy host `14` serves `qBittorrent.rotom.casa` and returned `200`. Prowlarr is active on `9696`; NPM proxy host `18` returned its expected `302`. Paperless is active again through its loopback `8000` listener and retained NPM host; this paragraph's former absence statement is superseded by the current Paperless route overlay. The retained Syncthing NPM record was not changed when the absent service's Homepage presentation was removed.

The current overlays above are authoritative for the Platform and Library service set. Older route and upstream statements below are retained as dated evidence and must not be read as proof that retired Syncthing or Paperless services remain active.

### Historical JAR-94 service-exposure state — 2026-10-13

Superseded for current service/presentation state by the JAR-95 and Homepage overlays above; retained as recovery evidence.

Nginx Proxy Manager, Homepage, Arcane, Glances, Uptime Kuma, Rotom Docs, and Cloudflare DDNS were recovered as the active Platform workload. No current container serves Syncthing, Paperless, Prowlarr, or qBittorrentVPN. The recovered NPM state was preserved rather than selectively rewritten, so proxy records for retired upstreams require a separate authenticated configuration audit before they are treated as usable routes. This section supersedes any unqualified current-service claims below; historical routing evidence remains historical.

### PVE web UI NPM route — 2026-09-30

NPM proxy host `pve.rotom.casa` forwards using **HTTPS** to the PVE host at `192.168.1.68:8006`. The enabled route uses certificate ID `24`, forced TLS, WebSocket upgrade, and access-list ID `1`. A local-SNI HTTPS request returned the Proxmox Virtual Environment page, while direct `https://192.168.1.68:8006` returned HTTP `200`. This confirms the Rotom-to-PVE proxy path; it does not establish public-WAN reachability or test authenticated VM-console WebSocket use. No DNS, certificate, firewall, VLAN, WAN-forwarding, PVE-service, or other network configuration changed in this verification.

### JAR-5 private documentation route — 2026-09-29

`rotom-docs` joins `rotom-proxy` but retained host-networked NPM cannot use Docker DNS on that bridge. NPM proxy host 22 therefore routes `docs.rotom.casa` to the loopback-only `127.0.0.1:8082` compatibility listener using existing wildcard certificate ID 24 and forced TLS. The dedicated `Rotom Docs trusted LAN` access list allows only `127.0.0.1` and `192.168.1.0/24`; an untrusted bridge request was denied with HTTP `403`. Local-SNI HTTPS returned `200`, including the rendered directory-tree page. This is an intentionally private administrative route: public Internet access is not expected or required, and no DNS, WAN-forwarding, VLAN, firewall, certificate, or other proxy route was changed.

Historical JAR-75 note: Syncthing was once proxied through loopback `127.0.0.1:8384`. Syncthing is now retired; the retained proxy record was not revalidated and is not an operational route.

### JAR-8 Paperless route — 2026-09-29

### JAR-10 RomM route — 2026-09-29

### JAR-90 ROM upload route — retired 2026-10-05

NPM proxy host `26` for `romm-upload.rotom.casa` is deleted/disabled and its generated configuration is absent. The former loopback `127.0.0.1:8090` bridge listener and its credential were removed; no browser-upload route remains. No DNS, firewall, or WAN policy changed.

RomM joins `rotom-proxy` and its database/broker remain private. Retained host-networked NPM routes `romm.rotom.casa` to the loopback-only `127.0.0.1:8081` compatibility listener using wildcard certificate ID 24; external HTTPS validation passed.

Proxy host `21` additionally defines an internal-only `/library/` location aliased to NPM's read-only active Libraries game bind. It is not a public browseable library route. Its exact post-JAR-96 configured source is documented in `02`/`04`; no NPM user/PUID/PGID or Docker-socket change was made for JAR-90.

Paperless web service attaches to `rotom-proxy` and exposes only `127.0.0.1:8000`; PostgreSQL and Valkey stay internal. Retained host-networked NPM routes `paperless.rotom.casa` to that loopback endpoint with existing wildcard certificate ID 24. Local SNI HTTPS returned a login redirect. Although Rotom's own public-IP hairpin path does not reach this route, a 2026-09-29 independent public TLS assessment reached it externally, received the expected `302` login redirect, and followed it to the login page with HTTP `200`.

### JAR-45 Web v2 network compatibility — 2026-09-29

Aloha Millworks now joins `rotom-proxy` from its v2 Web Compose module, but retained NPM remains host-networked and cannot use Docker DNS on that bridge. Its established `alohamillworks.com -> 192.168.1.69:7778` compatibility upstream and the `7778` listener therefore remain necessary. No NPM route, DNS, WAN, TLS, certificate, firewall, or VLAN change was made; local and public HTTPS checks returned HTTP 200.

### JAR-43 Smart Home v2 network compatibility — 2026-09-29

Home Assistant and Homebridge moved only their Compose/application-state paths to the v2 Smart Home domain. Both intentionally remain Docker host-network exceptions, preserving the established TCP listeners on `8123` and `8581` plus Home Assistant SSDP/mDNS and Homebridge/Avahi mDNS discovery behavior. The existing NPM routes remain `home-assistant.rotom.casa -> 192.168.1.69:8123` and `homebridge.rotom.casa -> 192.168.1.69:8581`; no proxy-host, certificate, DNS, WAN-forwarding, VLAN, firewall, or Docker-network change was made. Post-reboot local and HTTPS route probes returned HTTP 200 for both applications.

### JAR-38 Infra v2 network convergence — 2026-09-28

JAR-40 attached Jellyfin, Radarr, Sonarr, Prowlarr, and Gamarr to `rotom-proxy`; Radarr/Sonarr/Prowlarr/Gamarr also attached `rotom-arr`. JAR-77 retired Gamarr and its private network/port while retaining both shared networks for the remaining services. NPM is retained in host networking.

Homepage and Glances now use the external `rotom-monitoring` bridge (`172.31.0.0/16`) and resolve each other through Docker service DNS; Glances has no host-published listener. Homepage and Arcane also attach to `rotom-proxy` (`172.29.0.0/16`) for the Caddy-era proxy handoff, while their current host ports `3001` and `3552` remain only because the existing NPM host-network proxy still consumes them. Cloudflare DDNS moved from host networking to its ordinary private `cloudflare-ddns_default` bridge; it needs no inbound listener and its immediate update confirmed outbound/DNS operation. NPM remains the only Infra reverse proxy listener on 80/81/443 until JAR-39. No DNS, WAN forwarding, VLAN, certificate, or NPM route configuration changed.

### JAR-36 v2 Docker network contract — 2026-09-28

Three empty external local bridge networks now reserve v2 cross-stack roles: `rotom-proxy` for a future reverse proxy and proxied HTTP applications, `rotom-arr` for Arr/qBittorrent control-plane traffic, and `rotom-monitoring` for dashboard/monitoring integrations. Phase D stacks use service/container DNS on these networks rather than LAN host ports where practical. Existing production bridges and ports were retained. Home Assistant and Homebridge remain intentional host-network discovery exceptions; no VLAN redesign occurred.

### Evidence provenance

Last updated: 2026-10-14 (Rotom Restic Backup, PVE Restic Backup, and Rotom VM Backup display terminology standardized; technical VZDump references and pre-migration evidence retained)
Project: **Rotom-Home-Server**

This document records Rotom's networking state using three read-only host audits collected on 2026-09-14:

- `rotom-network-audit.txt` at 21:42:22 -07:00
- `rotom-network-audit-part2.txt` at 21:47:17 -07:00
- `rotom-network-audit-part3.txt` at 21:55:47 -07:00

Targeted 2026-09-18 live output additionally reverified the `homepage` Docker network after the Homepage/Glances project recreation and inspected UNAS NFS export identity/root-handling behavior. Separate supplied 2026-09-19 commissioning output verified the exact `Game/.data` export, the `/mnt/nas-game` fstab entry and active systemd automount, Game share root `988:5001` mode `2770`, and the service-user/negative access matrix. Later 2026-09-19 Prowlarr and central-downloader deployment output verified the recreated services and preserved host ports. Those targeted checks initially did not capture every bridge subnet or runtime IP; a subsequent full 2026-09-19 Docker-network audit superseded that limitation and established the current map recorded under **Docker Network Architecture**. UniFi/router-side DHCP reservation, VLAN, WAN-forwarding, and router-firewall configuration remain outside the host audit.

The first audit established host identity, interfaces, routing, DNS, network management, Docker container networking, and the initial Nginx Proxy Manager runtime state. Its collector stopped during environment redaction. The second audit completed Nginx Proxy Manager, Cloudflare DDNS, socket, firewall, DNS, and Docker-network details. The third audit verified public-versus-local DNS behavior, current public IPv4, DHCP lease details, NPM TLS metadata, IPv6 state, NFS/NAS networking, and reachability of legacy NPM upstreams.

The network baseline is the 2026-09-14 audits, supplemented by the 2026-09-15 verification update below. The 2026-09-16 documentation revision incorporates the user-supplied "Torrent Seeding Explanation" conversation for the media UID/GID change, isolated NFS behavior, and service-account storage policy. It is not a new live inspection. Network addresses, listeners, and other unchanged observations retain their audit dates. Remaining proposed future NAS shares are labeled separately from observed configuration; the Game share is current as of 2026-09-19.

JAR-26 removed the old `.69` fixed-IP/client association for the physical NIC. JAR-28 then established and verified the VM reservation for `.69` and restored the local `rotom.casa` DNS override. Other WAN forwarding, VLAN, and router-firewall policy remains outside the captured host evidence.

No passwords, API tokens, private keys, VPN credentials, cookies, or other authentication secrets are recorded here.

---

## 2. Current Host, VM, and NAS Network Baseline

The 2026-10-14 current audit in section 1 controls where it differs from the JAR-68-era details retained below.

| Layer | Verified current identity |
|---|---|
| PVE host | hostname `pve`; FQDN `pve.rotom.casa`; `vmbr0` `192.168.1.68/24`; gateway/DNS `192.168.1.1` |
| Physical NIC | `nic0`, Intel I219-V / `e1000e`, MAC `1c:69:7a:0e:f2:f7`; WOL `g`; persistent WOL/offload settings remain in `/etc/network/interfaces` |
| Rotom VM | hostname `rotom`; FQDN `rotom.casa`; VirtIO MAC `BC:24:11:97:10:47`; UniFi-reserved `192.168.1.69/24`; gateway/DNS `192.168.1.1` |
| NAS | UniFi UNAS 2 at `192.168.1.70` |

JAR-68 renamed only the physical PVE identity and PVE-only support components. PVE `/etc/hosts` now contains canonical `192.168.1.68 pve.rotom.casa pve`; the temporary `proxmox` / `proxmox.rotom.casa` compatibility names were removed after final acceptance. VMID 100 remains `rotom` and its guest network identity is unchanged.

### Current administrative SSH access

| Target | Mac aliases | Destination/user | Identity file | Verification |
|---|---|---|---|---|
| Rotom VM | `rotom`, `rotom.casa` | `rotom.casa` / `jared` | `~/.ssh/id_ed25519_rotom` | key-only login previously verified |
| PVE host | `pve`, `pve.rotom.casa` | `192.168.1.68` / `jared` | `~/.ssh/id_ed25519_pve` | Jared's PVE `sudo` and `jared@pam` Administrator access verified; Jared confirmed the canonical alias now targets this account and the matching root-authorized key was removed |

Fran is now a human administrator on both hosts, but no Fran SSH alias, identity file, authorized-key path, or successful SSH login is recorded. PVE `/home/fran/.ssh` was verified absent; SSH-key setup is intentionally deferred. Password authentication policy was not changed. Key contents and credentials are never recorded. The public-key comment may retain historical wording; it is metadata only and not an active alias.

PVE's existing HTTPS management route is unchanged. Enabled `fran@pam` now has propagated `Administrator` access at `/`, matching Jared's authorization scope; effective permissions were verified from the PVE CLI, and Jared subsequently confirmed direct Fran browser-login acceptance.

### Current PVE monitoring bridge

The read-only physical CPU temperature helper is `/usr/local/sbin/pve-cpu-temp-api` with `pve-cpu-temp-api.service`, listening on `192.168.1.68:8788`. Homepage uses the exact label **`PVE CPU Temperature`** and the Glances-compatible sensor endpoint. Final post-reboot monitoring verification passed.

JAR-69 adds the separate read-only `/usr/local/sbin/pve-backup-status-api` with `pve-backup-status-api.service`, listening only on `192.168.1.68:8789`. Its `/health`, `/pve-restic-backup-status`, and `/rotom-vm-backup-status` endpoints are reachable from Rotom and the Homepage container; it exposes status only and has no backup-control endpoint. Homepage consumes the two schedule endpoints as **PVE Restic Backup Schedule** and **Rotom VM Backup Schedule**.

### Current NAS / NFS connectivity

The former ten-automount count is historical. The final 2026-10-13 fstab and removal verification establishes five current guest NFS automount contracts: Game Servers, Smart Home, Rotom Restic Backup, Libraries, and Hosted Websites. There are no documented active bindfs compatibility views. Reconfirm their live activation read-only with `findmnt` and `systemctl list-units --type=automount` on Rotom. Two PVE-only backup exports are canonical:

- `PVE_Restic_Backup/.data` -> `/mnt/nas-pve-restic-backup` on `192.168.1.68`; repository subdirectory `pve-restic-backup`.
- `Rotom_VM_Backup/.data` -> PVE storage `nas-rotom-vm-backup` at `/mnt/pve/nas-rotom-vm-backup`.

`showmount` verification after cleanup showed those canonical PVE backup exports; the superseded `Proxmox_Restic_Backup` and `Rotom_Proxmox_Backup` export names were not present. Guest `Rotom_Restic_Backup/.data` remains separate and unchanged.

### Post-reboot verification

A controlled physical PVE reboot produced a new boot ID. `pve-cluster`, `pvedaemon`, and `pveproxy` returned active, VMID 100 auto-started, Rotom SSH returned, and final PVE/Rotom acceptance passed with zero failed systemd units. qBittorrentVPN `wg0` was `10.2.0.2/32`; genuine NAS mounts and monitoring endpoints passed.

## 2A. Pre-Migration Bare-Metal Host Identity — Historical Baseline

| Item | Observed value |
| --- | --- |
| Static hostname | `rotom` |
| Pretty hostname | `Rotom` |
| `hostname -f` | `rotom` |
| DNS domain name | none returned |
| Operating system | Linux Mint 22.3 |
| Kernel | Linux 7.0.0-31-generic |
| Architecture | x86-64 |

`/etc/hostname` contains `rotom`.

`/etc/hosts` maps:

- `127.0.0.1` → `localhost`
- `127.0.1.1` → `rotom`

The host does not report `rotom.casa` as its local FQDN or DNS suffix. `rotom.casa` is the Project's service/domain namespace rather than the host's configured local hostname.

---

## 3. Pre-Migration Physical and Host Network Interfaces

### Primary LAN interface

| Item | Observed value |
| --- | --- |
| Interface | `eno1` |
| Alternate name | `enp0s31f6` |
| Link state | UP |
| IPv4 address | `192.168.1.69/24` |
| IPv4 assignment | DHCP / NetworkManager `ipv4.method=auto` |
| DHCP server | `192.168.1.1` |
| DHCP lease time | 86400 seconds (24 hours) |
| LAN subnet | `192.168.1.0/24` |
| Broadcast | `192.168.1.255` |
| Default IPv4 gateway | `192.168.1.1` |
| DNS supplied by DHCP | `192.168.1.1` |
| DHCP domain | `localdomain` |
| Route metric | 100 |

Rotom is configured as a DHCP client rather than with a static IPv4 address in NetworkManager. JAR-22 user verification in UniFi confirmed that `192.168.1.69` is reserved/fixed to Rotom MAC `1c:69:7a:0e:f2:f7`. The reservation was not changed during the audit. The same UniFi check confirmed proposed Proxmox management address `192.168.1.68` is open/available.


### JAR-22 address-preservation verification — 2026-09-22

The pre-migration audit reverified `eno1` at `192.168.1.69/24` and captured MAC `1c:69:7a:0e:f2:f7`. Host-side probes for `192.168.1.68` produced no ICMP response and a `FAILED` neighbor entry; `arping` and `nmap` were not installed and were not added. The user then confirmed directly in UniFi that `.68` is open/available and that `.69` is a fixed/reserved address bound to Rotom's MAC above. These confirmations satisfy the JAR-22 preservation record; they do not modify the reservation or establish unrelated router firewall/VLAN/forwarding state.


### JAR-23 boot-time network/NFS incident scope — 2026-09-25

JAR-23 did not change Rotom's LAN address, DHCP reservation, gateway, DNS configuration, Cloudflare DDNS settings, Nginx Proxy Manager routes, firewall policy, or published ports. The September 23 reboot sequence did show transient DNS/NFS readiness failures while Docker was restoring containers. `NetworkManager-wait-online.service` was enabled and completed successfully, and `network-online.target` became active, but that did not guarantee the UNAS NFS exports were ready for immediate Docker bind-mount use. The resulting failure was therefore treated as an application-storage startup-order problem rather than a persistent IP/DNS configuration defect.

The `rotom-nas-docker-recovery.service` waits until after `docker.service` and `network-online.target`, then actively verifies the Downloads/Media/Game NFS mounts and currently starts the unused Media bindfs unit before recovering only containers whose stopped state carries a NAS/mount startup-failure signature. JAR-81 proposes retiring that view and branch separately. It changes no DNS or routing behavior. Manual validation passed; reboot validation is deferred.

### JAR-24 backup-network scope verification — 2026-09-25

JAR-24 changed no LAN addressing, DHCP reservation, gateway, DNS, Cloudflare DDNS, Nginx Proxy Manager route, firewall rule, published port, or Docker network. The backup preflight verified `/mnt/nas-rotom-backup` as a live NFSv3 mount from the existing UNAS `Rotom_Home_Server_Backup/.data` export before the final recovery point was created. Restic then completed against that existing repository without any network-topology change.

The final snapshot `fbe1838e` at `2026-09-25 18:14:22 PDT` is a host backup only. `/mnt` remains intentionally excluded, so NAS media, Downloads torrents, Game/Media libraries, Shared Drive, service shares, and the backup repository itself remain outside the host snapshot even though network access to the backup export was required to create and verify it.


### JAR-25 Shared Drive imaging-path verification — 2026-09-25

JAR-25 made no LAN-address, DHCP-reservation, gateway, DNS, Cloudflare DDNS, Nginx Proxy Manager, firewall, published-port, or Docker-network change. The imaging preflight resolved `/mnt/nas-shared-drive` to the existing NFSv3 source `192.168.1.70:/volume/4d6b927b-3d90-4af3-9bd7-be668266998a/.srv/.unifi-drive/Shared_Drive/.data` and reported approximately 4.7T available on a 5.5T filesystem.

The full `250059350016`-byte raw NVMe image was streamed across that existing NFS path, then the complete stored image was reread across the same path for SHA-256 verification. Both hashes matched. This is useful sustained-transfer evidence for the existing Rotom-to-UNAS path at the time of JAR-25; it does not change or independently characterize switch, VLAN, router, or WAN policy. The image and Restic repository remain on the same physical UNAS appliance and therefore share that hardware failure domain.

### Wi-Fi

`wlp0s20f3` exists but was disconnected and had no assigned address at audit time.

### Loopback

`lo` provides `127.0.0.1/8` and `::1/128`.

---

## 4. Pre-Migration IPv6 State

`eno1` has IPv6 Unique Local Addresses in:

`fda6:1b70:534e:4795::/64`

and a normal `fe80::/64` link-local address.

No IPv6 default route was present. IPv6 forwarding is disabled:

`net.ipv6.conf.all.forwarding = 0`

An additional route exists for `fd85:775d:9e2a::/64` through a link-local next hop on `eno1`. Its purpose is not established by the audits.

The audits do not establish public Internet IPv6 connectivity for Rotom.

---

## 5. Pre-Migration Routing and Forwarding

### IPv4 default route

```text
default via 192.168.1.1 dev eno1 proto dhcp src 192.168.1.69 metric 100
```

### Directly connected LAN

`192.168.1.0/24` is directly connected through `eno1`.

### Kernel forwarding

| Setting | Value |
| --- | --- |
| IPv4 forwarding | enabled (`1`) |
| IPv6 forwarding | disabled (`0`) |

IPv4 forwarding is used by Docker's bridge/NAT networking.

Docker-created bridge routes occupy `172.17.0.0/16` through `172.31.0.0/16`, with the actual network assignments documented below.

---

## 6. Pre-Migration DNS Resolver Architecture

Rotom uses `systemd-resolved`.

`/etc/resolv.conf` points to:

`/run/systemd/resolve/stub-resolv.conf`

Local applications use the stub resolver at `127.0.0.53`.

### Upstream resolver for `eno1`

| Item | Value |
| --- | --- |
| Current DNS server | `192.168.1.1` |
| Configured DNS server | `192.168.1.1` |
| Search/domain suffix | `localdomain` |
| DNS default route | enabled |
| LLMNR | disabled |
| systemd-resolved mDNS | disabled |
| DNS-over-TLS | disabled |
| DNSSEC | unsupported/not enabled |

The host therefore sends ordinary DNS requests to the LAN gateway/router at `192.168.1.1`. The router's upstream DNS provider is not established by these audits.

---

## 7. Pre-Migration Host Network Management

| Component | State |
| --- | --- |
| NetworkManager | active |
| systemd-networkd | inactive |
| systemd-resolved | active |

The primary connection is NetworkManager connection `Wired connection 1` on `eno1`.

Docker bridge devices appear to NetworkManager as externally managed. Docker veth interfaces appear as unmanaged.

---

## 8. Pre-Migration Docker Network Architecture

### Observed Pre-Migration Docker Networks

The detailed subnet/gateway/runtime-IP values below come from the full 2026-09-19 Docker-network audit. JAR-22 reverified the **currently present network names** on 2026-09-22 and confirmed the historical `jaredwinescom_default` network is now absent; the other listed network names remain present. Unless separately refreshed, their subnet/gateway/runtime-IP values remain dated 2026-09-19 observations. All listed user-defined bridge networks were `Internal=false` and `Attachable=false` in that full audit.

| Docker network | Driver | Subnet | Gateway | Current container use |
| --- | --- | --- | --- | --- |
| `bridge` | bridge | `172.17.0.0/16` | `172.17.0.1` | no current container assignment |
| `qbittorrentvpn_default` | bridge | `172.19.0.0/16` | `172.19.0.1` | qBittorrentVPN (`172.19.0.2`) |
| `jellyfin_default` | bridge | `172.20.0.0/16` | `172.20.0.1` | Jellyfin (`172.20.0.2`) |
| `sonarr_default` | bridge | `172.21.0.0/16` | `172.21.0.1` | Sonarr (`172.21.0.2`) |
| `prowlarr_default` | bridge | `172.22.0.0/16` | `172.22.0.1` | Prowlarr (`172.22.0.2`) |
| `arcane_default` | bridge | `172.23.0.0/16` | `172.23.0.1` | Arcane (`172.23.0.2`) |
| `homepage` | bridge | `172.24.0.0/16` | `172.24.0.1` | Glances (`172.24.0.2`), Homepage (`172.24.0.3`) |
| `alohamillworkscom_default` | bridge | `172.26.0.0/16` | `172.26.0.1` | Aloha Millworks (`172.26.0.2`) |
| `palworld-jared_default` | bridge | `172.27.0.0/16` | `172.27.0.1` | Jared Palworld (`172.27.0.2`) |
| `palworld-fran_default` | bridge | `172.28.0.0/16` | `172.28.0.1` | Fran Palworld (`172.28.0.2`) |
| `gamarr_default` | bridge | `172.29.0.0/16` | `172.29.0.1` | Gamarr (`172.29.0.2`) |
| `radarr_default` | bridge | `172.18.0.0/16` | `172.18.0.1` | Radarr (`172.18.0.2`) |
| `host` | host | n/a | n/a | Home Assistant, Homebridge, Cloudflare DDNS, Nginx Proxy Manager |
| `none` | null | n/a | n/a | none |

### Observed Pre-Migration Bridge-Container Assignments

| Container | Docker network | Runtime IPv4 | Host mapping |
| --- | --- | --- | --- |
| `qbittorrentvpn` | `qbittorrentvpn_default` | `172.19.0.2` | TCP 8080 → 8080 |
| `jellyfin` | `jellyfin_default` | `172.20.0.2` | TCP 8096 → 8096 |
| `sonarr` | `sonarr_default` | `172.21.0.2` | TCP 8989 → 8989 |
| `prowlarr` | `prowlarr_default` | `172.22.0.2` | TCP 9696 → 9696 |
| `arcane` | `arcane_default` | `172.23.0.2` | TCP 3552 → 3552 |
| `glances` | `homepage` | `172.24.0.2` | `127.0.0.1:61208` → 61208 |
| `homepage` | `homepage` | `172.24.0.3` | TCP 3001 → 3000 |
| `alohamillworks.com` | `alohamillworkscom_default` | `172.26.0.2` | TCP 7778 → 80 |
| `palworld-server-jared` | `palworld-jared_default` | `172.27.0.2` | UDP 8211, UDP 27015 |
| `palworld-server-fran` | `palworld-fran_default` | `172.28.0.2` | UDP 8212, UDP 27016 |
| `gamarr` | `gamarr_default` | `172.29.0.2` | TCP 6767 → 6767 |
| `radarr` | `radarr_default` | `172.18.0.2` | TCP 7878 → 7878 |

JAR-22 verified `/home/web/docker/jaredwines.com/compose.yaml` remains present, but there is currently no `jaredwines.com` container object and no `jaredwinescom_default` Docker network. The historical 2026-09-19 `172.25.0.0/16` Jared Wines network allocation remains useful historical evidence only, not current state. Runtime addresses are observations, not guaranteed static assignments.

The default Docker bridge has IP masquerading enabled and binds published ports to `0.0.0.0` by default. Docker adds NAT/forwarding rules for published bridge-container ports.

### Gamarr application connectivity — retired by JAR-77

This is historical deployment evidence. JAR-77 removed the container, private network, TCP `6767` listener, and NPM proxy-host ID `19`; the host is marked deleted and its generated configuration is absent. The shared wildcard certificate ID `24` remains active, and wildcard DNS resolution may continue without a route.


Runtime container IPv4 addresses are observations, not guaranteed static assignments. The audits did not inspect Compose IPAM/static-address declarations.

### Shared bridge networks

`homepage` is the only observed user-defined bridge currently shared by more than one running container:

- `glances` — `172.24.0.2`
- `homepage` — `172.24.0.3`

All other running bridge-mode application containers currently use separate Compose-created networks.

---

## 9. Pre-Migration Host-Networked Containers

The following containers use Docker `host` networking and therefore share Rotom's host network namespace:

- `nginx-proxy-manager`
- `home-assistant`
- `homebridge`
- `cloudflare-ddns`

On 2026-09-19 the Homebridge container name was corrected from historical `homebrige` to `homebridge`. Its host-network architecture and documented listeners were unchanged. The post-change whole-home scan reported no remaining `homebrige` references.

These containers do not receive bridge IP addresses and do not require Docker `PortBindings` for host listeners.

### Verified listeners from host-mode services

| Port | Protocol | Observed process/service |
| --- | --- | --- |
| 80 | TCP | Nginx Proxy Manager nginx |
| 81 | TCP | Nginx Proxy Manager nginx admin listener |
| 443 | TCP | Nginx Proxy Manager nginx |
| 8123 | TCP | Home Assistant Python process |
| 8581 | TCP | Homebridge service |
| 5353 | UDP | Homebridge/Avahi mDNS activity |

Additional Home Assistant/Homebridge dynamic/discovery listeners were present, including mDNS/SSDP and dynamic TCP/UDP sockets. Their architectural purpose should not be assigned solely from socket ownership unless needed for future troubleshooting.

---

## 10. Current Nginx Proxy Manager

### Nginx Proxy Manager runtime model

| Item | Value |
| --- | --- |
| Container | `nginx-proxy-manager` |
| Image | `jc21/nginx-proxy-manager:latest` |
| Docker network mode | `host` |
| Image-exposed ports | TCP 80, 81, 443 |
| Actual host listeners | TCP 80, 81, 443 |

Persistent bind mounts:

| Host path | Container path | Mode |
| --- | --- | --- |
| `/srv/rotom/appdata/infra/nginx-proxy-manager/data` | `/data` | read/write |
| `/srv/rotom/appdata/infra/nginx-proxy-manager/letsencrypt` | `/etc/letsencrypt` | read/write |

No NPM secrets or certificate/private-key contents are recorded here.

### Observed Pre-Migration NPM Proxy-Host Database Records

No redirect hosts, stream hosts, or dead hosts were present in the earlier audited NPM database. The table below records proxy-host database state, with later generated-config qualification where available.

| NPM domain | Forward scheme | Forward target | State | Notes |
| --- | --- | --- | --- | --- |
| `nginx.smart-hub.local` | HTTP | `192.168.68.69:81` | DB enabled; generated config absent | target is outside current `192.168.1.0/24` LAN |
| `portainer.smart-hub.local` | HTTPS | `192.168.68.69:9443` | DB enabled; generated config absent | target is outside current LAN; no running Portainer container observed |
| `home-assistant.rotom.casa` | HTTP | `192.168.1.69:8123` | enabled | current Rotom host |
| `hb.smart-hub.local` | HTTP | `192.168.68.69:8581` | DB enabled; generated config absent | target is outside current LAN |
| `homebridge.rotom.casa` | HTTP | `192.168.1.69:8581` | enabled | current Rotom host |
| `nginx.rotom.casa` | HTTP | `192.168.1.69:81` | enabled | NPM admin listener |
| `arcane.rotom.casa` | HTTP | `192.168.1.69:3552` | enabled | Arcane host-published port |
| `jaredwines.com` | HTTP | `192.168.1.69:7777` | enabled | site container not running at audit time |
| `rotom.casa` | HTTP | `192.168.1.69:3001` | enabled; September 16 chat verification | Homepage canonical apex URL; replaces `homepage.rotom.casa` |
| `palworld.smart-hub.xyz` | HTTP | `192.168.1.69:8211` | DB enabled; generated config absent | current Palworld 8211 listener is UDP, not HTTP/TCP; verify intent |
| `alohamillworks.com` | HTTP | `192.168.1.69:7778` | enabled | website host-published port |
| `olivetin.smart-hub.xyz` | HTTP | `192.168.1.69:1337` | DB enabled; generated config absent | no OliveTin container/listener observed |
| `paltools.smart-hub.xyz` | HTTP | `192.168.1.69:14100` | DB enabled; generated config absent | no listener on 14100 observed |
| `qBittorrent.rotom.casa` | HTTP | `192.168.1.69:8080` | enabled | qBittorrentVPN Web UI |
| `radarr.rotom.casa` | HTTP | `192.168.1.69:7878` | enabled | Radarr |
| `sonarr.rotom.casa` | HTTP | `192.168.1.69:8989` | enabled | Sonarr |
| `jellyfin.rotom.casa` | HTTP | `192.168.1.69:8096` | enabled | Jellyfin |
| `prowlarr.rotom.casa` | HTTP | `192.168.1.69:9696` | enabled | Prowlarr; local and proxied HTTP both remained `302` after the 2026-09-19 migration |
| `gamarr.rotom.casa` | HTTP | `192.168.1.69:6767` | enabled at the historical audit | Historical Gamarr route; JAR-77 later marked proxy-host ID `19` deleted and removed its generated configuration |

The legacy Smart Hub/Portainer/OliveTin/PalTools rows above remain `enabled=1` in the NPM database, but their IDs are absent from the generated proxy-host configuration set under current Platform NPM appdata. Treat those rows as stale/unresolved database records rather than active generated routes until administrator intent is reviewed. Gamarr's route is retired: JAR-77 marked proxy-host ID `19` deleted and its generated configuration is absent; JAR-82 did not re-establish it. JAR-82 verified that the remaining active Rotom/application routes regenerated from the preserved database after cutover.

The database output did not expose a current WebSocket-support field in the selected schema output, so WebSocket behavior is not documented here.

### NPM routing model

For current Rotom-hosted routes, NPM generally proxies back to Rotom's LAN address and host-published application port rather than joining each application's Docker bridge directly:

```text
Client
  ↓
Rotom TCP 80/443
  ↓
Nginx Proxy Manager (host network)
  ↓
192.168.1.69:<host-published application port>
  ↓
Docker DNAT / host-networked service
  ↓
Application
```

This is a verified routing pattern for the current `rotom.casa` application proxy-host definitions.

### TLS and access-list metadata

All active `rotom.casa` application proxy hosts observed in the audit use NPM certificate ID `24`, force SSL, and have no access list assigned. A direct 2026-09-19 TLS inspection of `rotom.casa` confirmed the served wildcard certificate below, issued by Let's Encrypt (`YE2`):

| Certificate | Provider | Domains | Observed expiry |
| --- | --- | --- | --- |
| ID 24 | Let's Encrypt | `*.rotom.casa`, `rotom.casa` | 2026-12-10 06:41:00 |

Additional observed certificates include current Let's Encrypt certificates for `jaredwines.com`, `alohamillworks.com`, and `*.smart-hub.xyz` / `smart-hub.xyz`. NPM also retains an older `jaredwines.com` certificate record that expired in 2025; the active proxy host references the newer certificate ID `14`.

For the audited proxy hosts, `http2_support=0`, `hsts_enabled=0`, and `hsts_subdomains=0`. No NPM access list is assigned (`access_list_id=0`) to the listed proxy hosts. This records current state only and does not imply a recommendation.

---

## 11. Pre-Migration Cloudflare DDNS

### Cloudflare DDNS runtime model

| Item | Value |
| --- | --- |
| Container | `cloudflare-ddns` |
| Image | `favonia/cloudflare-ddns:1` |
| Docker network mode | `host` |
| Published/exposed ports | none |

### Secret handling

The Cloudflare API token is supplied through a read-only Docker secret-style bind mount:

```text
/home/infra/docker/cloudflare-ddns/secrets/cloudflare_api_token.txt
    → /run/secrets/cloudflare_api_token
```

The token value is intentionally not recorded.

### Non-secret DDNS configuration

| Setting | Observed value |
| --- | --- |
| Managed domain setting | `DOMAINS=rotom.casa` |
| Cloudflare proxy mode | `PROXIED=false` |
| IPv6 provider | `IP6_PROVIDER=none` |
| Update interval | `@every 1m` |
| Update on container start | `true` |

The audit therefore shows that this DDNS container is configured to manage `rotom.casa`, does not enable the Cloudflare proxy for that managed record, does not update IPv6 through its configured provider, checks every minute, and updates on startup.

Individual service subdomains are not listed in the DDNS container's `DOMAINS` value. DNS lookup results below show the observed service records separately.

---

## 12. Pre-Migration Domain and DNS Architecture

### Confirmed split/local DNS behavior

The third audit compared Rotom's normal resolver with Cloudflare (`1.1.1.1`), Google (`8.8.8.8`), and Cloudflare's authoritative nameserver. The results establish two different DNS views:

| Query view | `rotom.casa` A result |
| --- | --- |
| Rotom/local resolver | `192.168.1.69` |
| Cloudflare public resolver | `68.8.40.225` |
| Google public resolver | `68.8.40.225` |
| Cloudflare authoritative nameserver | `68.8.40.225` |

Rotom's current externally observed IPv4 was also `68.8.40.225`. Therefore the Cloudflare authoritative A record matched the server's WAN/public IPv4 at audit time, while the LAN resolver at `192.168.1.1` provided a private local answer. This confirms split-horizon/local-DNS behavior in the observed environment. The audit does not identify the exact UniFi setting implementing that local override.

### Homepage cutover — September 16, 2026

“Change Homepage URL” supersedes the earlier Homepage hostname in the audit. The canonical URL is `https://rotom.casa`. Historical source output and the later verified generated NPM configuration show `server_name rotom.casa;`, so the Homepage proxy no longer includes `homepage.rotom.casa`. Current NPM appdata is under the Platform domain; the former Infra path is historical.

The supplied command `curl -Ik --resolve rotom.casa:443:127.0.0.1 https://rotom.casa/` returned `HTTP/1.1 200 OK` and `X-Served-By: rotom.casa`. This verifies local TLS routing with the correct SNI and a successful application response; because `-k` skips certificate verification, it does not independently prove certificate trust or external reachability. Testing `https://127.0.0.1` with only an HTTP Host header does not provide the same TLS SNI and previously produced an unrecognized-name error.

Homepage's selected identity is title `Rotom`, description `Home automation and server management`, and allowed host `rotom.casa`. The final output verifies the route and that Homepage accepts the apex; it did not separately print the title, description, or exact allowed-host environment value. Container names, data paths, and port `3001` remain unchanged.

### `rotom.casa` service records

The audited `rotom.casa` service names resolve as CNAMEs to `rotom.casa`. Resolution alone does not establish an explicit DNS record for each name: the September 16 chat demonstrated wildcard behavior using `definitely-not-a-real-host.rotom.casa`, which resolved through `rotom.casa` to the then-observed public IPv4 `68.8.40.225`. Their baseline resolution differs by viewpoint:

```text
LAN / Rotom resolver:
service.rotom.casa -> rotom.casa -> 192.168.1.69

Public / authoritative DNS:
service.rotom.casa -> rotom.casa -> 68.8.40.225
```

Names confirmed to resolve in the baseline include (not a list of current NPM routes). A later LAN lookup also verified `gamarr.rotom.casa` resolving through `rotom.casa` to `192.168.1.69`:

- `arcane.rotom.casa`
- `home-assistant.rotom.casa`
- `homebridge.rotom.casa`
- `gamarr.rotom.casa`
- `homepage.rotom.casa` — still resolved in September 16 testing, but its Homepage proxy route has been retired
- `jellyfin.rotom.casa`
- `nginx.rotom.casa`
- `prowlarr.rotom.casa`
- `qBittorrent.rotom.casa`
- `radarr.rotom.casa`
- `sonarr.rotom.casa`

Whether Cloudflare also retains an explicit `homepage.rotom.casa` record is unverified. Wildcard resolution is enough to explain the old hostname resolving; it does not mean NPM still routes it to Homepage. Keep the wildcard intact unless its dependencies are separately reviewed. No DNS records were changed by this documentation update.

No public AAAA record was observed for `rotom.casa`; Cloudflare DDNS is configured with `IP6_PROVIDER=none`.

### Public website names

At audit time:

| Domain | Public A | NPM forward target |
| --- | --- | --- |
| `alohamillworks.com` | `68.8.40.225` | `192.168.1.69:7778` |
| `jaredwines.com` | `68.8.40.225` | `192.168.1.69:7777` |

The audits still do not capture the UniFi WAN/NAT rule that would carry incoming Internet traffic from `68.8.40.225` to Rotom.

### Other NPM names

The following NPM proxy-host names did not resolve through the tested DNS paths:

- `nginx.smart-hub.local`
- `portainer.smart-hub.local`
- `hb.smart-hub.local`
- `olivetin.smart-hub.xyz`
- `paltools.smart-hub.xyz`
- `palworld.smart-hub.xyz`

The `.local` proxy hosts point to `192.168.68.69`. Rotom routes that address through gateway `192.168.1.1`, but a one-packet ICMP test received no reply during the audit. This establishes that the old target was not ping-reachable at that moment; it does not alone prove the host no longer exists or that TCP services are unreachable.

---

## 13. Pre-Migration Port Exposure and Listening Ports

### Core host / host-network listeners

| Bind | Protocol | Port | Observed owner/purpose |
| --- | --- | ---: | --- |
| all IPv4/IPv6 | TCP | 22 | SSH |
| all IPv4/IPv6 | TCP | 80 | Nginx Proxy Manager |
| all IPv4/IPv6 | TCP | 81 | Nginx Proxy Manager admin |
| all IPv4/IPv6 | TCP | 443 | Nginx Proxy Manager TLS |
| all IPv4/IPv6 | TCP | 8123 | Home Assistant |
| all | TCP | 8581 | Homebridge |
| all | TCP | 3389 | XRDP |
| all IPv4 | TCP | 8787 | Python process; firewall comment identifies Homepage backup status API |
| loopback | TCP | 61208 | Glances published via Docker |
| loopback | TCP | 18554 | `go2rtc` |
| all | TCP | 18555 | `go2rtc` |
| all IPv4/IPv6 | TCP | 111 | rpcbind |
| all IPv4/IPv6 | UDP | 111 | rpcbind |
| all/multicast | UDP | 5353 | mDNS / Avahi / Homebridge / Home Assistant activity |
| all | UDP | 1900 | SSDP-related Python process |

There are additional dynamic RPC, discovery, Homebridge accessory, and Home Assistant sockets. They are not assigned a permanent architectural role here unless the audit identifies one directly.

### Docker-published application listeners

| Host port | Protocol | Service |
| ---: | --- | --- |
| 3001 | TCP | Homepage |
| 3552 | TCP | Arcane |
| 6767 | TCP | Gamarr |
| 7777 | TCP | `jaredwines.com` configured, but no listener while container stopped |
| 7778 | TCP | `alohamillworks.com` |
| 7878 | TCP | Radarr |
| 8080 | TCP | qBittorrentVPN |
| 8096 | TCP | Jellyfin |
| 8211 | UDP | Palworld Jared |
| 8212 | UDP | Palworld Fran |
| 8989 | TCP | Sonarr |
| 9696 | TCP | Prowlarr |
| 27015 | UDP | Palworld Jared |
| 27016 | UDP | Palworld Fran |

---

## 14. Pre-Migration Firewall and Packet Filtering

### Firewall implementation

UFW is active. `firewalld` is not installed.

UFW defaults:

| Direction | Default policy |
| --- | --- |
| Incoming | deny |
| Outgoing | allow |
| Routed | deny |

Logging is enabled at `low` level.

The host uses the nftables backend through `iptables-nft`. Docker also installs its own filter and NAT chains.

### UFW rules observed

#### LAN-restricted IPv4 rules

The following TCP ports are explicitly allowed from `192.168.1.0/24`:

- 22 — SSH
- 81 — NPM admin
- 1337
- 3001 — Homepage
- 8123 — Home Assistant
- 8581 — Homebridge
- 9443
- 21064

A fresh 2026-09-19 UFW audit verifies TCP `8787` allowed specifically from the current Homepage bridge `172.24.0.0/16`, with comment `Homepage backup status API`. A live Python listener is present on `0.0.0.0:8787`. This supersedes the historical `172.18.0.0/16` rule record.

#### Broad IPv4/IPv6 allow rules

The following are explicitly allowed from anywhere by UFW:

- TCP 80
- TCP 443
- TCP 7777 (`jaredwines.com`)
- TCP 7778 (`alohamillworks.com`)
- port 3552 without protocol restriction in the UFW command representation
- port 27016 without protocol restriction
- port 8212 without protocol restriction
- port 22 without protocol restriction
- port 3389 without protocol restriction

Equivalent IPv6 allow rules are present for these broadly allowed entries.

Because there is also an earlier LAN-only rule for TCP 22, the later broad port-22 rule makes SSH permitted from any IPv4/IPv6 source at the host firewall layer. This is an observed rule interaction and should be verified against intended policy before any change is made.

Likewise, XRDP port 3389 is currently broadly permitted by UFW.

### Docker and UFW interaction

Docker publishes bridge-container ports through its own DNAT and forwarding chains. The following DNAT rules are from the **2026-09-14 captured NAT table**. They are historical packet-filter evidence, not a current DNAT map. In particular, the Homepage and Glances `172.18.0.x` destinations below are superseded by the 2026-09-18 live verification of the `homepage` network as `172.24.0.0/16` with Glances at `172.24.0.2` and Homepage at `172.24.0.3`; re-read the live NAT rules before using those older destinations operationally:

- 8212/UDP → `172.25.0.2:8212`
- 27016/UDP → `172.25.0.2:27016`
- 8211/UDP → `172.26.0.2:8211`
- 27015/UDP → `172.26.0.2:27015`
- 7778/TCP → `172.23.0.2:80`
- 3001/TCP → `172.18.0.2:3000`
- 8080/TCP → `172.27.0.2:8080`
- 7878/TCP → `172.28.0.2:7878`
- 8989/TCP → `172.29.0.2:8989`
- 9696/TCP → `172.31.0.2:9696`
- 3552/TCP → `172.24.0.2:3552`
- 8096/TCP → `172.30.0.2:8096`
- `127.0.0.1:61208/TCP` → `172.18.0.3:61208`

Docker's `DOCKER-USER` chain was empty in the captured ruleset.

This distinction matters: host INPUT/UFW rules and Docker-published bridge-container forwarding are separate packet paths. A UFW rule list alone is therefore not sufficient to determine exposure of Docker-published ports. Actual Internet reachability still also depends on UniFi/router forwarding and upstream network policy.

### IPv6 packet filtering

The IPv6 INPUT and FORWARD policies are DROP and OUTPUT is ACCEPT. Docker had no IPv6 NAT destination rules for the bridge applications because the Docker networks were observed with IPv6 disabled.

---

## 15. Pre-Migration Verified Traffic Flows

### Host outbound path

```text
Rotom / containers
    ↓
eno1 — 192.168.1.69/24
    ↓
Default gateway — 192.168.1.1
    ↓
Upstream network
```

### Reverse-proxied Rotom service path

```text
Client resolving a rotom.casa service name
    ↓
service-name CNAME → rotom.casa
    ↓
rotom.casa → 192.168.1.69 in the captured DNS view
    ↓
Rotom TCP 80/443
    ↓
Nginx Proxy Manager (host network)
    ↓
192.168.1.69:<service host port>
    ↓
Docker DNAT or host-network listener
    ↓
Application
```

### Website path — verified and unverified portions

The audits establish:

```text
alohamillworks.com / jaredwines.com
    ↓
A = 68.8.40.225 in Rotom's DNS view
    ↓
[UniFi/WAN forwarding NOT CAPTURED]
    ↓
Nginx Proxy Manager
    ↓
192.168.1.69:7778 or :7777
    ↓
website container
```

The WAN/router forwarding hop remains to be documented from UniFi.

---

## 16. Preserved Pre-Migration NAS / NFS Networking

Rotom and the UniFi UNAS NFS server share the `192.168.1.0/24` LAN: Rotom is `192.168.1.69`, UNAS is `192.168.1.70`. JAR-6 did not change routing, DNS, firewall policy, Docker networking, or reverse-proxy behavior; it expanded the set of persistent NFS dependencies.

Pre-migration NAS mount classes were:

| Mount class | Rotom paths | NFS behavior |
|---|---|---|
| Active application storage | `/mnt/nas-media`, `/mnt/nas-game` | NFSv3 `sec=sys`; existing Media/Game Docker contracts |
| JAR-6 service boundaries | `/mnt/nas-infra`, `/mnt/nas-smarthome`, `/mnt/nas-documents`, `/mnt/nas-downloads`, `/mnt/nas-web`, `/mnt/nas-filesync`, `/mnt/nas-apps`, `/mnt/nas-auth` | NFSv3 `sec=sys`; no Docker binds/workload data yet |
| Backup / shared storage | `/mnt/nas-rotom-backup`, `/mnt/nas-shared-drive` | NFSv3; existing contracts unchanged |

At the pre-migration baseline, all twelve paths were fstab-backed systemd automounts using `_netdev,nofail,x-systemd.automount,x-systemd.device-timeout=30s`. JAR-29 later established that systemd 257 ignores `x-systemd.device-timeout` for these NFS entries and therefore uses `x-systemd.mount-timeout=30s` in the current VM. A normal bare-metal Rotom reboot on 2026-09-21 had verified every new JAR-6 automount returned; the current VM has separately passed normal-reboot and NAS-unavailable-boot tests.

The network dependency is therefore:

```text
Rotom 192.168.1.69
   ↓ same LAN / NFSv3
UniFi UNAS 192.168.1.70
   ├─ Media / Game active application shares
   ├─ Infra / Smarthome / Documents / Downloads
   ├─ Web / Filesync / Apps / Auth service boundaries
   ├─ Backup
   └─ Shared Drive
```

### NAS access model — preserved JAR-6 pre-migration state

NFS reachability does not imply Unix access. UNAS `rpc.mountd --manage-gids` was observed rebuilding supplementary groups server-side while retaining the request primary GID, so the dedicated primary service GIDs are the tested ordinary-access contract.

| Account | UID:primary GID | Mount | Root metadata |
|---|---|---|---|
| media | `127:5000` | `/mnt/nas-media` | `988:5000` mode `2770` |
| game | `995:5001` | `/mnt/nas-game` | `988:5001` mode `2770` |
| infra | `997:5002` | `/mnt/nas-infra` | `988:5002` mode `2770` |
| smarthome | `126:5003` | `/mnt/nas-smarthome` | `988:5003` mode `2770` |
| documents | `900:5004` | `/mnt/nas-documents` | `988:5004` mode `2770` |
| downloads | `901:5005` | `/mnt/nas-downloads` | `988:5005` mode `2770` |
| web | `902:5006` | `/mnt/nas-web` | `988:5006` mode `2770` |
| filesync | `903:5007` | `/mnt/nas-filesync` | `988:5007` mode `2770` |
| apps | `904:5008` | `/mnt/nas-apps` | `988:5008` mode `2770` |
| auth | `905:5009` | `/mnt/nas-auth` | `988:5009` mode `2770` |

Functional testing before and after the Rotom reboot verified each intended account could traverse/write its own JAR-6 share while an unrelated service account could not traverse or write. This is an ordinary filesystem boundary only; sudo/root and Docker-daemon administrators remain privileged.

Backup remains `988:988` mode `0700` and Shared Drive remains `988:988` mode `0770`. Numeric UID/GID `988` maps differently by host name database; preserve numeric ownership rather than interpreting Rotom's local `fwupd-refresh` name as the NAS owner identity.

### JAR-9 rollback / unused path — 2026-09-21

`/mnt/nas-game-servers` remained absent from fstab and unmounted at this checkpoint. During the later JAR-6 UNAS audit the former `Game_Servers/.data` path was observed absent rather than merely unused. JAR-6 did not delete or recreate it. The then-active Game dependency was `/mnt/nas-game` → `Game/.data`; JAR-88 later replaced it with `/mnt/nas-gameserver` → `Gameserver/.data`.

### Network-side implications of the JAR-6 service shares

The eight new JAR-6 mounts do not add listening ports, Docker networks, NPM routes, Cloudflare records, or public exposure. They also do not create a global Docker→NAS startup dependency. Future applications attached to a service share should receive a separate storage/dependency review; the existence of a mounted share alone is not authorization to add a route or bind.

---

## 17. Pre-Migration Network Dependencies

Important dependencies established by the audits:

- Rotom's primary IPv4 connectivity depends on `eno1` and LAN gateway/DHCP/DNS server `192.168.1.1`.
- Host DNS resolution depends on `systemd-resolved` and the local resolver at `192.168.1.1`.
- Public DNS for `rotom.casa` depends on Cloudflare authoritative DNS and currently resolves to `68.8.40.225`.
- Local service resolution uses a private DNS view returning `192.168.1.69`.
- Nginx Proxy Manager, Home Assistant, Homebridge, and Cloudflare DDNS depend directly on the host network namespace.
- Most other applications depend on Docker bridge networks plus Docker NAT for host-published ports.
- NPM currently proxies most active `rotom.casa` applications through Rotom's own LAN address and published host ports.
- Cloudflare DDNS depends on a read-only token file under `/home/infra/docker/cloudflare-ddns/secrets/` and is configured to manage `rotom.casa` every minute.
- At the pre-migration baseline, all twelve NAS mountpoints depended on NFS connectivity from Rotom to `192.168.1.70`. qBittorrentVPN uses only the Media and Game torrent trees, and Gamarr uses the full Game share; the eight JAR-6 service shares are not Docker-bound. Docker as a whole remains independent of NAS availability.
- External website and reverse-proxy reachability depends on UniFi/router WAN forwarding that has not yet been captured.

---

## 18. Historical Pre-Migration Architecture Snapshot

This section is intentionally concise because the detailed evidence is preserved in sections 2A–17. At the 2026-09-19 bare-metal network snapshot, Rotom used `eno1` at `192.168.1.69/24`, NetworkManager/systemd-resolved, host UFW, host-network NPM/Home Assistant/Homebridge/Cloudflare DDNS, and the then-current Docker bridge allocation spanning `172.18.0.0/16` through `172.29.0.0/16`. Split DNS returned the LAN address locally while public DNS returned the observed WAN IPv4 `68.8.40.225`. NFS storage reached the UNAS at `192.168.1.70`.

Later pre-migration work on 2026-09-21 changed service identities, storage GIDs, Downloads ownership, and related paths before the final JAR-22/JAR-25 preservation baseline. Those later changes are preserved in the dated sections of this file and in documents 04/07. None of this historical snapshot should override the current Proxmox/VM identity, JAR-31 Docker network map, current `downloaders` paths, current no-UFW guest state, or JAR-32/JAR-33 backup paths recorded in section 2.

## 19. Historical Consolidated Verification — 2026-09-19

- `eno1` remains `192.168.1.69/24` by DHCP with gateway/DNS `192.168.1.1`; public resolvers return `68.8.40.225` for `rotom.casa`. UniFi-side reservation, VLAN, NAT/forwarding, firewall, and local-DNS implementation remain uninspected.
- `eno1` reports `Supports Wake-on: pumbg` and `Wake-on: g`, verifying host/NIC magic-packet configuration. End-to-end delivery from another device while Rotom is off/suspended remains untested.
- NPM host-mode listeners on TCP 80/81/443 are active. The served `rotom.casa` certificate is the Let's Encrypt wildcard for `*.rotom.casa` and `rotom.casa`, valid 2026-09-11 through 2026-12-10. A real future renewal event remains unobserved.
- `gamarr.rotom.casa` is now a verified active proxy host to `192.168.1.69:6767`. NPM database row 19 is enabled, uses certificate ID `24`, forces SSL, has no access list, and the local HTTPS route probe returned HTTP `302`.
- Legacy Smart Hub/Portainer/OliveTin/PalTools records remain enabled in the NPM database but were absent from the inspected generated `proxy_host/*.conf` set; their intent remains unresolved and no deletion/disable action was taken.
- Fresh UFW output still shows default-deny incoming plus the previously documented broad allows, and `DOCKER-USER` remains empty. Public reachability still cannot be inferred without UniFi/WAN forwarding and router-firewall evidence.

## 20. Reconstruction-Critical Information

- Rotom LAN identity, addressing, gateway/DNS behavior, and host interface details are recorded in the current host/network sections above.
- The current Docker network and published-port maps are canonical in this file; service Compose definitions remain canonical in `02-Docker-Services.md`.
- Current Nginx Proxy Manager routes, `rotom.casa` relationships, and Cloudflare/DDNS behavior are recorded here without secret values.
- Router-side DHCP reservation, VLAN, WAN-forwarding, and router-firewall intent remain outside the verified host evidence unless explicitly documented.

## 21. Known Gaps / Needs Verification

Only unresolved **current** questions belong here. Dated UFW, `eno1`, old Docker-network, and bare-metal NFS observations remain preserved above as historical evidence, but they are not current Phase B gaps.

### UniFi / router layer — Verified by Jared confirmation

Current VM reservation and local-DNS identity are already verified: `192.168.1.69` is reserved to VM MAC `BC:24:11:97:10:47`, and the LAN resolver returns `rotom.casa -> 192.168.1.69`. Jared confirmed the required UniFi/controller inspection of reservations, VLANs, firewall/NAT/port-forwarding, and hairpin behavior completed. Detailed router-policy output was not supplied in this chat.

### Wake-on-LAN end-to-end trigger — Verified by Jared confirmation

Host-side WOL is current and persistent: Proxmox `nic0` supports magic-packet wake, reports `Wake-on: g`, PCI wake is enabled, and `/etc/network/interfaces` reasserts `wol g` on interface bring-up. Jared confirmed the end-to-end controlled wake test completed successfully. The sender transcript was not retained in this chat.

### Legacy NPM records — Reviewed by Jared confirmation

Historical NPM evidence retained legacy Smart Hub/Portainer/OliveTin/PalTools database rows that were `enabled=1` but absent from generated `proxy_host/*.conf` files. Jared confirmed the current-record review and ownership decision completed. The decision and any resulting record changes were not supplied in this chat, so none are inferred.

### Current exposure policy — Intentional trusted-LAN design

The Debian VM currently has **no UFW installation** per JAR-31; the broad UFW rules documented in section 14 belong to the retired bare-metal host. Jared accepted the current PVE-disabled-firewall and direct-Rotom-listener posture as intentional trusted-LAN exposure. No firewall hardening, listener restriction, router, VLAN, or port-forwarding change is planned from this decision. The 2026-09-27 and 2026-10-14 listener inventories remain the operational reference.

### qBittorrent live Libraries-NFS-loss behavior — Verified by Jared confirmation

Fail-closed recreation and the current Libraries guard are documented. Jared confirmed the separately planned non-disruptive test of an already-running qBittorrent container during Libraries-NFS loss completed successfully. Detailed runtime observations were not supplied in this chat.

### Resolved or intentional states — not gaps

- The retired Downloader and Media guest boundaries must not be used as current service guidance. The supported current qBittorrent/ROM boundary is `/mnt/nas-libraries`; the former Game guest mount was later replaced by `/mnt/nas-game-servers`. Historical boot-recovery observations remain diagnostic evidence only.
- `jaredwines.com` is intentionally undeployed; its Compose project is retained. Starting it later is an administrative decision, not missing evidence.
- Historical `jaredwinescom_default`, `olivetin_default`, `portainer_default`, and `palworld_default` network observations are not part of the current JAR-31 bridge map.
- Homepage backup-status access no longer depends on a guest UFW rule because UFW is not installed in the current VM.

---

## 22. Historical Network Verification — 2026-09-15

Read-only checks reconfirmed rotom, Linux Mint 22.3, eno1 at 192.168.1.69/24, gateway and DNS at 192.168.1.1, local DNS answers for rotom.casa, public website answers, NPM listeners, published ports, and NFS connectivity to 192.168.1.70. The media, backup, and shared-drive mounts are all present in fstab with systemd automount options. This resolves the older media-persistence uncertainty in this historical network audit. Router-side facts remain unverified.

For the later UID/GID and mount-state qualifications, use **NAS / NFS Networking** above. No network, NAS, account, or server configuration was changed as part of this documentation revision.

## 23. Related Documentation

- `02-Docker-Services.md` — service/Compose deployment details behind the network endpoints.
- `04-NAS-and-Storage.md` — canonical NFS export, mountpoint, and storage details.
- `01-Rotom-Server-Inventory.md` — high-level architecture/index summary.
