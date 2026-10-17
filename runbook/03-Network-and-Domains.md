# 03 - Rotom Network and Domains

**Documentation set:** Rotom Support
**Document role:** Canonical source for Rotom LAN, DNS, Docker networking, ports, Cloudflare, NPM, and domain routing
**Hosts:** PVE hypervisor `pve` plus Debian VM `rotom`  
**Baseline verified:** Mixed evidence dates; see section-level evidence notes  
**Documentation updated:** 2026-10-15 — Migrate the private documentation portal to Rotom Support
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Navigation and Authority

- Start with **1. Current Network and Exposure Summary**, **2. Current Access Paths**, and **3. Current Listener and Docker-Network Model**.
- Treat retired network contracts and pre-migration evidence as reconstruction context, not current firewall, DNS, listener, or routing instructions.
- Use **8. Reconstruction-Critical Information** only after comparing it with the current sections and read-only live state.
- Review **Known Gaps / Needs Verification** before treating router, exposure, certificate, or application-path behavior as settled.
- This file owns network and routing facts; storage ownership, service deployment, backup, and account privileges remain canonical in documents 02 and 04–07.

## Start by Task or Symptom

- Application or container itself unavailable: start in [document 02](02-Docker-Services.md#diagnose-and-recover-a-container-or-docker-backed-website).
- Direct application works, but proxy, listener, DNS, TLS, routing, or public access fails: use the procedure below.
- NAS route or NFS path failure: use [document 04](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount).

## Diagnose a Proxy, Listener, DNS, TLS, Routing, or External-Access Failure

### Outcome, entry conditions, and exclusions

Locate the first failing layer without changing DNS, certificates, firewall, proxy records, WAN forwarding, or application data. Use this when a supported direct application endpoint is known or can be established from current reference. Exclude new domains, new exposure, certificate issuance, router changes, and PVE changes; those require separate scope and authorization.

### Risk and authorization

The diagnostic path is read-only. Restarting a container belongs to document 02. Proxy-record, certificate, DNS, firewall, router, or PVE changes are outside this procedure. Stop before exposing a credential or making a shared NPM change merely to test a theory.

### Canonical inputs and read-only preflight

Use the current route, listener, address, DNS, and certificate tables in this document plus the application definition in document 02. Confirm the expected hostname, direct host/loopback listener, NPM route, access-list expectation, and whether public reachability is intended.

**Read-only**

```bash
sudo ss -lntup
getent ahostsv4 rotom.casa
curl -sS -H 'Host: rotom.casa' -o /dev/null -w '%{http_code}\n' --max-time 10 http://127.0.0.1:3001/
curl -k -sS -o /dev/null -w '%{http_code}\n' --resolve rotom.casa:443:127.0.0.1 https://rotom.casa/
sudo docker logs --since 10m nginx-proxy-manager
```

Replace the example hostname and documented direct endpoint only with values from the current reference. `-k` tests routing/SNI, not certificate trust.

### Stop conditions

Stop if the direct endpoint also fails; the expected listener or route is not documented; desired public exposure is unclear; NPM data, certificate state, DNS, or router state conflicts materially with this document; a credential would be displayed; or the next action would affect other proxy hosts.

### Procedure and expected result

1. Verify the application/container and direct listener in document 02. If they fail, recover there and return.
2. Probe local-SNI HTTPS through `127.0.0.1`. A success isolates NPM routing from DNS/WAN behavior.
3. Resolve the hostname through the client/LAN resolver and compare the answer with this document. Do not edit DNS during diagnosis.
4. Probe the intended LAN or public URL from the appropriate vantage point. A local-SNI success with external failure points to DNS, hairpin/WAN forwarding, access-list, or upstream reachability rather than the application.
5. Inspect only the relevant NPM logs and current proxy-host record/configuration. Prepare a separately reviewed smallest correction if recorded and generated state differ.

### Verification, recovery, and troubleshooting

Success is an expected direct response, expected local-SNI response, correct resolution for the tested vantage point, and expected LAN/public response where that access is intended. If a diagnostic probe changes nothing, no rollback is needed. If a separately authorized correction fails, preserve the prior record/configuration and stop rather than broadening exposure or restarting all proxy services.

- Direct fails: return to document 02.
- Listener absent: compare rendered Compose and container publications in document 02.
- Direct works, local-SNI fails: inspect the exact NPM host and shared NPM health.
- Local-SNI works, LAN/public fails: inspect resolver answer, access list, routing/hairpin, WAN forwarding, and certificate trust as distinct layers.
- TLS route works only with `-k`: investigate trust/name/chain/expiry; do not call routing success certificate verification.

Retain the time, vantage point, hostname, resolved address, direct/local-SNI/external status codes, relevant listener and proxy-host identity, and sanitized error. Revalidate after any address, listener, route, domain, DNS, proxy, certificate, firewall, or access-policy change.

## 1. Current Network and Exposure Summary

**Evidence date:** 2026-10-15 unless stated otherwise.

Rotom is a DHCP client with a UniFi reservation at `192.168.1.69`. PVE is `192.168.1.68`, and the UniFi NAS is `192.168.1.70`, all on the trusted `192.168.1.0/24` LAN.

PVE firewalling is disabled and Rotom has no UFW installation. Jared accepted the documented direct-LAN listener posture as intentional. This does not authorize new listeners, WAN forwarding, DNS records, proxy routes, or firewall changes.

## 2. Current Access Paths

Nginx Proxy Manager is host-networked and owns the guest's HTTP/HTTPS reverse-proxy listeners. Current important routes are:

| Name | Upstream | Notes |
|---|---|---|
| `rotom.casa` | Homepage on `192.168.1.69:3001` | Homepage requires `Host: rotom.casa`; a bare loopback request returning `400` is expected |
| `support.rotom.casa` | `127.0.0.1:8082` | Private Rotom Support portal; NPM host `27`, wildcard certificate `24`, and trusted-LAN access list `1` |
| `docs.rotom.casa` | `127.0.0.1:8082` | Legacy private hostname; NPM host `22` permanently redirects paths and query strings to `https://support.rotom.casa/runbook/` |
| `pve.rotom.casa` | HTTPS `192.168.1.68:8006` | Forced TLS, WebSockets, and the configured access list |
| `jaredwines.com` | `127.0.0.1:7777` | Active behind NPM proxy host `8`; public and local-SNI checks returned `200` |
| `alohamillworks.com` | `192.168.1.69:7778` | Active website |

Application routes for Paperless, Prowlarr, RomM, Home Assistant, Homebridge, Jellyfin, Radarr, Sonarr, and the other supported services must be diagnosed through the access-path runbook above and their current NPM records. Do not infer route state from a dated database table.

`jaredwines.com` is active behind NPM proxy host `8`. Glances no longer has a host listener; Homepage reaches it through private Docker service DNS.

## 3. Current Listener and Docker-Network Model

Supported host or loopback publications include Homepage `3001`, Arcane `3552`, Rotom Support `127.0.0.1:8082`, qBittorrent `127.0.0.1:8080`, Jared Wines `127.0.0.1:7777`, Aloha `7778`, Jellyfin `8096`, Home Assistant `8123`, Radarr `7878`, Sonarr `8989`, Prowlarr `9696`, NPM `80/81/443`, and the read-only Rotom backup-status API `8787`.

Container ports and current process listeners are time-sensitive; verify them read-only before diagnosis or change.

Active Compose files no longer reference the former shared bridges `rotom-proxy`, `rotom-arr`, or `rotom-monitoring`. Workloads use private Compose networks, host networking, or documented host/loopback listeners. Actual residual Docker network objects were not independently enumerated during the documentation cleanup.

## 4. Current NAS and NFS Connectivity

Rotom's five current guest NFSv3 contracts use systemd automounts to `192.168.1.70`:

- `/mnt/nas-game-servers`
- `/mnt/nas-smart-home`
- `/mnt/nas-libraries`
- `/mnt/nas-hosted-websites`
- `/mnt/nas-rotom-restic-backup`

All five were observed mounted read/write during the audit. An automount declaration is not a readiness guarantee; use document 04 to verify the exact source, mount type, generated units, ownership, and sentinel before recovering a consumer.

PVE has separate backup-only NAS paths for PVE Restic Backup and Rotom VM Backup. It does not mount guest application shares.

## 5. DNS, DDNS, and TLS

Cloudflare DDNS remains an outbound-only Platform service. Its protected token value is not documented. NPM remains the TLS termination and reverse-proxy authority for Rotom application routes.

Local and public DNS can legitimately differ. Diagnose the local resolver result, public authoritative result, NPM route, certificate, access list, and upstream independently. A wildcard DNS answer does not prove that an application or proxy route exists.

Home Assistant and Homebridge explicitly use router DNS `192.168.1.1` in their Compose definitions. On 2026-10-16 their previous container resolver files had no nameserver even though the host resolver worked. For outbound DNS failures, compare the host and affected container resolver files and resolution results before changing shared DNS. After the PVE reboot, both containers retained the configured DNS and Home Assistant resolved the Sensibo hostname; the integration's full operation was not independently verified.

NPM certificate-renewal behavior remains current. Jared confirmed a successful renewal event; inspect the currently served certificate and NPM renewal state during certificate incidents rather than relying on the historical certificate dates.

## 6. Administrative and Monitoring Access

- Rotom SSH listens on the trusted LAN and is administered by authorized human accounts.
- PVE UI is reachable directly on the LAN and through its restricted NPM route.
- PVE CPU-temperature status is exposed read-only at `192.168.1.68:8788`.
- PVE backup schedule status is exposed read-only at `192.168.1.68:8789`.
- Rotom backup status is exposed read-only at `192.168.1.69:8787`.

These status services have no backup-control endpoint.

## 7. Retired Network Contracts

Uptime Kuma route/listener `3002`, Syncthing GUI `8384`, Gamarr `6767`, and browser upload `8090` are retired and had no listeners during the 2026-10-15 audit. Historical NPM rows or wildcard DNS must not be treated as active routes.

The guest mount paths and network dependencies associated with retired Media, Downloader, Documents, Filesync, singular Library, Web, Gameserver, Smarthome, Customapps, Apps, Auth, and Shared Drive contracts are not current.

The pre-virtualization bare-metal interface, routing, UFW, Docker bridge-address, and listener inventories are retained in Git history and the protected change log. They are not instructions for the current VM.

## 8. Reconstruction-Critical Information

- Preserve Rotom reservation `192.168.1.69`, PVE `192.168.1.68`, NAS `192.168.1.70`, and the currently documented MAC/VM identity contracts.
- Preserve NPM state, certificates, access lists, and host-networking role.
- Preserve loopback-only exposure for Jared Wines, Rotom Support, and qBittorrent. Preserve the trusted-LAN restriction on both the support hostname and its legacy redirect.
- Preserve the five exact guest NFS sources from document 04.
- Keep secret values out of diagnostic output and documentation.
- Reconcile public DNS, LAN DNS, TLS, proxy configuration, listeners, and application state before changing any layer.

## 9. Known Gaps / Needs Verification

- Router WAN forwarding and VLAN implementation remain outside guest-level evidence.
- Residual Docker network objects require privileged Docker inspection before any object cleanup.
- Retained historical NPM database rows require owner intent plus generated-config and route checks before removal.
- Continue normal PVE NIC/iGPU monitoring; existing evidence does not prove a single cause for historical resets.

## 10. Related Documentation

- [02 — Docker Services](02-Docker-Services.md)
- [04 — NAS and Storage](04-NAS-and-Storage.md)
- [06 — Maintenance and Automation](06-Maintenance-and-Automation.md)
- [07 — Users and Permissions](07-Users-and-Permissions.md)

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate after a host address, reservation, listener, published port, proxy route, access list, certificate, DNS record, Docker network, firewall posture, NFS source, or monitoring endpoint changes.
