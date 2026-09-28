# 03 - Rotom Network and Domains

**Documentation set:** Rotom Project Documentation  
**Document role:** Canonical source for Rotom LAN, DNS, Docker networking, ports, Cloudflare, NPM, and domain routing  
**Hosts:** Proxmox hypervisor `proxmox` plus Debian VM `rotom`  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-09-28 — JAR-68 canonical PVE identity, SSH aliases, monitoring name, and PVE-only NFS names documented
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## 1. Purpose and Scope

### Evidence provenance

Last updated: 2026-09-27 (Proxmox host-config Restic NFS boundary added; JAR-66/JAR-33 VZDump boundary, JAR-32 guest Restic cutover, and JAR-31 application networking/reboot acceptance retained; pre-migration sections remain historical evidence)
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

## 2. Current Phase B Host, VM, and NAS Network Identity — 2026-09-28 JAR-68 final

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
| PVE host | `pve`, `pve.rotom.casa` | `192.168.1.68` / `root` | `~/.ssh/id_ed25519_pve` | fingerprint preserved as `SHA256:xVX+MEAncK6Z2aTbNinSufYpTwyER+NFSp8iIPf11Zg`; both canonical aliases logged into `pve` / `pve.rotom.casa`; old Mac aliases removed |

Password authentication policy was not changed. Key contents and credentials are never recorded. The public-key comment may retain historical wording; it is metadata only and not an active alias.

### Current PVE monitoring bridge

The read-only physical CPU temperature helper is `/usr/local/sbin/pve-cpu-temp-api` with `pve-cpu-temp-api.service`, listening on `192.168.1.68:8788`. Homepage uses the exact label **`PVE CPU Temperature`** and the Glances-compatible sensor endpoint. Final post-reboot monitoring verification passed.

### Current NAS / NFS connectivity

The Rotom guest retains its twelve fstab-backed NFS systemd automounts and two read-only bindfs compatibility views. Two PVE-only backup exports are canonical:

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

The new `rotom-nas-docker-recovery.service` waits until after `docker.service` and `network-online.target`, then actively verifies the Downloads/Media/Game NFS mounts and related bindfs prerequisites before recovering only containers whose stopped state carries a NAS/mount startup-failure signature. It changes no DNS or routing behavior. Manual validation passed; reboot validation is deferred.

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

### Gamarr application connectivity

Gamarr is a host-published bridge service on TCP `6767` and now has an active Nginx Proxy Manager route: `gamarr.rotom.casa -> 192.168.1.69:6767`. The verified NPM record uses certificate ID `24`, forces SSL, has no assigned access list, and returned HTTP `302` through the HTTPS route. Its qBittorrent download client connects to `192.168.1.69:8080` and uses category `Games`. Its working Prowlarr Torznab base is `http://192.168.1.69:9696/2`; Gamarr appends `/api`, so configuring a base that already ends in `/api` produces an invalid `/api/api` request. API keys are intentionally omitted.


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

## 10. Pre-Migration Nginx Proxy Manager

### Runtime model

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
| `/home/infra/docker/nginx-proxy-manager/data` | `/data` | read/write |
| `/home/infra/docker/nginx-proxy-manager/letsencrypt` | `/etc/letsencrypt` | read/write |

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
| `gamarr.rotom.casa` | HTTP | `192.168.1.69:6767` | enabled | Gamarr; NPM proxy-host ID 19, certificate ID 24, forced SSL, no access list; HTTPS probe returned `302` |

The legacy Smart Hub/Portainer/OliveTin/PalTools rows above remain `enabled=1` in the NPM database, but their IDs are absent from the currently generated `/home/infra/docker/nginx-proxy-manager/data/nginx/proxy_host/*.conf` set inspected in the later comprehensive audit. Treat them as stale/unresolved database records rather than active generated routes until administrator intent is reviewed. The current generated configuration includes the active Rotom/application routes, including Gamarr.

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

