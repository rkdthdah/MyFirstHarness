---
id: revise-test-design.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/test.tmp.md
rules:
  - @~/{{ProjectRoot}}/frontend/qa.qa.rules.md
  - @~/{{ProjectRoot}}/backend/qa.qa.rules.md
---

## Purpose

Re-derive a test design against the current contract after a review found it at fault, a dev redesign changed the contract, or TE reported it unimplementable. Two inputs: a paired test (re-derive its scenarios, and revise the story suite where this unit's change reaches integration) or the story suite itself (re-derive its integration and contract scenarios). The output is a corrected design in the form design-test produces. When no scenario can be designed against the current contract, report the observed block.

## Workflow

### 1. Revision Intake

Branch on what returned.

**A paired test** (`test-XXX-NN`, NN ≥ 01):

1. Read the prior test design — Subject, Scenarios, AC Coverage, Fixtures, Notes — and the return reason in its Handoff Note.
2. Read the paired dev document as it stands now — `covers-acs`, Behavioral Contract, Files / Public Signatures / Schemas. The contract may have changed; the current dev document governs.
3. Read the Story AC body for every current `covers-acs` ID, and the Story / UX spec where outcome validation requires.

**The story suite** (`test-XXX-00`):

1. Read the prior suite design — Scope, Scenarios with level tags, AC Coverage, Fixtures, Notes — and the return reason.
2. Read every dev document the suite's scenarios span, as they stand now — the composition or contract may have changed.
3. Read the Story AC body for every deferred AC the suite covers.

**Checklist:**

- [ ] Prior design and return reason read
- [ ] Current governing documents read (paired: the dev; suite: the spanned devs)
- [ ] AC bodies read for the current covered ACs
- [ ] Story / UX spec consulted, or N/A

### 2. Coverage Mapping

**Paired** — assign every current `covers-acs` ID a disposition (`scenario` / `covered-by-ui-test` / `defer-to-verify` / `N/A`) against the current contract; a prior disposition no longer binds. A facet that is now `defer-to-verify` is the story suite's to carry — note it for Step 4.

**Suite** — confirm every deferred AC the suite owes still maps to a suite scenario under the current composition; drop scenarios whose AC is no longer deferred here, add those newly deferred.

Record in the AC Coverage table.

**Checklist:**

- [ ] Paired: every current `covers-acs` ID dispositioned; newly `defer-to-verify` facets noted for the suite
- [ ] Suite: every owed deferred AC mapped to a scenario; stale scenarios dropped
- [ ] `covered-by-ui-test` cross-referenced against Story UI Test Coverage
- [ ] Excessive `defer-to-verify` recorded as a possible decomposition issue, or N/A

### 3. Scenario Design

For each `scenario`-disposed behavior (paired: at the unit boundary; suite: at integration / contract / journey level, each level-tagged), write a given/when/then asserting the observable terminal state. Link each to its AC ID(s) with a one-line restatement.

If a behavior cannot be expressed observably against the current contract — unobservable terminal state, signatures that cannot produce the *given*, or no behavior to design from — it is an observed block. Even one blocks this design; scenarios drafted for its other ACs are discarded. End the task, stating the observed fact. Outcome `Design Blocked`. Scope is this unit (paired) or the spanned units (suite) only.

**Forbidden in scenarios:** assertion code or mock implementation; timing or internal-path assertions; mock call-count assertions unless the contract mandates the count.

**Checklist:**

- [ ] Each `scenario`-disposed entry has a given/when/then, or is reported as an observed block
- [ ] Each scenario links to its AC ID(s) with a one-line restatement
- [ ] Scenarios assert observable terminal state only; nothing forbidden present

### 4. Handoff Composition

Compose the corrected document per template.

- **Baseline** — via git history, the commit of this document's most recent implementation; `—` if it was never implemented
- **Subject** — paired: file / export from the current dev document; suite: the units, boundaries, contracts, journeys under verification
- **Fixtures** — what the scenarios require at outcome level; tooling per rules. A fixture needing a test-util the spec does not describe is recorded as a spec-increment need and authored as if present
- **Notes** — cross-unit sequencing, edge cases not captured as scenarios, dependencies

**Paired only — suite impact:** if this revision changed what this unit defers to integration (a facet now `defer-to-verify`, or one no longer deferred), revise the story suite `test-XXX-00` to match — add, drop, or adjust the affected scenarios; leave the rest of the suite untouched. Where the suite's prior implementation must change, set its Baseline too. If the change is none, leave the suite as is.

Validate TE self-containment: no unrestated Story / UX-spec / dev-unit reference. On failure → return to Step 3.

**Checklist:**

- [ ] Baseline set to the most recent prior implementation commit, or `—`
- [ ] Subject correct for the mode (paired export / suite composition)
- [ ] Fixtures at outcome level, no tooling specifics
- [ ] Paired: story suite revised where this unit's deferral changed, or confirmed unaffected
- [ ] Document self-contained for TE

> Scenarios re-derived → outcome `Designed`
> A behavior cannot be expressed observably against the current contract → outcome `Design Blocked`

## Instructions

- The output is a corrected design, not a diff — write the scenarios as they should now stand; the prior design is a reference to fix. The change rationale belongs in the Handoff Note.
- A paired revision can ripple into the story suite: when this unit's integration-level facets change, the suite's scenarios for them must change with it. Touch only those; the suite is otherwise other units' shared instrument.
- Rolling back a prior implementation is done at implement time (integration / contract) or at verification (E2E) via the Baseline commit; this task only sets the Baseline correctly.
- QA observes a block, never diagnoses the contract — report `Design Blocked` and stop, neither redesigning against a contract you cannot design to nor proposing its fix.
- Task outcomes: `Designed` | `Design Blocked`