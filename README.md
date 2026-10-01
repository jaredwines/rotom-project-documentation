# Rotom Project Documentation

This repository is the Git working copy for the **Rotom Project Documentation** (**RPD**): the maintained, evidence-based record of Rotom's architecture, configuration, operations, recovery posture, and material decisions.

The RPD describes supported state. It distinguishes **Current**, **Historical**, **Retired**, **Proposed**, and **Needs Verification** information so that plans and old evidence are not mistaken for live configuration.

## Start here

Begin with [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md), the architecture and navigation index. For how the documentation is maintained and for the historical record, see [00 - Rotom Change Log](00-Rotom-Change-Log.md).

## Documentation map

The current RPD consists of the Core Numbered Reference Set:

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

Available Sources defines the RPD membership boundary. Repository-support files, including this README, are not RPD members unless intentionally added to Available Sources. The current source map and membership details live in [01 - Rotom Server Inventory](01-Rotom-Server-Inventory.md).

## Using this repository

- Treat dated evidence and explicit state labels as more authoritative than unstated assumptions.
- Use the current RPD documents for operational reference; do not use older change-log entries as instructions to recreate prior state.
- Use the guarded `rpd` helper for normal repository operations. `rpd path` resolves the active checkout.
- Read [CONTRIBUTING.md](CONTRIBUTING.md) before preparing a change.

## Safety

Never commit passwords, tokens, private keys, VPN credentials, cookies, Home Assistant secrets, Restic passwords, or other authentication material. Do not publish or share repository contents without reviewing them for sensitive operational, personal, or infrastructure information.

## Keeping the RPD current

The canonical RPD maintenance contract is [00 - Rotom Change Log](00-Rotom-Change-Log.md). It defines the required evidence standard, consistency review, state labels, dated change-history entries, and the supported publication workflow. This README and [CONTRIBUTING.md](CONTRIBUTING.md) are repository guides and do not replace that contract.
