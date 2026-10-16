# 06 - Maintenance and Automation

**Documentation set:** Rotom Runbook
**Document role:** Canonical source for scheduled/routine maintenance, automation, monitoring behavior, and operational administration workflow
**Hosts:** PVE hypervisor `pve` and Debian VM `rotom`  
**Baseline verified:** Current state reconciled through 2026-10-15; see outstanding items for remaining verification
**Documentation updated:** 2026-10-15 — Prepare Rotom Runbook identity and migration gates
**Related canonical sources:** `01-Rotom-Server-Inventory.md`, `02-Docker-Services.md`, `04-NAS-and-Storage.md`, `05-Backup-and-Restore.md`, `07-Users-and-Permissions.md`  
**Index:** [01-Rotom-Server-Inventory.md](01-Rotom-Server-Inventory.md)  
**Change history and update rules:** [00-Rotom-Change-Log.md](00-Rotom-Change-Log.md)

Record substantive changes to this document in the change log as part of the same task, following its maintenance guide.

## Start by Task or Symptom

- Failed timer, missed run, failed oneshot service, or automation drift: use [Investigate a Failed Timer or Automation](#investigate-a-failed-timer-or-automation).
- Rotom Restic execution or repository behavior: use [document 05](05-Backup-and-Restore.md#start-by-task-or-symptom).
- NAS recovery-helper behavior: establish storage state in [document 04](04-NAS-and-Storage.md#diagnose-and-recover-a-configured-nas-mount), then use this document for service/timer scheduling.
- Container lifecycle or health: use [document 02](02-Docker-Services.md#start-by-task-or-symptom).

## Investigate a Failed Timer or Automation

### Outcome, entry conditions, exclusions

Identify whether a supported timer/worker failed because of schedule, dependency, storage, application, or policy state, and retry only the existing owning service when its desired action and safety conditions are established. Exclude new automation, schedule changes, package-policy changes, retention/prune, destructive cleanup, and PVE action not executed by Jared.

### Risk, preflight, and Stop conditions

Inspection is read-only. Starting a service may change Rotom and, for backup workers, protected repository state. Before a retry, identify the exact timer/service pair, last/next trigger, current process state, dependencies, and documented owner. Stop if another run is active, the unit invokes an unreviewed script, required storage or credentials are unavailable, the desired outcome is unclear, or logs show protected-data/repository damage.

**Read-only**

```bash
systemctl list-timers --all --no-pager
systemctl --failed --no-pager
systemctl status UNIT.timer UNIT.service --no-pager
sudo journalctl -u UNIT.timer -u UNIT.service -n 160 --no-pager
systemctl cat UNIT.timer UNIT.service
```

Replace `UNIT` only with the exact documented unit name.

### Smallest targeted action and immediate result

Correct the owning dependency first: document 04 for NFS, document 05 for Restic/repository issues, document 02 for application state, or document 07 for identity/permission state. When current policy clearly supports a retry and no concurrent work exists:

**Changes Rotom**

```bash
sudo systemctl start UNIT.service
```

For `rotom-restic-backup.service`, this label is instead **Changes Rotom and protected storage/repository state** and the complete document 05 procedure is mandatory. For any PVE unit, the action is **PVE — Jared executes**.

### Verification, safe stop, branches, and evidence

Verify the exact service result, journal, dependency state, downstream application outcome, next timer trigger, and `systemctl --failed`. Do not repeatedly retry a deterministic failure. A failed oneshot normally needs no rollback; preserve logs, leave the timer policy unchanged, and stop safely. If a partial action changed state, use the owning document's rollback guidance.

Retain the unit name, trigger time, exit result, sanitized journal error, dependency finding, retry authorization, downstream check, and next trigger. Revalidate after any service, timer, schedule, worker, dependency, update process, monitoring behavior, or automation-policy change.

## Navigation and Authority

- Start with **1. Current Maintenance and Automation** for active timers, helpers, and routine operating behavior.
- Use **5. Command-Line Safety and Administration**, **6. Documentation and Synchronization Workflow**, and **7. RR Validation** for administration and RR workflow.
- Treat retired automation and preserved historical evidence as context, not as active scheduling or execution instructions.
- Review **Outstanding / Needs Verification** before depending on an automation, notification, certificate, or recovery behavior.
- This file owns operational workflow and automation; backup semantics, service deployment, and privileges remain canonical in documents 05, 02, and 07.

## 1. Current Maintenance and Automation

### Rotom guest

| Function | Current implementation |
|---|---|
| NAS-backed Docker recovery | `rotom-nas-docker-recovery.service`; validates and recovers only eligible Libraries consumers |
| Rotom Restic Backup | `rotom-restic-backup.timer`; around `03:00`, persistent, randomized delay up to ten minutes |
| qBittorrent mount guard | `rotom-qbittorrent-media-guard.timer`; validates Libraries NFS/sentinel and stops qBittorrentVPN if unsafe |
| Weekly maintenance | `rotom-weekly-maintenance.timer`; Sunday `01:15`; ordinary package upgrades without automatic reboot |
| Backup status | `rotom-restic-backup-status-api.service`; read-only port `8787` |

The older reboot-capable `weekly-linux-update` script exists, but its cron schedule is intentionally absent. Do not reinstall that schedule merely because the script remains.

### PVE host

| Function | Current implementation |
|---|---|
| PVE Restic Backup | Daily `04:00` service/timer |
| Rotom VM Backup | Native Proxmox job `rotom-vm-daily` at `05:00` |
| Weekly maintenance | `pve-weekly-maintenance.timer`; Sunday `00:30` |
| CPU-temperature status | Read-only API at `192.168.1.68:8788` |
| Backup status | Read-only API at `192.168.1.68:8789` |
| NIC stability / Wake-on-LAN | Interface hooks retain WOL and the documented TSO/GSO workaround |

PVE actions remain Jared-executed where labelled.

## 2. Libraries Recovery Automation

The NAS recovery helper handles the boot-time race in which Docker requests Libraries before the NAS is reachable. It resets failed generated units, arms `mnt-nas\x2dlibraries.automount`, starts or triggers the exact mount, verifies the documented NFS source and required directories, and fails closed if the automount does not remain active.

The helper can recover only supported containers with an eligible restart policy and a recorded NAS/mount startup-failure signature. It leaves running and intentionally stopped containers untouched. Its verified reboot path required the configured retry about thirty seconds after the first pre-NAS attempt.

## 3. Arcane Automation

The latest complete evidence reports `autoUpdate=true`, `autoHealEnabled=true`, `scheduledPruneEnabled=false`, and `followProjectSymlinks=true`.

Arcane automatic activity has recreated or updated containers in the past. Exact project exclusions and intended long-term automation policy remain **Needs Verification**. Do not infer cleanup safety for images, named volumes, or intentionally stopped services from Arcane activity alone.

## 4. Monitoring and Retired Alerting

Homepage, Glances, the Rotom/PVE backup-status APIs, and the PVE CPU-temperature API are current read-only monitoring surfaces.

Uptime Kuma and the Rotom/PVE alert-delivery layer are retired. No worker, service, timer, script, Homepage mapping, or delivery input remains. An empty root-only `/var/lib/rotom-alerting` directory persists; it is not active state and is not authorized for deletion by documentation cleanup.

NPM certificate renewal remains active. Diagnose renewal by inspecting the currently served certificate, NPM state, and logs without exposing protected inputs.

## 5. Command-Line Safety and Administration

- Use small, reviewable command blocks.
- Read before writing.
- Do not expose secrets, protected environment files, repository passwords, private keys, cookies, or tokens.
- Do not use `chmod 777`.
- Keep PVE commands separate and clearly labelled for Jared when required.
- Stop if a command would delete, recursively rewrite, remount, prune, overwrite protected state, or affect more than the approved target.

## 6. Documentation and Synchronization Workflow

Resolve the active checkout with `rr path`. `/usr/local/bin/rr` is the sole canonical installed implementation on Rotom and the Mac. The transitional `retired-helper` wrappers were retired after both environments passed independent RR verification and dependency audits. Repository `AGENTS.md` and document 00 govern RR changes.

Supported helper commands are `path`, `status`, `check`, `lint`, `pull`, `diff`, `log`, `commit`, `push`, and `help`. Do not bypass a helper safety failure with reset, clean, merge, rebase, force-push, or ad-hoc publication.

The Rotom RR checkout intentionally retains the working legacy directory `/home/jared/documentation/rotom-runbook`; `/home/jared/.config/rr/repository` points to it. Rotom Docs publishes from a separate root-owned deployment clone under `/srv/rotom/appdata/platform/rotom-docs/repository`; it does not publish from a human home checkout.

Repository support files are not RR members merely because they are tracked. Available Sources remains the membership boundary and replacement remains manual by Jared.

## 7. RR Validation

The tracked `support/tooling/rr` owns offline lint. Before publication, the maintenance contract requires:

- `support/tooling/test-rr-runbooks`
- `support/tooling/test-rr-cli`
- `support/tooling/rr lint`
- `git diff --check`
- semantic ownership/duplication review
- `rr check` at a clean checkpoint

`rr check` performs a network comparison and requires explicit authorization when the task withholds it.

## 8. Outstanding / Needs Verification

- Arcane exclusions and intended long-term automatic-update/auto-heal policy.
- Root-only rollback archive retention.
- PVE iGPU option-ROM warning and historical reset/NIC stability only if symptoms recur.
- Exact NAS snapshot settings and incident-specific recovery points.
- Fran's deferred SSH-key setup.

## 9. Related Documentation

- [02 — Docker Services](02-Docker-Services.md)
- [04 — NAS and Storage](04-NAS-and-Storage.md)
- [05 — Backup and Restore](05-Backup-and-Restore.md)
- [07 — Users and Permissions](07-Users-and-Permissions.md)

## Maintenance and Revalidation Triggers

Follow the canonical [Runbook-First Maintenance Standard](00-Rotom-Change-Log.md#runbook-first-maintenance-standard). Revalidate after a service, timer, schedule, worker, update policy, monitoring endpoint, maintenance procedure, RR tool, checkout path, or automation dependency changes.
