---
id: redesign-dev.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/dev.tmp.md
rules:
  - @~/{{ProjectRoot}}/frontend/ar.ar.rules.md
  - @~/{{ProjectRoot}}/backend/ar.ar.rules.md
---

## Purpose

Judge a dev-side redesign — the single decision point for any contract change that is not a review rework. One dev unit returns (an AR Review design-origin finding, a DE block, a Verification fail, or an Integration doubt), and this decides per pair: `re-architecture` (impact spreads beyond the pair, or the decomposition is unsound), `dev redesign` (this unit's contract must change), or `test-only` (the contract holds; only its test was at fault). On `dev redesign`, re-derive the unit's contract as architect-story would, set its Baseline for rollback, and mark dropped boundary files for removal. The judgment is AR's; the test side never decides a dev redesign.

## Workflow

### 1. Redesign Intake

1. Read the returned dev document — Files, Public Signatures, Schemas, Behavioral Contract, `covers-acs`, `test-kind` — and the return reason in its Handoff Note (which gate sent it, the observed fact, and whether the return is flagged decomposition-wide).
2. Read the paired test document and its Handoff Note — the test-side observation that may have triggered the return.
3. Read the Story AC body for every `covers-acs` ID, the architecture spec, and the prior contracts and cross-dev artifacts this unit touches — enough to judge whether a correct fix stays inside this pair.

**Checklist:**

- [ ] Returned dev document and return reason read
- [ ] Paired test document and its observation read
- [ ] AC bodies, architecture spec, and touched contracts read

### 2. Impact Classification

A return already scoped to the decomposition — a merge-hook failure, or a finding the returning gate flagged decomposition-wide — is `re-architecture` directly; the per-unit read below does not apply. Otherwise decide the outcome by how far a correct fix reaches:

- **Impact spreads beyond this pair** — other units' contracts or the decomposition itself must change to fix this, or the decomposition is unsound → `re-architecture`. Stop; the reset and its Story note are the persona's.
- **The fix is this unit's contract** — a signature, behavior, schema, or boundary file must change, contained within this pair → `dev redesign`. Continue to Step 3.
- **The contract holds** — this unit correctly realizes its ACs; the returned observation originates in the test design, not the contract → `test-only`. Stop; no dev document change.

A single returned unit can still be `re-architecture`: the input is one pair, but the judgment is the blast radius.

**Checklist:**

- [ ] Outcome chosen by blast radius, not by which gate returned the unit
- [ ] `re-architecture` when the fix reaches other units or the decomposition is unsound
- [ ] `test-only` only when the contract is intact and the fault is the test design's
- [ ] `dev redesign` scoped to this unit's contract

### 3. Contract Re-derivation

`dev redesign` only. Re-derive the unit's contract against the corrected understanding, at contract level exactly as architect-story drafts — Files, Public Signatures, Schemas, Behavioral Contract; bodies, pseudocode, private helpers, and internal state forbidden.

1. **Baseline** — set it to the commit holding this unit's most recent implementation (via git history); the implementer diffs against it to roll back what the new contract drops. `—` only if it was never implemented.
2. **Files** — a boundary file the new contract no longer carries gets Action `remove`; a kept file stays `modify`; a new one `create`.
3. **ar-integration unit** — it has no DE: re-derive its wiring surfaces and re-implement them in place across the AR-rw files per rules, keeping `Implemented` / `Reviewed` `[N/A]`. A business-logic unit's body is left for DE.

Validate DE self-containment: no unrestated Story / UX-spec reference. On failure → restate and re-check.

**Checklist:**

- [ ] Baseline set to the most recent prior implementation commit, or `—`
- [ ] Contract level only — no body, pseudocode, private helper, or internal state
- [ ] Dropped boundary files marked `remove`; kept `modify`; new `create`
- [ ] ar-integration re-derived and re-implemented in place, `[N/A]` preserved; business-logic body left for DE
- [ ] Dev document self-contained for DE

> Impact reaches beyond the pair, or the decomposition is unsound → outcome `re-architecture`
> This unit's contract must change → outcome `dev redesign`
> The contract holds; only its test was at fault → outcome `test-only`

## Instructions

- AR is the single judge of a dev-side redesign. A review or verification step only signals "cannot proceed"; the decision of what changes — contract, test, or the whole decomposition — lives here, where the full Story and architecture context is held.
- Classify by blast radius, not by origin gate. The same returned unit is `dev redesign` if a contract fix contains it and `re-architecture` if the fix ripples; the gate that returned it does not decide this.
- The output of `dev redesign` is a corrected contract, not a diff — write the contract as it should now stand; the prior is a reference. The change rationale belongs in the Handoff Note.
- Rolling back the prior implementation happens at implement (business-logic) or in place here (ar-integration) via the Baseline commit; for the business-logic case this task only sets the Baseline correctly.
- `test-only` touches no dev contract — the contract is right; the test side re-derives against it.
- Outcome only — DoD check-off, owner rename, the reset, and Handoff Notes are the persona's; this task ends at the classified outcome.
- Task outcomes: `re-architecture` | `dev redesign` | `test-only`
