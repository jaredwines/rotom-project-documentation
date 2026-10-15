# legacy documentation set

This repository is the Git working copy for the **legacy documentation set** (**legacy documentation**): a nine-file, runbook-first operating manual and evidence-based reference for Rotom.

The legacy documentation describes supported state. It distinguishes **Current**, **Historical**, **Retired**, **Proposed**, and **Needs Verification** information so that plans and old evidence are not mistaken for live configuration.

## Start here

Begin with [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md), the architecture and navigation index. For how the documentation is maintained and for the historical record, see [00 - Rotom Change Log](00-Rotom-Change-Log.md).

Use this reading order for administration or recovery:

1. Start with document 01 and choose the task or symptom.
2. Follow its link to the canonical procedure in documents 02–07, or to the path reference in 08.
3. Read the procedure's current-state inputs, preflight, and stop conditions before taking action.
4. Treat reconstruction, retired, and historical sections as evidence, not current instructions.

## Documentation map

The current legacy documentation consists of the Core Numbered Reference Set:

| Document | Purpose |
| --- | --- |
| [00 - Rotom Change Log](00-Rotom-Change-Log.md) | Canonical maintenance contract and material change history |
| [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md) | Task router, symptom index, architecture overview, and source map |
| [02 - Docker Services](02-Docker-Services.md) | Container and application operations, recovery, and reference |
| [03 - Network and Domains](03-Network-and-Domains.md) | Listener, routing, DNS, proxy, TLS, and access operations |
| [04 - NAS and Storage](04-NAS-and-Storage.md) | NAS/NFS diagnosis, mount recovery, and storage reference |
| [05 - Backup and Restore](05-Backup-and-Restore.md) | Backup execution, assurance levels, repositories, and restore operations |
| [06 - Maintenance and Automation](06-Maintenance-and-Automation.md) | Timers, automation, updates, and routine administration |
| [07 - Users and Permissions](07-Users-and-Permissions.md) | Identity, ownership, permission, privilege, and access operations |
| [08 - Rotom Directory Tree](08-Rotom-Directory-Tree.txt) | Current filesystem reference with historical paths clearly separated |

Available Sources defines the legacy documentation membership boundary. Repository-support files, including this README, are not legacy documentation members unless intentionally added to Available Sources. The current source map and membership details live in [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md).

## Using this repository

- Treat dated evidence and explicit state labels as more authoritative than unstated assumptions.
- Use the current legacy documentation documents for operational reference; do not use older change-log entries as instructions to recreate prior state.
- Use the guarded `retired-helper` helper for normal repository operations. `retired-helper path` resolves the active checkout.
- Run the repository's offline `support/tooling/retired-helper lint` validation before synchronization checks. After the matching helper version is installed, `retired-helper lint` is equivalent.
- From any directory, run `bash "$(retired-helper path)/support/tooling/test-retired-helper-runbooks"` to validate the numbered runbook-first legacy documentation's permanent structure, links, safety labels, and nine-file membership.
- Read the [contribution guide](support/CONTRIBUTING.md) before preparing a change.

## Safety

Never commit passwords, tokens, private keys, VPN credentials, cookies, Home Assistant secrets, Restic passwords, or other authentication material. Do not publish or share repository contents without reviewing them for sensitive operational, personal, or infrastructure information.

## Keeping the legacy documentation current

The canonical legacy documentation maintenance contract is [00 - Rotom Change Log](00-Rotom-Change-Log.md). It defines the required evidence standard, consistency review, state labels, dated change-history entries, and the supported publication workflow. This README and the [contribution guide](support/CONTRIBUTING.md) are repository guides and do not replace that contract.
