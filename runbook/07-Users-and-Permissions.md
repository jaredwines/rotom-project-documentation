# 07 - Users and Permissions

**Documentation set:** Rotom Support
**Document role:** Canonical source for Linux accounts, UID/GID identities, groups, privileges, ACLs, and access policy
**Hosts:** PVE hypervisor `pve` and Debian VM `rotom`  
**Baseline verified:** Current state reconciled through 2026-10-15; see outstanding items for remaining verification
**Documentation updated:** 2026-10-15 — Prepare Rotom Runbook identity and migration gates
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`, `06-Maintenance-and-Automation.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Start by Task or Symptom

- Permission denied, unexpected owner/group/mode/ACL, or service identity mismatch: use [Diagnose an Identity, Ownership, or Access Failure](#diagnose-an-identity-ownership-or-access-failure).
- NAS-root or NFS permission behavior: establish the mount/export contract in [document 04](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount) and interpret numeric identity here.
- Container process identity: compare Compose/runtime state in [document 02](02-Docker-Services.md#start-by-task-or-symptom).

## Diagnose an Identity, Ownership, or Access Failure

### Outcome, entry conditions, and exclusions

Determine whether denial or unexpected access comes from account identity, primary/supplementary groups, parent traversal, ownership, mode, ACL, sudo policy, Docker privilege, or an NFS numeric-ID contract. Exclude recursive normalization, UID/GID renumbering, broad group additions, sudo expansion, SSH-key work, NAS policy changes, and credential access.

### Risk, canonical inputs, and preflight

Read-only diagnosis is authorized. Ownership, mode, ACL, membership, sudo, SSH, or service-identity changes alter security boundaries and require explicit, scoped authorization plus a current owner contract from this document. Docker-group membership is root-equivalent. On NFS, preserve numeric UID/GID and the documented NAS contract.

**Read-only**

```bash
id ACCOUNT
getent passwd ACCOUNT
getent group GROUP
namei -l /exact/path
stat -c '%n %u:%g %a' /exact/path
getfacl -p /exact/path
sudo -u ACCOUNT test -r /exact/path
sudo -u ACCOUNT test -w /exact/path
sudo -u ACCOUNT test -x /exact/path
```

Use the exact account and path from the current identity/path reference. The `test` commands produce status, not file content.

### Stop conditions

Stop if the expected owner/identity is not documented, the path is a NAS mount with a source/mount mismatch, the change would be recursive or cross a filesystem boundary, the target contains credentials/private keys, an application deliberately maintains the metadata, or the proposed fix changes UID/GID, sudo, SSH, Docker membership, or an appliance-side policy.

### Smallest targeted action, verification, and recovery

Prefer running service-specific administration as the established service identity rather than broadening membership:

**Changes Rotom**

```bash
sudo -iu SERVICE_ACCOUNT
```

Any `chown`, `chmod`, `setfacl`, `usermod`, sudoers, or SSH change must be separately reviewed with an exact non-recursive target and recorded prior metadata. Expected immediate result is only the intended access outcome; unrelated identities must remain unchanged. Re-run `stat`, `getfacl`, positive access as the intended identity, negative access as an unrelated identity where safe, application verification, and `systemctl --failed`. If verification fails, restore the recorded owner/mode/ACL or stop before expanding scope. Never use `chmod 777`.

Retain the exact path, numeric and named identity, parent traversal result, prior/final metadata, positive and negative access results, authorization, and application result without recording protected content. Revalidate after any account, UID/GID, group, ownership, mode, ACL, sudo rule, SSH access, service identity, or privilege-boundary change.

## Navigation and Authority

- Start with **1. Current Identity and Access Model** and **2. Ownership and Permission Contracts** for supported accounts, UID/GID contracts, groups, ACLs, and privilege boundaries.
- Treat retired identities and historical evidence as context, not as instructions to recreate retired users, groups, homes, memberships, or permissions.
- Use **7. Reconstruction-Critical Information** only after reconciling it with sections 1–2 and read-only NSS, ownership, ACL, sudo, and Docker-group evidence.
- Review **Outstanding / Needs Verification** before granting access or changing an identity boundary.
- This file owns identities and access policy; storage mount semantics, Docker deployment, and operational workflow remain canonical in documents 04, 02, and 06.

## 1. Current Identity and Access Model

### Human administrators

| Host | Account | Current access |
|---|---|---|
| Rotom | `jared` | Primary administrator; sudo and GitHub/RS owner workflow |
| Rotom | `fran` (`1002:1002`) | Full sudo; groups include `users` and Docker GID `989` |
| PVE | `jared` | Primary administrator |
| PVE | `fran` (`1001:1001`) | Sudo, `users`, `systemd-journal`, and propagated PVE Administrator role |

Docker group membership is root-equivalent. Fran is the sole listed human Docker-group member; Jared and service identities remain outside it.

### Current service identities

| Domain | UID:GID | NSS home field | Active roots |
|---|---:|---|---|
| Platform | `997:5002` | `/home/platform` | `/srv/rotom/{stacks,appdata,secrets}/platform` |
| Libraries | `127:5000` | `/home/libraries` | `/srv/rotom/{stacks,appdata,secrets}/libraries` |
| Hosted Websites | `902:5004` | `/home/hosted-websites` | `/srv/rotom/{stacks,appdata}/hosted-websites` |
| Game Servers | `995:5001` | `/home/game-servers` | `/srv/rotom/{stacks,appdata,secrets}/game-servers` |
| Smart Home | `126:5003` | `/home/smart-home` | `/srv/rotom/{stacks,appdata}/smart-home` |
| Remote Desktop Commander | `1001:1001` | `/opt/desktop-commander` | Native application and workspace below `/opt/desktop-commander` |

The five `/home/<service>` directories are absent even though NSS retains those fields. Do not recreate them as deployment or recovery roots. All service accounts are locked.

## 2. Ownership and Permission Contracts

`/srv/rotom` is root-administered. Declarative `stacks/` and `scripts/` content, mutable `appdata/`, protected `secrets/`, and any explicitly retained recovery material have different ownership and exposure requirements. Preserve existing owners, modes, ACLs, and default ACL inheritance.

Jared has named read/traverse or administrative ACL access where documented. That access does not change the owning service identity and should not be replaced by broad group membership or world-writable modes.

Smart Home Compose files remain owned by `smart-home:smart-home` with their documented restrictive modes. Unintended Game Servers ACL entries were removed without changing service access.

## 3. NAS Numeric Identity

NFSv3 `sec=sys` uses numeric identity. Current storage GIDs are Platform `5002`, Libraries `5000`, Hosted Websites `5004`, Game Servers `5001`, and Smart Home `5003`.

The NAS commonly displays owner UID `988` on export roots. Rotom may map that number to an unrelated local name; interpret it numerically. Do not recursively rewrite NAS ownership merely to make displayed names look familiar.

Use the owning service identity for ordinary service-specific administration. Root, sudo, and Docker administrators remain privileged across these boundaries.

## 4. Application-Specific Boundaries

- Paperless application processes use Libraries identity; authoritative media is `/mnt/nas-libraries/paperless`.
- qBittorrentVPN uses Libraries storage identity and a fail-closed NFS/sentinel contract.
- Palworld uses Game Servers identity and VM-local state.
- Home Assistant and Homebridge use Smart Home identity and VM-local state.
- Aloha and Jared Wines active content is owned within Hosted Websites roots.
- NPM retains its image-default container identity; do not add a user override or Docker socket without a separate design review.

## 5. Administrative and Credential Boundaries

- RS GitHub operations run as `jared`; do not copy Jared's private key into a service account.
- Protected Restic credentials remain root-only and are never documented by value.
- Service secrets remain under `/srv/rotom/secrets` with restrictive ownership and modes.
- Remote Desktop Commander remains locked, has no sudo or Docker membership, and uses its existing outbound service registration without exposing its device state.
- Fran's SSH-key setup remains deferred; password/sudo and PVE browser access are separate from SSH authorization.

## 6. Retired Identities and Paths

Legacy local identities and homes named `documents`, `downloaders`, `filesync`, `customapps`, `auth`, `infra`, `web`, `gameserver`, `smarthome`, `media`, and `game` are absent. Their historical numeric mappings and paths remain available through Git history and the protected change log.

Do not recreate a legacy account or home from historical documentation. First identify the current owning domain, storage GID, Compose path, recovery source, and whether NAS-side data still exists.

## 7. Reconstruction-Critical Information

- Preserve numeric UID/GID contracts rather than names alone.
- Preserve locked service-account status.
- Preserve root-only secret and backup credential boundaries.
- Preserve setgid and ACL behavior on service storage.
- Preserve Docker-group membership as an explicit root-equivalent grant.
- Preserve Jared's RS/GitHub identity separation from service accounts.
- Verify selected recovery material before changing an account, ownership, or home path.

## 8. Outstanding / Needs Verification

- Fran's optional SSH-key authorization.
- Exact owner intent for any remaining root-only rollback artifact.
- NAS-side metadata persistence after a future NAS reconfiguration or update.
- Any proposed new service identity, storage GID, or privilege grant.

## 9. Related Documentation

- [02 — Docker Services](02-Docker-Services.md)
- [04 — NAS and Storage](04-NAS-and-Storage.md)
- [05 — Backup and Restore](05-Backup-and-Restore.md)
- [06 — Maintenance and Automation](06-Maintenance-and-Automation.md)

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate after an account, group, UID/GID, home, shell, lock state, sudo rule, Docker membership, ACL, ownership, service process identity, secret boundary, SSH authorization, or NAS permission contract changes.
