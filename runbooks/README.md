# Rotom Runbooks

**Document role:** Task-oriented entry point for safe Rotom operations  
**Status:** Pilot runbook layer prepared for manual addition to Available Sources  
**Documentation updated:** 2026-10-15  
**Canonical current-state index:** [01-Rotom-Server-Inventory.md](../01-Rotom-Server-Inventory.md)  
**Maintenance contract:** [00-Rotom-Change-Log.md](../00-Rotom-Change-Log.md)

These runbooks answer **how to perform or recover a specific operation safely**. The numbered reference documents remain authoritative for current architecture, identities, paths, ports, storage contracts, backup scope, and other system facts. A runbook links to those owners rather than replacing them.

These pilot files are repository sources until Jared manually adds them to Available Sources. Available Sources remains the legacy documentation membership boundary. Do not describe the runbook layer as current legacy documentation or publish it through Rotom Docs until that manual addition and the corresponding publisher/source-map update are complete.

## Start by task

| Need | Runbook |
|---|---|
| A Docker-backed website is unavailable or returns `502` | [Diagnose and Recover a Docker-Backed Website](Docker-Website-Recovery.md) |
| A current Rotom NAS mount is missing or a NAS-backed container failed during startup | [Diagnose and Recover a Rotom NAS Mount](NAS-Mount-Recovery.md) |
| Run, verify, or troubleshoot Rotom Restic Backup | [Run and Verify Rotom Restic Backup](Rotom-Restic-Backup.md) |

## Safety model

Runbook command blocks use these labels:

- **Read-only:** observes state and is not intended to change configuration. A read of a systemd automount path can activate that already-configured mount and is labeled separately.
- **Changes Rotom:** changes service, container, mount, repository, or other live state on the Rotom VM.
- **PVE — Jared executes:** applies to the Proxmox host and must be run manually by Jared under the established PVE workflow.
- **Destructive — explicit approval required:** may delete, overwrite, prune outside an established policy, or make recovery materially harder.
- **Credential-sensitive:** may access protected authentication material. Never print, paste, or record secret values.

Every runbook includes explicit stop conditions. If a stop condition is reached, preserve the evidence and reconcile the discrepancy with the numbered references and live state before continuing.

## Authoring standard

New runbooks must follow [RUNBOOK-TEMPLATE.md](RUNBOOK-TEMPLATE.md). Keep commands narrowly scoped, show expected results, and use the smallest targeted recovery action. Do not use a runbook to decide desired deployment state, invent missing authority, or replay historical commands as current instructions.

