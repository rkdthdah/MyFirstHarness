---
id: verify-implementation.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/test.tmp.md
rules:
  - @~/{{ProjectRoot}}/frontend/qa.qa.rules.md
  - @~/{{ProjectRoot}}/backend/qa.qa.rules.md
---

## Purpose

Verify a fully-implemented story at the integration level — what no single unit can show once the units are combined. Run the story suite's integration and contract scenarios (TE-implemented, reaching their first full-composition execution here), author and run its E2E scenarios (designed upstream, implementable only now that the whole stack exists), and re-run the accumulated integration and E2E suites for regression. Unit behavior is settled at its gate and review; this does not repeat it. Observe and report which checks fail and which pairs they run through; never diagnose which unit is at fault or why.

## Workflow

### 1. Scope Intake

1. Read the Story — AC bodies (the full intent behind each deferred AC) and Context (existing behavior this story changes, the baseline for regression).
2. Read the story suite `test-XXX-00` — Scope, Scenarios with their level tags (`integration` / `contract` / `e2e`), AC Coverage, Fixtures & Environment, and the Baseline field.
3. Read every dev document — `covers-acs` and the verification surfaces of each `ar-integration` unit.
4. Assemble what must be verified: the deferred AC set this story owes integration — every AC dispositioned `defer-to-verify` across the paired tests, plus every `ar-integration` surface — and confirm each maps to a suite scenario (or surface) to run. An owed AC with no scenario is a coverage gap, recorded as a finding in Step 5. Then assemble the regression scope per rules — the integration and E2E suites accumulated across the codebase.

**Checklist:**

- [ ] Story AC bodies and Context read
- [ ] Suite scenarios read, grouped by level; Baseline noted as a commit (redesign) or `—`
- [ ] All dev documents read (covers-acs, ar-integration surfaces)
- [ ] Deferred AC set assembled; each mapped to a suite scenario or surface, gaps noted
- [ ] Regression scope assembled

### 2. Integration & Contract Execution

The suite's `integration` and `contract` scenarios are already implemented (by TE) but have not run against the full composition until now — every unit they span exists only at this point.

1. Execute them per rules. Confirm each `ar-integration` surface is active in the running system, and each contract holds against real interaction at the boundary.
2. A scenario cannot run because the spec lacks an increment it requires — a harness, tool, or environment the spec does not describe → record the spec-increment need and stop for the workflow to resolve; this is a "cannot verify" signal, not a `Fail`.

**Checklist:**

- [ ] Every `integration` / `contract` scenario executed against the full composition
- [ ] Every ar-integration surface confirmed active
- [ ] Contracts exercised against real interaction at their boundaries
- [ ] A missing spec increment recorded as a need, not classified `Fail`

### 3. E2E Authoring & Execution

The suite's `e2e` scenarios were designed upstream as journeys, not code — author and run them now, against the running stack.

1. If the suite Baseline is a commit (a redesign), first restore the E2E code: diff it against the current suite design and undo every change the prior E2E implementation made that the current design no longer carries, returning each affected file to its pre-change state. Baseline `—` (the suite never reached implementation) skips this. Work from the referenced commit and git history, not from memory. (The `integration` / `contract` code was already restored against this same Baseline at implementation.)
2. For each `e2e` scenario, author the journey as test code per rules — arrange the *given*, drive the *when*, assert the *then* as an observable terminal state across the running stack. Implement the designed scenarios as written: none added, dropped, or re-scoped.
3. Execute them.

**Checklist:**

- [ ] When Baseline is a commit, prior E2E changes the current design drops are restored to pre-change state
- [ ] Every `e2e` scenario authored as designed; none added or dropped
- [ ] E2E assertions are on observable terminal state only
- [ ] All `e2e` scenarios executed against the running stack

### 4. Regression

Re-run the accumulated integration and E2E suites across the codebase per rules — the story's own scenarios plus every prior story's, against the changed tree. Existing behavior the Story did not set out to change must still hold.

**Checklist:**

- [ ] Accumulated integration and E2E suites re-run per rules
- [ ] Regressions in existing behavior identified as failed checks

### 5. Classify

- Every deferred AC is covered by a scenario or surface, and every check — suite scenarios, ar-integration surfaces, regression — passes → `Pass`.
- A deferred AC has no scenario or surface covering it (a coverage gap from Step 1), or any check fails → `Fail`. State the observed facts: which AC is uncovered, or which scenario, surface, contract boundary, or existing behavior failed, and the pairs that gap or failing check runs through (the AC's `covers-acs`, the units on either side of a contract boundary, or the ar-integration unit whose surface failed). Report those pairs as implicated. Where it cannot be traced to specific pairs — a story-wide or cross-cutting break, common for an E2E journey — report the whole story as implicated. Narrow only when the trace is clear; otherwise the whole story. Report what is uncovered or failed and what it runs through, not which unit is at fault or why.

**Checklist:**

- [ ] Every deferred AC confirmed covered by a scenario or surface
- [ ] Result classified to exactly one outcome
- [ ] `Fail` states the coverage gap or failed checks, as observed facts
- [ ] Implicated pairs reported by trace, or the whole story where the trace is unclear
- [ ] No unit named as at-fault and no cause given

> Every deferred AC is covered and every integration-level check passes → outcome `Pass`
> A deferred AC is uncovered or any check fails → outcome `Fail` (stating the gap or failed checks and the implicated pairs as observed facts)

## Instructions

- Integration level, not unit. Unit behavior is settled at its gate and review; this step verifies only what emerges once units integrate — the suite's scenarios, ar-integration surfaces — and that nothing regressed. Do not re-derive or re-judge a unit's own scenarios.
- Observe, never diagnose. Report which checks failed and which pairs they run through — both are observed by running the verification. Which pair is at fault and why is the owner's judgment at redesign. A failure surviving to here is not a coding slip — each unit passed its own gate, review, and contract — so it points to a design fault, never a rework.
- Execution task, not authoring of design. The E2E *code* is authored here because the stack exists only now, but its *scenarios* are the upstream design — implement them as written, do not redesign. Integration and contract scenarios are run, not re-authored.
- Self-sufficient verification. Regression is this step's own full re-run, not a dependency on outside infrastructure; an increment the spec does not describe is a "cannot verify" signal for the workflow to resolve, never a `Fail`.
- The form of each check — integration, contract, E2E, regression: tooling, boundaries, environment, how a contract is exercised, regression scope — is per rules.
- Task outcomes: `Pass` | `Fail`