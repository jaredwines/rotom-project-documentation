# Rotom Runbook

This repository is the Git working copy for the Rotom Runbook (RR).

## Documentation authority

- `00-Rotom-Change-Log.md` is the canonical RR maintenance contract.
- `01-Rotom-Server-Inventory.md` is the starting index when unsure which RR document applies.
- Follow the current RR rather than duplicating its detailed procedures in this file.
- Preserve historical entries as historical evidence. Do not silently rewrite old records to match current state.

Repository support files such as root `AGENTS.md`, `.gitignore`, `README.md`, and files under `support/` are not RR members merely because they are stored in this Git repository. Available Sources remains the RR membership boundary.

## Supplementary guides

Every file under `guide/` is separate from Rotom Runbook. Files in that directory are not RR members, are not part of the Core Numbered Reference Set, must not be added to Available Sources, and must not be required by RR-only validation or publication. Validate guide content with `support/tooling/test-guides`, not by expanding RR membership. A guide may link to RR fact owners; if a guide change also changes an RR-owned fact or procedure, update that RR owner separately under the maintenance contract.

## RR Git workflow

Use the `rr` helper for normal RR Git operations.

On Rotom, when the sandbox restricts `.git` or GitHub access, request the needed execution access **before** running `rr check`, `rr commit`, `rr push`, or other affected Git operations. Do not first run them in the restricted sandbox. Follow the stop-on-failure rules in `00-Rotom-Change-Log.md`.

- Use `rr path` to resolve the active local checkout.
- Use `rr status`, `rr check`, `rr pull`, `rr diff`, `rr log`, `rr commit`, and `rr push` rather than bypassing the helper's guardrails.
- Do not hard-code Mac or Rotom checkout paths in shared workflow instructions.
- Do not commit unrelated changes, temporary files, or secrets.
- If an `rr` safety check fails, stop and report the problem rather than automatically resetting, cleaning, merging, rebasing, force-pushing, or otherwise bypassing the guardrail.

When an RR update is authorized, follow `00-Rotom-Change-Log.md` for the environment-specific completion workflow.

## Safety

Never place passwords, API tokens, SSH private keys, VPN credentials, WireGuard private keys, cookies, Home Assistant secrets, Restic passwords, or other authentication material in this repository.

Use `vim` instead of `nano` for command-line text editing.
