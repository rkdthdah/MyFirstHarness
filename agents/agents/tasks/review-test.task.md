---
id: review-test.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/qa.qa.rules.md
  - @~/{{ProjectRoot}}/backend/qa.qa.rules.md
---

## Purpose

Check a TE-implemented test against its design — that every designed scenario is implemented, nothing beyond the design is asserted, and the verification is valid — then classify whether a finding originates in the code or the design.

## Workflow

### 1. Intake

1. Read the test design — Scenarios (given/when/then), AC Coverage dispositions, Fixtures, Notes, and the Baseline field. A story suite carries scenarios tagged by level (`integration` / `contract` / `e2e`); its `e2e` scenarios are authored at verification, not implemented here.
2. Read the implemented test code.
3. Work only from these two. Do not read the Story, UX spec, or other units — the design carries its own AC restatements, and review compares code against design, not against the AC. Note whether the design was handed off *executed* or *unexecuted* (subject pending), and whether Baseline is a commit (a redesign — prior test code may need reverting) or `—` (initial design).

**Checklist:**

- [ ] Design scenarios, AC Coverage dispositions, and Baseline read
- [ ] Test code read
- [ ] Executed / unexecuted status of the handoff noted
- [ ] Baseline noted as a commit (redesign) or `—` (initial)

### 2. Conformance Review

Review on three axes. Each finding is a fact decided by comparing code to design — cite the specific axis and location. Subjective code quality, structure, or "a better test would…" is not a finding.

- **Completeness** — every `scenario`-disposed entry in the design has a corresponding test. Dispositions other than `scenario` (`covered-by-ui-test`, `defer-to-verify`, `N/A`) are not implemented here — their absence is not a finding. In a story suite, `e2e`-level scenarios are likewise not implemented at this gate (they are authored at verification) — their absence is not a finding; review the `integration` and `contract` scenarios.
- **Scope fidelity** — the code carries only what the design specifies. An assertion on timing, internal path, or an unmandated call count is a finding (it verifies mechanism, not outcome). Asserting behavior the design does not carry is a finding. When Baseline is a commit, also diff it against the current tree: any change the prior implementation made that the current design no longer carries — an addition, edit, or deletion — left un-restored to its pre-change state is a finding, easy to miss as it predates the latest work.
- **Validity** — the verification runs in the correct environment and through the required infrastructure, per rules. A test reaching its observable surface through a forbidden path verifies under wrong conditions — a finding.

For an **unexecuted** handoff, review the same three axes statically — the code expresses the scenarios, asserts only designed outcomes, and uses the valid environment. Do not pass it merely because nothing ran; behavior that only execution reveals is not this gate's to catch (the implementation-time run is). Judge only what is visible in code against design.

**Checklist:**

- [ ] Every `scenario` disposition checked for a corresponding test
- [ ] Non-`scenario` dispositions, and a suite's `e2e` scenarios, confirmed not implemented (not flagged as missing)
- [ ] No mechanism assertion (timing / internal-path / unmandated call-count)
- [ ] No assertion beyond the design's scenarios
- [ ] When Baseline is a commit, every prior change the current design drops is restored to its pre-change state
- [ ] Environment and infrastructure valid per rules
- [ ] Each finding cites a specific axis and location

### 3. Origin Classification

No finding → `Pass`. For each finding, classify by one question — **does the code faithfully follow the design?**

- **Code diverges from the design** (a designed scenario unimplemented, an assertion the design does not carry, a wrong environment the design did not call for, or a prior change left un-restored) → the code is at fault → `Rework`.

- **Code faithfully follows the design, and the design is what carries the defect** (the design itself specified a mechanism assertion, an unobservable outcome, an unreachable given) → the design is at fault → `Redesign`.

Mixed findings: any `Redesign`-class finding makes the outcome `Redesign` — a design defect must be resolved before re-implementation against it is meaningful. Record the code-class findings in the same note so they are not lost when the design returns.

**Checklist:**

- [ ] Each finding classified `Rework` (code) or `Redesign` (design) by the fidelity question
- [ ] Outcome is `Pass` (no finding), `Rework` (all findings code-class), or `Redesign` (any design-class finding)
- [ ] For `Redesign`, code-class findings recorded alongside

> No finding → `Pass`
> Findings, all originating in the code → `Rework`
> Any finding originating in the design → `Redesign`

## Instructions

- Review compares code to design, never code to the AC — the design is the standard, settled upstream. Re-reading the AC here re-opens design judgment that belongs to design-test and verification.
- QA validates outcomes, not implementation craft. Style, naming, and structure are the implementer's; a finding must name a conformance fact, not a preference.
- The fidelity question is the whole of origin classification: a defect the code introduced is the implementer's to fix; a defect the code faithfully inherited is the designer's. Sending an inherited defect back to the implementer only loops — the implementer has nothing to change.
- An unexecuted test is reviewed more strictly on what is statically visible, never passed for lack of a result. Behavior that only running reveals is caught when the implementation runs against it, not here.
- Outcome only — DoD check-off, owner rename, and the Handoff Note are the persona's; this task ends at the classified outcome.
- Task outcomes: `Pass` | `Rework` | `Redesign`