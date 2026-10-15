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

Use the canonical [Runbook-First Maintenance Standard](../00-Rotom-Change-Log.md#runbook-first-maintenance-standard) for task-oriented operational procedures. A complete procedure includes, in this order where practical:

1. observable outcome;
2. precise entry conditions and explicit exclusions;
3. risk, impact, and authorization;
4. canonical current-state inputs;
5. read-only preflight;
6. explicit stop conditions;
7. the smallest targeted action and expected immediate result;
8. observable verification;
9. rollback or safe-stop recovery;
10. troubleshooting branches;
11. evidence to retain;
12. related canonical documentation; and
13. maintenance and revalidation triggers.

Use the applicable command-block label exactly: **Read-only**, **Read-only; may activate an existing automount**, **Changes Rotom**, **Changes Rotom and protected storage/repository state**, **Credential-sensitive**, **PVE — Jared executes**, or **Destructive — explicit approval required**. Never include secret values. A procedure must not make the reader infer desired state, authority, success criteria, or what to do when live state differs.

Prefer fail-fast, narrowly scoped commands. A state-changing block must be preceded by enough read-only evidence to resolve the exact target and must be followed by an application-level or system-level observable check. Define what to preserve when the action fails, and do not make destructive cleanup an automatic recovery step.

Keep current-state facts in their canonical subject document. Link to those facts from procedures instead of copying commands, paths, identities, schedules, or configuration details into multiple files. Separate proposed work and historical evidence from supported current procedure.

### Historical material

Keep historical evidence within `00`–`08` while the numbered runbook-first set remains the complete publication set. A future separately authorized extraction must:

1. Inventory every section to be moved and every inbound filename or heading reference.
2. Define one historical destination per subject area rather than one file per ticket or date.
3. Copy and verify the history before removing it from the numbered source.
4. Replace removed material with a concise scope statement and direct link.
5. Validate recovery-topic coverage, internal links, generated Rotom Docs input, and Available Sources membership before treating the split as complete.
6. Keep the numbered current-state document authoritative throughout the transition; do not maintain duplicate canonical copies.

Until that coordinated release is possible, improve navigation and state labels in place. This preserves a complete nine-file legacy documentation and avoids silently dropping historical recovery evidence from Available Sources.

The previously proposed extraction map is intentionally deferred; it is not an active path or publication plan. If extraction is later authorized, derive its destinations from the then-current numbered set and update the publisher, tests, source map, links, and Available Sources as one reviewed change.

Document 00 remains the canonical chronological log and maintenance contract rather than a subject-history extraction target.

## Change history

For every substantive legacy documentation update, add one dated entry to the **Change History** in [00 - Rotom Change Log](../00-Rotom-Change-Log.md), following its required format. The entry should name the affected files, state what changed and why, identify evidence, and record outstanding work.

Do not add a change-history entry solely for an unchanged re-upload, download, packaging action, or formatting-only edit unless it alters an operational meaning. The full decision rule and update procedure are defined only in document 00.

## Before committing

- Confirm that changed facts are supported and state labels are accurate.
- Confirm affected documents and links remain consistent.
- Confirm no secrets, temporary files, generated packages, or unrelated changes are included.
- Confirm current instructions do not depend silently on historical paths, users, groups, mounts, networks, storage, backup scopes, or service ownership.
- Run `bash "$(retired-helper path)/support/tooling/test-retired-helper-runbooks"` and complete a semantic cross-document ownership/duplication review; the shell test cannot prove semantic uniqueness.
- Check referenced filenames and internal links, and search for stale references before any rename or move.
- Confirm critical recovery, backup, permissions, NAS, Docker, networking, Home Assistant, website, database, certificate, Restic, and Palworld information was preserved.
- Use the guarded `retired-helper` workflow: review with `retired-helper diff`, run `support/tooling/retired-helper lint` (or installed `retired-helper lint` only after its version is confirmed current), validate repository synchronization with `retired-helper check`, then use `retired-helper commit` and `retired-helper push` only when the canonical maintenance contract authorizes publication.

Repository-support files such as root `README.md`, root `AGENTS.md`, `.gitignore`, and files under `support/` are not legacy documentation members merely because they are stored here. Available Sources remains the legacy documentation membership boundary.
