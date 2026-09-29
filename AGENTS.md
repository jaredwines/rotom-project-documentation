# legacy documentation set

This repository is the Git working copy for the legacy documentation set (legacy documentation).

## Documentation authority

- `00-Rotom-Change-Log.md` is the canonical legacy documentation maintenance contract.
- `01-Rotom-Server-Inventory.md` is the starting index when unsure which legacy documentation document applies.
- Follow the current legacy documentation rather than duplicating its detailed procedures in this file.
- Preserve historical entries as historical evidence. Do not silently rewrite old records to match current state.

Repository support files such as `AGENTS.md`, `.gitignore`, and files under `tooling/` are not legacy documentation members merely because they are stored in this Git repository. Available Sources remains the legacy documentation membership boundary.

## legacy documentation Git workflow

Use the `retired-helper` helper for normal legacy documentation Git operations.

- Use `retired-helper path` to resolve the active local checkout.
- Use `retired-helper status`, `retired-helper check`, `retired-helper pull`, `retired-helper diff`, `retired-helper log`, `retired-helper commit`, and `retired-helper push` rather than bypassing the helper's guardrails.
- Do not hard-code Mac or Rotom checkout paths in shared workflow instructions.
- Do not commit unrelated changes, temporary files, or secrets.
- If an `retired-helper` safety check fails, stop and report the problem rather than automatically resetting, cleaning, merging, rebasing, force-pushing, or otherwise bypassing the guardrail.

When an legacy documentation update is authorized, follow `00-Rotom-Change-Log.md` for the environment-specific completion workflow.

## Safety

Never place passwords, API tokens, SSH private keys, VPN credentials, WireGuard private keys, cookies, Home Assistant secrets, Restic passwords, or other authentication material in this repository.

Use `vim` instead of `nano` for command-line text editing.
