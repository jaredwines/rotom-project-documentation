# Rotom Guides

These guides help choose the right Runbook procedure. They are explanatory entry points, not independent operational instructions.

## Get oriented

Start with the [Runbook Server Inventory](../runbook/01-Rotom-Server-Inventory.md) for the current topology and task router. Use its *Start by Task or Symptom* table to select the canonical owner before acting on a service, network, storage, recovery, automation, or identity issue.

## Deploy or diagnose a Docker application

Read [Docker Services](../runbook/02-Docker-Services.md) first. It identifies the Compose contract, persistent data boundary, preflight, stop conditions, verification, and recovery path for Docker-backed services. Inspect the existing Compose definition before changing it, and do not recreate unrelated containers.

## Manage a reverse-proxy service

Use [Network and Domains](../runbook/03-Network-and-Domains.md) to trace the access path: client resolution, TLS, Nginx Proxy Manager route and access list, listener, and application upstream. Keep private-access and loopback boundaries intact; a successful backend response alone does not prove a safe proxy configuration.

## Work with NAS-backed storage

Use [NAS and Storage](../runbook/04-NAS-and-Storage.md) to identify the exact automount contract, service dependency, and ownership model. Never infer that a retired guest path means NAS data was deleted, and do not manufacture an outage merely to test recovery.

## Back up or restore Rotom

Follow [Backup and Restore](../runbook/05-Backup-and-Restore.md). It distinguishes the Rotom Restic repository, PVE configuration backup, VM backup, and NAS snapshot layers, including their verification and restoration limits. Treat credential handling and protected repository changes as separately controlled work.

## Troubleshoot scheduled work

[Maintenance and Automation](../runbook/06-Maintenance-and-Automation.md) is the source for timers, maintenance behavior, publisher workflow, and monitoring. Start with read-only evidence and preserve the existing safeguards around backups and publication.

## Use Codex with Rotom

Resolve the Runbook with `rs path`, read the repository instructions and the relevant authoritative source, inspect live state, compare it with the documented state, make the smallest targeted change, and verify it. Update the Runbook from the final verified state under its maintenance contract. Codex does not replace the required authorization boundaries for protected data, credentials, or PVE actions.
