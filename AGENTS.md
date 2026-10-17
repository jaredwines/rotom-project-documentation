# Rotom Support

This repository is the Git working copy for the Rotom Support (RS).

## Documentation authority

- `runbook/00-Rotom-Change-Log.md` is the canonical RS maintenance contract.
- `runbook/01-Rotom-Server-Inventory.md` is the starting index when unsure which RS document applies.
- Follow the current RS rather than duplicating its detailed procedures in this file.
- Preserve historical entries as historical evidence. Do not silently rewrite old records to match current state.

Repository support files such as root `AGENTS.md`, `.gitignore`, `README.md`, and files under `support/` are not RS members merely because they are stored in this Git repository. Available Sources remains the RS membership boundary.

The current numbered RS files `00`–`08` live under `runbook/` in the checkout resolved by `rs path`. Relative links between these files remain within that directory.

## Supplementary guides

Every file under `guide/` is separate from Rotom Support. Files in that directory are not RS members, are not part of the Core Numbered Reference Set, must not be added to Available Sources, and must not be required by RS-only validation or publication. Validate guide content with `support/tooling/test-guides`, not by expanding RS membership. A guide may link to RS fact owners; if a guide change also changes an RS-owned fact or procedure, update that RS owner separately under the maintenance contract.

## RS Git workflow

Use the `rs` helper for normal RS Git operations.

On Rotom, when the sandbox restricts `.git` or GitHub access, request the needed execution access **before** running `rs check`, `rs commit`, `rs push`, or other affected Git operations. Do not first run them in the restricted sandbox. Follow the stop-on-failure rules in `runbook/00-Rotom-Change-Log.md`.

- Use `rs path` to resolve the active local checkout.
- Use `rs status`, `rs check`, `rs pull`, `rs diff`, `rs log`, `rs commit`, and `rs push` rather than bypassing the helper's guardrails.
- Do not hard-code Mac or Rotom checkout paths in shared workflow instructions.
- Do not commit unrelated changes, temporary files, or secrets.
- If an `rs` safety check fails, stop and report the problem rather than automatically resetting, cleaning, merging, rebasing, force-pushing, or otherwise bypassing the guardrail.

When an RS update is authorized, follow `runbook/00-Rotom-Change-Log.md` for the environment-specific completion workflow.

## Safety

Never place passwords, API tokens, SSH private keys, VPN credentials, WireGuard private keys, cookies, Home Assistant secrets, Restic passwords, or other authentication material in this repository.

Use `vim` instead of `nano` for command-line text editing.
