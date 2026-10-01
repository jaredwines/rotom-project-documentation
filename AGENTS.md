# Rotom Project Documentation

This repository is the Git working copy for the Rotom Project Documentation (RPD).

## Documentation authority

- `00-Rotom-Change-Log.md` is the canonical RPD maintenance contract.
- `01-Rotom-Server-Inventory.md` is the starting index when unsure which RPD document applies.
- Follow the current RPD rather than duplicating its detailed procedures in this file.
- Preserve historical entries as historical evidence. Do not silently rewrite old records to match current state.

Repository support files such as root `AGENTS.md`, `.gitignore`, `README.md`, and files under `support/` are not RPD members merely because they are stored in this Git repository. Available Sources remains the RPD membership boundary.

## RPD Git workflow

Use the `rpd` helper for normal RPD Git operations.

- Use `rpd path` to resolve the active local checkout.
- Use `rpd status`, `rpd check`, `rpd pull`, `rpd diff`, `rpd log`, `rpd commit`, and `rpd push` rather than bypassing the helper's guardrails.
- Do not hard-code Mac or Rotom checkout paths in shared workflow instructions.
- Do not commit unrelated changes, temporary files, or secrets.
- If an `rpd` safety check fails, stop and report the problem rather than automatically resetting, cleaning, merging, rebasing, force-pushing, or otherwise bypassing the guardrail.

When an RPD update is authorized, follow `00-Rotom-Change-Log.md` for the environment-specific completion workflow.

## Safety

Never place passwords, API tokens, SSH private keys, VPN credentials, WireGuard private keys, cookies, Home Assistant secrets, Restic passwords, or other authentication material in this repository.

Use `vim` instead of `nano` for command-line text editing.
