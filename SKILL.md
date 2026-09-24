---
name: handoff-faber-rigor
description: >-
  Use for handoffs between implementer (Faber) and deep reviewer (Rigor). Faber
  delivers intent, diff scope, how verified (commands+outputs), what was NOT
  tested, known risks. Rigor refuses review without that packet and returns
  Approve / Approve-with-follow-ups / Request-changes with evidence. Deep code
  review is Rigor-only; Faber does not self-certify.
---
# Handoff: Faber ↔ Rigor

Protocol between **Faber** (implementer) and **Rigor** (deep reviewer). Separates building from certifying. Faber does not Approve their own work. Rigor does not review fog.

## Roles

| Role | Does | Does not |
|------|------|----------|
| **Faber** | Implement; run Verified checks; write handoff packet; fix Request-changes | Self-certify; skip NOT tested; invent metrics |
| **Rigor** | Deep review (`fail-closed-review` + contract/migration/adversarial as needed); verdict with evidence | Implement the fix (unless explicitly dual-hatted); Approve without packet |

## When this skill applies

- End of an implementation chunk before merge.
- Authz, migration, contract, or bench-affecting changes.
- Any PR where "LGTM" without evidence would be the failure mode.
- After `adversarial-qa` HOLE fixes — re-enter handoff before Approve.

## Faber packet (required)

Rigor **refuses review** if any required section is missing or hand-waved.

```text
### Faber handoff packet
1. Intent
   - Problem: <one paragraph>
   - Non-goals: <list>
   - User-visible behaviour change: <yes/no + summary>

2. Diff scope
   - Paths / packages touched: <list>
   - Surfaces: API | MCP | gRPC | CLI | worker | DB | other
   - Out of scope paths deliberately not touched: <list>

3. How verified (Verified only)
   - Commands: <exact>
   - Result files: <absolute paths>
   - Evidence quotes: <N passed / EXIT / bench line>
   - Link skills used: verified-delivery | bench-loop | adversarial-qa | ...

4. NOT tested
   - <explicit list — empty list only if truly none>
   - Include: other doors, Postgres-only, adversarial rows skipped, load, etc.

5. Known risks
   - <fail-open candidates, migration phase, warn vs enforced, resolver drift>
   - Mitigations in flight: <or "none">

6. Ask to Rigor
   - Focus areas: <gates, RLS, contracts, ...>
   - Desired verdict depth: standard | authz-deep | migration-deep
```

### Faber rules

1. Status in packet must match `verified-delivery` (Verified / Written / Failed). Written-only → Rigor may review design but cannot Approve behaviour.
2. Do not say "fully tested" if NOT tested is non-empty.
3. Bench numbers only from files (`bench-loop`); overclaim → fix packet first.
4. After Request-changes: new packet with new result files (no reuse of stale evidence).

## Rigor intake

```text
IF packet missing section → refuse: "Packet incomplete: <section>. Resubmit."
IF Verified evidence absent for behavioural claim → refuse or Request-changes
IF authz change AND no adversarial-qa matrix AND NOT tested omits it → Request-changes or require matrix
ELSE proceed with deep review
```

Deep code review is **Rigor-only**. Faber may run self-checks; those are not a verdict.

## Rigor verdict format

Exactly one primary verdict:

### Approve

```text
### Rigor verdict: Approve
- Evidence reviewed: <files/PR refs>
- Checklist: fail-closed-review <clean|n/a>
- Residual risk: <none | accepted with reason>
- Certification note: measurement ≠ certification if benches cited
```

### Approve-with-follow-ups

```text
### Rigor verdict: Approve-with-follow-ups
- Evidence reviewed: <...>
- Follow-ups:
  - [ ] <item> — owner: <Faber|other> — due: <when>
- No open P0/P1
```

### Request-changes

```text
### Rigor verdict: Request-changes
- Findings: <fail-closed-review blocks or equivalent>
- Blocking: <P0/P1 list>
- Re-review trigger: <what Verified proof is needed>
```

Every verdict cites evidence (paths, packet quotes). "Looks good" is not a verdict.

## Mapping to other skills

| Change type | Rigor must apply |
|-------------|------------------|
| Authz / gates | `fail-closed-review`; prefer `adversarial-qa` matrix |
| Schema / RLS / backfill | `migration-and-data-safety` |
| API/MCP/CLI shape | `contract-and-compat` |
| Bench WIP | `bench-loop` receipts |
| Any behaviour claim | `verified-delivery` |
| Dead / second doors | `reachability-audit` |
| Checker / parity tests | `guards-that-scan` |
| Defect families while reading | `code-that-holds` |

## Failure modes (named)

| Name | Meaning |
|------|---------|
| **self-certify** | Faber Approves own PR |
| **fog-packet** | Intent/diff without Verified evidence |
| **empty-not-tested** | Claims full coverage; hides gaps |
| **stale-evidence** | Old result file after new commits |
| **verdict-without-evidence** | Rigor Approves on skim |
| **role-collapse** | Same message implements and Approves without packet boundary |
| **bench-as-cert** | Approve governance safety from bench % alone |
| **follow-up-parks-P0** | Approve-with-follow-ups used to defer a blocker |

## Minimal happy path

```text
Faber: implement → verified-delivery receipt → fill packet → request Rigor
Rigor: intake gate → fail-closed-review (if authz) → verdict
Faber: if Request-changes → fix → new Verified files → new packet
Rigor: re-review → Approve or Approve-with-follow-ups
```

## Anti-patterns

- "Rigor, quick look?" without packet.
- Faber marking PR `approved` in the tool after fixing nits themselves.
- Rigor rewriting the feature mid-review without switching to Faber role explicitly.
- Using Approve-with-follow-ups to park a P0.

## Checklist

**Faber**

- [ ] Packet sections 1–6 complete
- [ ] Result files Read this turn
- [ ] NOT tested honest
- [ ] No self-Approve

**Rigor**

- [ ] Intake passed
- [ ] Relevant skill checklists applied
- [ ] Verdict is one of the three forms
- [ ] Evidence cited; follow-ups have owners if used
