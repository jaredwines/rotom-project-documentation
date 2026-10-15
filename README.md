# legacy documentation set

This repository is the Git working copy for the **legacy documentation set** (**legacy documentation**): the maintained, evidence-based record of Rotom's architecture, configuration, operations, recovery posture, and material decisions.

The legacy documentation describes supported state. It distinguishes **Current**, **Historical**, **Retired**, **Proposed**, and **Needs Verification** information so that plans and old evidence are not mistaken for live configuration.

## Start here

Begin with [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md), the architecture and navigation index. For how the documentation is maintained and for the historical record, see [00 - Rotom Change Log](00-Rotom-Change-Log.md).

Use this reading order for administration or recovery:

1. Start with document 01 to identify the current system boundary and canonical subject owner.
2. Read the current-state sections near the start of the relevant subject document.
3. Use reconstruction, recovery, and troubleshooting sections only after reconciling them with the current-state sections and read-only live evidence.
4. Treat preserved pre-migration and historical sections as evidence, not as current instructions.

## Documentation map

The current legacy documentation consists of the Core Numbered Reference Set:

| Document | Purpose |
| --- | --- |
| [00 - Rotom Change Log](00-Rotom-Change-Log.md) | Canonical maintenance contract and material change history |
| [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md) | Architecture overview, source map, and starting index |
| [02 - Docker Services](02-Docker-Services.md) | Service and container configuration |
| [03 - Network and Domains](03-Network-and-Domains.md) | Network, DNS, proxy, and access topology |
| [04 - NAS and Storage](04-NAS-and-Storage.md) | NAS mounts, storage boundaries, and data layout |
| [05 - Backup and Restore](05-Backup-and-Restore.md) | Backup design, recovery material, and restoration evidence |
| [06 - Maintenance and Automation](06-Maintenance-and-Automation.md) | Recurring maintenance and operational automation |
| [07 - Users and Permissions](07-Users-and-Permissions.md) | Identities, ownership, and access boundaries |
| [08 - Rotom Directory Tree](08-Rotom-Directory-Tree.txt) | Recorded filesystem reference tree |

Available Sources defines the legacy documentation membership boundary. Repository-support files, including this README, are not legacy documentation members unless intentionally added to Available Sources. The current source map and membership details live in [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md).

## Using this repository

- Treat dated evidence and explicit state labels as more authoritative than unstated assumptions.
- Use the current legacy documentation documents for operational reference; do not use older change-log entries as instructions to recreate prior state.
- Use the guarded `retired-helper` helper for normal repository operations. `retired-helper path` resolves the active checkout.
- Run the repository's offline `support/tooling/retired-helper lint` validation before synchronization checks. After the matching helper version is installed, `retired-helper lint` is equivalent.
- From any directory, run `bash "$(retired-helper path)/support/tooling/test-runbooks"` to validate the three pilot runbooks against the final template's required safety structure.
- Read the [contribution guide](support/CONTRIBUTING.md) before preparing a change.

## Pilot runbooks

The repository contains a staged task-oriented runbook layer under [runbooks/](runbooks/README.md):

- [final runbook template](runbooks/RUNBOOK-TEMPLATE.md);
- [Docker-backed website diagnosis and recovery](runbooks/Docker-Website-Recovery.md);
- [NAS mount diagnosis and recovery](runbooks/NAS-Mount-Recovery.md);
- [Rotom Restic Backup execution and verification](runbooks/Rotom-Restic-Backup.md).

These files are prepared repository sources but do not become current legacy documentation set until Jared manually adds them to Available Sources. Rotom Docs requires a separate publisher/source-list update before it can publish them. Until both steps are complete, the nine-file Core Numbered Reference Set remains the complete current legacy documentation.

## Safety

Never commit passwords, tokens, private keys, VPN credentials, cookies, Home Assistant secrets, Restic passwords, or other authentication material. Do not publish or share repository contents without reviewing them for sensitive operational, personal, or infrastructure information.

## Keeping the legacy documentation current

The canonical legacy documentation maintenance contract is [00 - Rotom Change Log](00-Rotom-Change-Log.md). It defines the required evidence standard, consistency review, state labels, dated change-history entries, and the supported publication workflow. This README and the [contribution guide](support/CONTRIBUTING.md) are repository guides and do not replace that contract.