### Runtime model

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

“Change Homepage URL” supersedes the earlier Homepage hostname in the audit. The canonical URL is `https://rotom.casa`. Supplied output from `/home/infra/docker/nginx-proxy-manager/data/nginx/proxy_host/9.conf` shows `server_name rotom.casa;`, so the Homepage proxy no longer includes `homepage.rotom.casa`.

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

`/mnt/nas-game-servers` remains absent from fstab and unmounted. During the later JAR-6 UNAS audit the former `Game_Servers/.data` path was observed absent rather than merely unused. JAR-6 did not delete or recreate it. The active Game dependency remains `/mnt/nas-game` → `Game/.data`.

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

### UniFi / router layer — Needs Verification

Current VM reservation and local-DNS identity are already verified: `192.168.1.69` is reserved to VM MAC `BC:24:11:97:10:47`, and the LAN resolver returns `rotom.casa -> 192.168.1.69`. Remaining router-side facts not established by the RPD are:

- whether `192.168.1.70` is statically assigned/reserved for the UNAS;
- VLAN/network assignment for Proxmox, the Rotom VM, and UNAS;
- WAN port-forward/NAT rules for TCP 80/443, Palworld UDP ports, or other exposed services;
- UniFi firewall policy before traffic reaches Proxmox/Rotom;
- NAT reflection/hairpin behavior.

A read-only UniFi/controller inspection of the `.68`, `.69`, and `.70` clients plus firewall/NAT/port-forward configuration would resolve these items. Host-side `ip route`, resolver queries, and listener checks can reconfirm client state but cannot prove router configuration.

### Wake-on-LAN end-to-end trigger — Needs Verification

Host-side WOL is current and persistent: Proxmox `nic0` supports magic-packet wake, reports `Wake-on: g`, PCI wake is enabled, and `/etc/network/interfaces` reasserts `wol g` on interface bring-up. A complete physical power-off/return cycle also preserved the setting and restored VMID 100 plus the full Rotom runtime. The supplied evidence does not include the client-side magic-packet transmission, however, so it cannot establish that WOL caused that power-on. Resolve with one future controlled power-off where the sender invocation is captured before the NUC returns.

### Legacy NPM records — Needs Verification

Historical NPM evidence retained legacy Smart Hub/Portainer/OliveTin/PalTools database rows that were `enabled=1` but absent from generated `proxy_host/*.conf` files. The RPD does not contain a fresh JAR-31/JAR-33 reread proving whether those rows still exist in the restored NPM database or whether administrator intent is to retain them.

A read-only inspection of the current NPM SQLite database and generated proxy-host directory would resolve current presence; administrator intent still requires an explicit decision. Do not expose certificate/private-key contents while checking.

### Current exposure policy — Needs Verification

The Debian VM currently has **no UFW installation** per JAR-31; the broad UFW rules documented in section 14 belong to the retired bare-metal host. Current externally reachable services therefore cannot be inferred from those old rules.

The 2026-09-27 read-only audit refreshed Rotom listeners and Docker-published ports, so the host-side portion is verified current. UniFi WAN forwarding/firewall/NAT policy remains unverified and must still be inspected separately before drawing conclusions about Internet exposure.

### qBittorrent live NFS-loss behavior — Needs Verification

Boot/recreation fail-closed behavior and the JAR-31 recovery retry path are verified. The RPD does not establish what an already-running qBittorrent container does if the mounted Downloader NFS filesystem disappears after startup; no runtime watchdog is claimed. A purely read-only check cannot reproduce that failure mode. Resolve only through a separately planned non-destructive maintenance test, not by inferring from the startup guard.

### Resolved or intentional states — not gaps

- Current `/mnt/nas-downloaders`, `/mnt/nas-media`, `/mnt/nas-game`, both bindfs views, and boot-time recovery behavior were verified by JAR-31 and reverified during the later physical Proxmox power-cycle.
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