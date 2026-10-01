# Contributing to the Rotom Project Documentation

This repository records verified Rotom state. Before preparing a change, read the canonical maintenance contract in [00 - Rotom Change Log](00-Rotom-Change-Log.md). That document governs RPD updates; this guide only summarizes the repository-facing expectations.

## Before changing documentation

1. Establish the final supported state from direct verification, supplied command output, or an explicit recorded decision.
2. Do not represent a suggestion, unrun command, failed experiment, or temporary troubleshooting state as current configuration.
3. Check all materially affected RPD documents for contradictions and preserve valid historical evidence.
4. Never include passwords, tokens, private keys, VPN credentials, cookies, Home Assistant secrets, Restic passwords, or other authentication material.

## Writing standards

- Preserve the distinction between **Current**, **Historical**, **Retired**, **Proposed**, and **Needs Verification**.
- Identify the evidence source and date when verification matters.
- Use `America/Los_Angeles` for RPD documentation-update dates.
- Preserve established filenames and cross-references where practical.
- Keep operational detail accurate and proportionate to the repository's approved visibility. Review carefully before sharing any infrastructure or personal information outside its intended audience.

## Change history

For every substantive RPD update, add one dated entry to the **Change History** in [00 - Rotom Change Log](00-Rotom-Change-Log.md), following its required format. The entry should name the affected files, state what changed and why, identify evidence, and record outstanding work.

Do not add a change-history entry solely for an unchanged re-upload, download, packaging action, or formatting-only edit unless it alters an operational meaning. The full decision rule and update procedure are defined only in document 00.

## Before committing

- Confirm that changed facts are supported and state labels are accurate.
- Confirm affected documents and links remain consistent.
- Confirm no secrets, temporary files, generated packages, or unrelated changes are included.
- Use the guarded `rpd` workflow: review with `rpd diff`, validate with `rpd check`, then use `rpd commit` and `rpd push` only when the canonical maintenance contract authorizes publication.

Repository-support files such as `README.md`, `CONTRIBUTING.md`, `AGENTS.md`, `.gitignore`, and `tooling/` are not RPD members merely because they are stored here. Available Sources remains the RPD membership boundary.
