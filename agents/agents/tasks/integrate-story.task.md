---
id: integrate-story.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/ar.ar.rules.md
  - @~/{{ProjectRoot}}/backend/ar.ar.rules.md
---

## Purpose

Gate a verification-passed story for merge — the last architecture judgment before its work joins the integration branch. Two checks, both review and never re-execution: the **merge-hook** (is the composed decomposition eligible to merge into the integration branch — clean assembly, architecture invariants intact across the whole, no cross-story structural conflict), and the **E2E review** (does the E2E code QA authored at verification realize the designed journeys with real, observable coverage). Verification already ran the integration, contract, E2E, and regression suites green; re-running them in the same environment adds no information and a differing verdict would be noise, so this trusts that green and re-runs nothing. Judge eligibility and review-soundness; classify to one outcome.

## Workflow

### 1. Intake

1. Read the Story — AC bodies and Context — and confirm the story's pairs are all `_AR` with `Verified` checked (verification-passed and assembled in the worktree).
2. Read the story suite `test-XXX-00` — its `e2e` scenario designs (the journeys QA was to author) and AC Coverage — and the E2E code authored at verification (location per rules).
3. Read each dev document's Position and Files — the ownership zones and boundary files the composition spans — and the integration branch the story merges into.

**Checklist:**

- [ ] Story AC bodies and Context read; all pairs confirmed `_AR` `Verified`
- [ ] Suite `e2e` scenario designs and the authored E2E code read
- [ ] Composition's ownership zones / boundary files and the integration-branch target read

### 2. Merge-Hook

Judge whether the assembled story is eligible to merge into the integration branch — an architecture judgment, not a test run.

1. Assess the composition against the integration branch per rules: it assembles cleanly (no unresolvable conflict with work already integrated), the architecture invariants hold across the whole (ownership boundaries, dependency rules, prototype-production mirror), and no pair left a cross-unit structural contradiction the unit gates could not see.
2. Eligible → Step 3. Not eligible → the decomposition cannot integrate, and the fault is not attributable to any one pair → outcome `merge-hook fail`, stating the observed structural facts.

**Checklist:**

- [ ] Clean assembly against the integration branch confirmed per rules
- [ ] Architecture invariants confirmed across the whole composition
- [ ] Ineligibility reported as `merge-hook fail` with observed structural facts, not pair-attributed

### 3. E2E Review

Review the E2E code QA authored at verification against the suite's `e2e` scenario designs — review, never re-run; verification already ran it green.

1. **Completeness** — every `e2e` scenario the suite designed has corresponding code, and every deferred AC the suite routed to E2E is exercised by it.
2. **Journey fidelity** — the code drives the real running stack end to end and asserts observable terminal state, not a shortcut through a mocked seam nor an assertion on mechanism. A journey that only appears to traverse the stack is a finding.
3. Sound → Step 4. Unsound (a designed journey unimplemented, a deferred-at-E2E AC unexercised, a shortcut or mechanism assertion) → the verification's E2E green cannot be trusted, though the dev bodies it ran against are sound → outcome `e2e-review inadequate`, citing each finding.

**Checklist:**

- [ ] Every `e2e` scenario has corresponding code; every deferred-at-E2E AC exercised
- [ ] Each journey drives the real stack and asserts observable terminal state
- [ ] Findings cited by scenario / AC / location; no re-run performed
- [ ] Unsoundness reported as `e2e-review inadequate`, dev bodies left untouched

### 4. Classify

- Merge-eligible and E2E review sound → `Pass`.
- Not merge-eligible → `merge-hook fail` (decomposition-wide, observed structural facts).
- E2E review unsound → `e2e-review inadequate` (the findings; dev pairs unaffected).

**Checklist:**

- [ ] Result classified to exactly one outcome
- [ ] `merge-hook fail` states structural facts and is not pair-attributed
- [ ] `e2e-review inadequate` cites findings and leaves dev pairs untouched

> Merge-eligible and E2E review sound → outcome `Pass`
> Composition cannot merge into the integration branch → outcome `merge-hook fail`
> E2E verification cannot be trusted on review → outcome `e2e-review inadequate`

## Instructions

- Review and judgment, never re-execution. Verification ran every suite green; a second run in the same environment cannot add information, and a differing verdict would be noise. Trust the green; judge what review alone decides — merge-eligibility and E2E soundness.
- The merge-hook is an architecture judgment, asking whether the composed decomposition can join the integration branch — never whether a unit behaves (settled at its gate) or the story behaves (settled at verification). A failure here is the decomposition's, not a pair's, and routes to re-architecture.
- E2E soundness is coverage and reality, not a re-run. The one gap verification can honestly leave is a shallow E2E — green but not truly traversing the stack or not covering its deferred ACs — caught by reading the code against the designed journeys, and routed back to re-verification without disturbing the dev bodies.
- Outcome only — DoD check-off, owner rename, and the Handoff Note are the persona's; this task ends at the classified outcome.
- Task outcomes: `Pass` | `merge-hook fail` | `e2e-review inadequate`
