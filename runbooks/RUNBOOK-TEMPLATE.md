# Runbook: _Action-Oriented Title_

**Document role:** Operational runbook  
**Status:** Draft / Verified / Needs Verification / Retired  
**Applies to:** Exact host, service, or boundary  
**Risk:** Read-only / Low / Moderate / High  
**Authorization:** What the standing request permits and what requires separate approval  
**Last live verification:** `YYYY-MM-DD`, evidence summary  
**Canonical references:** Links to the numbered current-state owners  
**Maintenance contract:** [00-Rotom-Change-Log.md](../00-Rotom-Change-Log.md)

## 1. Outcome

State the observable end condition in one or two sentences. Describe what will work and what must remain unchanged.

## 2. Use this runbook when

- List the symptoms or requests that enter this runbook.
- Identify the supported scope precisely.

Do **not** use this runbook to:

- List adjacent tasks that require another runbook or a new decision.
- Identify ambiguous, destructive, credential-sensitive, or unsupported cases.

## 3. Impact and authorization

State:

- systems and persistent data that may be affected;
- expected interruption, if any;
- whether routine execution is reversible;
- actions that require Jared or separate explicit approval;
- secrets or protected inputs that must not appear in output.

## 4. Canonical current-state inputs

Link to the authoritative reference for each fact needed by the procedure. Repeat only the minimum values required to execute safely. If live state conflicts with a reference, stop and record the discrepancy rather than silently changing either side.

## 5. Read-only preflight

Label every command block. Resolve exact targets and capture enough evidence to choose the correct branch without exposing secrets.

**Read-only**

```bash
# narrowly scoped inspection commands
```

Expected result:

- Define the acceptable state.
- Define which differences are time-sensitive observations rather than errors.

## 6. Stop conditions

Stop before changing state if any of the following applies:

- the exact target or desired state is ambiguous;
- ownership, source, path, identity, or persistent-data location differs from the canonical reference;
- a backup, mount, container, or maintenance operation is already active and would conflict;
- recovery would require deleting, overwriting, recreating, unlocking, force-removing, or exposing protected data beyond the authorization recorded above;
- the preflight reveals an unrelated failure or a broader blast radius than this runbook covers.

Add procedure-specific stop conditions here.

## 7. Procedure

Use the smallest targeted action. Separate read-only checks from state changes and present branches in decision order.

**Changes Rotom**

```bash
# one targeted action or a short fail-fast block
```

State the expected immediate result after each action. Do not combine unrelated recovery steps into one opaque command block.

## 8. Verification

Verification must be observable and proportional to risk.

**Read-only**

```bash
# status, health, storage, route, log, or application checks
```

Success requires:

- exact service or process state;
- exact application, mount, route, or backup result;
- confirmation that persistent data, ownership, and unrelated services remain intact where relevant;
- no unexplained new failures.

## 9. Rollback or recovery

Describe a tested rollback when one exists. If the operation has no meaningful rollback, say so and describe how to stop safely, preserve evidence, and avoid making the incident worse.

Never invent a rollback image, path, archive, snapshot, or credential. Confirm it exists and applies before use.

## 10. Troubleshooting branches

Map each likely observed failure to the next safe check. End every branch with one of:

- resolved and verified;
- return to a named earlier step;
- hand off to another runbook;
- stop and request a decision.

## 11. Evidence to retain

Record only operationally useful, non-secret evidence:

- date and timezone;
- exact target;
- preflight state;
- action performed;
- verification result;
- discrepancies and unresolved work.

Do not treat raw command transcripts as the final record. Summarize the supported result in the canonical subject document and document 00 when an legacy documentation update is authorized.

## 12. Related documentation

- Link to the relevant numbered reference owners.
- Link to adjacent runbooks.
- Link to historical evidence only when it is needed for context or recovery.

## 13. Maintenance notes

State what must trigger revalidation, such as a changed Compose path, mount contract, service identity, backup worker, proxy route, or authorization boundary.

