# Rotom Architecture

This page is a verified orientation to Rotom's design. For current operational facts and procedures, use the linked [Runbook](../runbook/).

## Physical infrastructure and virtualization

Rotom is a Debian virtual machine (VMID 100) on the `pve` Proxmox VE host. The VM uses the reserved LAN address `192.168.1.69`; the PVE host is `192.168.1.68`; and the UniFi NAS is `192.168.1.70`. The [Server Inventory](../runbook/01-Rotom-Server-Inventory/) is the authoritative current system map.

## Application platform

Compose-managed workloads are grouped under five local domains: Platform, Libraries, Hosted Websites, Game Servers, and Smart Home. Their stack definitions, local application state, identities, and recovery boundaries are documented by [Docker Services](../runbook/02-Docker-Services/). Persistent state stays on the VM unless the Runbook explicitly assigns it to NAS-backed storage.

`mkdocs-rr` is the read-only documentation service. Its guarded publisher uses a root-owned deployment clone, a fixed source allowlist, a clean-tree and fast-forward check, an isolated build, and an atomic generated-site swap. The service remains loopback-only; it is not a source of authority or a writable documentation checkout.

## Network, DNS, and private access

Nginx Proxy Manager is host-networked and terminates TLS for Rotom application routes. Cloudflare DDNS maintains the Rotom hostname, while the wildcard DNS and certificate cover supported subdomains. The Support portal is proxied to Rotom's loopback-only MkDocs endpoint and retains the trusted-LAN access policy. The [Network and Domains Runbook](../runbook/03-Network-and-Domains/) owns the exact routes, listeners, certificate, and access-list facts.

## Storage, backup, and recovery

Five NFSv3 systemd automount contracts provide scoped NAS dependencies. Application and system recovery use complementary layers: Rotom Restic backups, PVE Restic configuration backups, VM backups, and applicable NAS snapshots. They are deliberately not interchangeable. See [NAS and Storage](../runbook/04-NAS-and-Storage/) and [Backup and Restore](../runbook/05-Backup-and-Restore/) before relying on any recovery path.

## Security and ownership boundaries

Human administration and locked service identities are intentionally separate. Docker, NPM, generated documentation, and backup credentials have distinct ownership and access constraints. The [Users and Permissions Runbook](../runbook/07-Users-and-Permissions/) is the authoritative source for those contracts.

## Design principle

The portal explains the system without copying its controlling procedures. If this overview conflicts with the Runbook, treat the Runbook as authoritative and reconcile the discrepancy through its maintenance process.
