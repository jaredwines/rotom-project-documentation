# Contributing to the legacy documentation set

This repository records verified Rotom state. Before preparing a change, read the canonical maintenance contract in [00 - Rotom Change Log](../00-Rotom-Change-Log.md). That document governs legacy documentation updates; this guide only summarizes the repository-facing expectations.

## Before changing documentation

1. Establish the final supported state from direct verification, supplied command output, or an explicit recorded decision.
2. Do not represent a suggestion, unrun command, failed experiment, or temporary troubleshooting state as current configuration.
3. Check all materially affected legacy documentation documents for contradictions and preserve valid historical evidence.
4. Never include passwords, tokens, private keys, VPN credentials, cookies, Home Assistant secrets, Restic passwords, or other authentication material.

## Writing standards

- Preserve the distinction between **Current**, **Historical**, **Retired**, **Proposed**, and **Needs Verification**.
- Identify the evidence source and date when verification matters.
- Use `America/Los_Angeles` for legacy documentation documentation-update dates.
- Preserve established filenames and cross-references where practical.
- Keep operational detail accurate and proportionate to the repository's approved visibility. Review carefully before sharing any infrastructure or personal information outside its intended audience.

### Procedural section structure

Use the following order where it applies; omit a heading only when it would add no useful information:

1. Purpose
2. Scope
3. Prerequisites
4. Current architecture or ownership
5. Procedure
6. Verification
7. Rollback or recovery
8. Troubleshooting
9. Related documentation
10. Last verified date or maintenance notes

Keep current-state facts in their canonical subject document. Link to those facts from procedures instead of copying commands, paths, identities, schedules, or configuration details into multiple files. Separate proposed work and historical evidence from supported current procedure.

### Staged history extraction

Do not extract historical sections from the Core Numbered Reference Set until all destination files can be added to Available Sources manually by Jared in the same coordinated release. An extraction must:

1. Inventory every section to be moved and every inbound filename or heading reference.
2. Define one historical destination per subject area rather than one file per ticket or date.
3. Copy and verify the history before removing it from the numbered source.
4. Replace removed material with a concise scope statement and direct link.
5. Validate recovery-topic coverage, internal links, generated Rotom Docs input, and Available Sources membership before treating the split as complete.
6. Keep the numbered current-state document authoritative throughout the transition; do not maintain duplicate canonical copies.

Until that coordinated release is possible, improve navigation and state labels in place. This preserves a complete nine-file legacy documentation and avoids silently dropping historical recovery evidence from Available Sources.

Proposed extraction map for that coordinated release:

| Current source | Proposed historical destination | Initial extraction scope |
| --- | --- | --- |
| `01-Rotom-Server-Inventory.md` | `history/01-Server-Inventory-History.md` | Historical ownership, pre-migration architecture, startup/dependency, limitations, and verification-record sections |
| `02-Docker-Services.md` | `history/02-Docker-Services-History.md` | Retired-domain inventories and historical verification sections; keep current runtime, reconstruction, gaps, and proposals in `02` |
| `03-Network-and-Domains.md` | `history/03-Network-and-Domains-History.md` | Pre-migration host, interface, routing, DNS, Docker-network, listener, firewall, flow, and verification sections |
| `04-NAS-and-Storage.md` | `history/04-NAS-and-Storage-History.md` | Preserved pre-migration mounts, ownership, layouts, service relationships, architecture, and verification sections |
| `05-Backup-and-Restore.md` | `history/05-Backup-and-Restore-History.md` | Final pre-migration baseline, preserved execution/scope/retention material, restore-kit history, and historical verification sections |
| `06-Maintenance-and-Automation.md` | `history/06-Maintenance-and-Automation-History.md` | Pre-migration and retired automation only; first disentangle the current subsections embedded among historical sections |
| `07-Users-and-Permissions.md` | `history/07-Users-and-Permissions-History.md` | Preserved pre-migration identity/access models and historical verification sections |
| `08-Rotom-Directory-Tree.txt` | `history/08-Directory-Tree-History.txt` | Pre-migration restoration targets and captured historical trees; retain current topology and current follow-ups in `08` |

Document 00 remains the canonical chronological log and maintenance contract rather than a subject-history extraction target. Before using these proposed paths, update the Rotom Docs publisher allowlist/build, repository validation, source map, related-document links, and Jared's manual Available Sources set as one reviewed change.

## Change history

For every substantive legacy documentation update, add one dated entry to the **Change History** in [00 - Rotom Change Log](../00-Rotom-Change-Log.md), following its required format. The entry should name the affected files, state what changed and why, identify evidence, and record outstanding work.

Do not add a change-history entry solely for an unchanged re-upload, download, packaging action, or formatting-only edit unless it alters an operational meaning. The full decision rule and update procedure are defined only in document 00.

## Before committing

- Confirm that changed facts are supported and state labels are accurate.
- Confirm affected documents and links remain consistent.
- Confirm no secrets, temporary files, generated packages, or unrelated changes are included.
- Confirm current instructions do not depend silently on historical paths, users, groups, mounts, networks, storage, backup scopes, or service ownership.
- Check referenced filenames and internal links, and search for stale references before any rename or move.
- Confirm critical recovery, backup, permissions, NAS, Docker, networking, Home Assistant, website, database, certificate, Restic, and Palworld information was preserved.
- Use the guarded `retired-helper` workflow: review with `retired-helper diff`, run `support/tooling/retired-helper lint` (or installed `retired-helper lint` only after its version is confirmed current), validate repository synchronization with `retired-helper check`, then use `retired-helper commit` and `retired-helper push` only when the canonical maintenance contract authorizes publication.

Repository-support files such as root `README.md`, root `AGENTS.md`, `.gitignore`, and files under `support/` are not legacy documentation members merely because they are stored here. Available Sources remains the legacy documentation membership boundary.
