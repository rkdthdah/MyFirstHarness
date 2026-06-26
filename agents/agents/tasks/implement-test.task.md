---
id: implement-test.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/te.qa.rules.md
  - @~/{{ProjectRoot}}/backend/te.qa.rules.md
---

## Purpose

Implement a QA-designed test design as executable code, faithfully and at outcome level, then classify whether the subject runs — observe, never diagnose.

## Workflow

### 1. Design Intake

1. Read the test design — its Subject (a paired test names one file :: export; a story suite names the units, boundaries, and contracts under test), Scenarios (given/when/then), AC Coverage, Fixtures, Notes for TE.
2. Work only from this design, this task, the consumed rules, and the source under test. The design is authored to stand alone — do not read the Story, UX spec, or other units to fill a gap. A gap that blocks implementation is a design flaw, surfaced in Step 3, never patched from outside.

**Checklist:**

- [ ] Subject identified — the file / export, or the units / boundaries / contracts under test
- [ ] Every scenario's given/when/then read
- [ ] Fixtures and mock strategy read
- [ ] No unrestated external reference required to implement

### 2. Test Implementation

1. If Baseline is a commit (a redesign), first restore: diff it against the current design and undo every change the prior implementation made that the current design no longer carries, returning each affected file to its pre-change state — a prior addition is removed, a prior edit reverted, a prior deletion restored. Baseline `—` (initial design) skips this. Work from the referenced commit and git history, not from memory of what changed.
2. For each scenario, implement test(s) per rules — arrange the *given*, trigger the *when*, assert the *then* as an observable terminal state. In a story suite, implement the `integration` and `contract` scenarios only; skip any `e2e` scenario — E2E is authored at verification, not here.
3. Implement the design as written — no scenario added, dropped, merged, or re-scoped. Assert terminal state only; never timing, internal path, or call counts the design does not mandate.
4. Author against the fulfilled world — assume the subject and any designed fixtures exist. Never write "not yet implemented" into the test; a pending subject is a Step 3 signal, not test content.

**Checklist:**

- [ ] When Baseline is a commit, every prior change the current design drops is undone to its pre-change state (additions removed, edits reverted, deletions restored)
- [ ] One or more tests per scenario; none added or dropped
- [ ] In a story suite, `e2e` scenarios skipped (integration / contract implemented)
- [ ] Assertions are on observable terminal state only
- [ ] No timing / internal-path / unmandated call-count assertions
- [ ] No unmet-prerequisite note written into the test

### 3. Run & Classify

1. Execute the suite per rules.
2. Determine first whether the subject runs, then classify — always by what is observable, never by inferred cause:
   - **Subject does not import or compile** (implementation unwritten) → `Implemented`, flagged *unexecuted*. The test is authored and committed but never ran, so its logic is unverified; a red-but-pending subject is the expected state, not a defect. Own test-code errors are not observable here — leave the file well-formed and hand off.
   - **Subject runs** → the test executes. Classify the run:
     - own assertion or setup error → self-correct, return to Step 2 (not a handoff state)
     - passes, or fails against an implemented subject → `Implemented`
   - **Subject is present but the design cannot be implemented against it** — an observed structural impossibility (an export contradicting the stated signature, a terminal state unobservable at the boundary, a *given* unreachable from design and rules, or a designed test-util that is absent and TE cannot author) → `Redesign Requested`, stating the observed fact only.

**Checklist:**

- [ ] Suite executed
- [ ] Own test-code defects self-corrected (no unresolved author error where the subject runs)
- [ ] Result classified to exactly one outcome
- [ ] `unexecuted` and `Redesign Requested` each carry an observed fact, not a diagnosed cause

> Suite executes, or subject is pending → outcome `Implemented` (flagged *unexecuted* when subject pending)
> Design unimplementable (observed structural impossibility) → outcome `Redesign Requested`

## Instructions

- Observe, never diagnose. TE reports whether the subject runs; *why* it does not is for the owner who holds the full context to judge — that owner always re-evaluates, so a mis-signal never stalls the flow.
- Faithful to the design — design quality is settled at review, not here. Implementing it as written is the contribution.
- Author the fulfilled world. A pending or unrunnable subject is signalled as the outcome, never recorded in the test as "does not exist yet."
- A red-but-runnable test with implementation pending is the expected state, handed off as `Implemented`. Green is not the bar TE clears — observability is.
- A stable stop is a well-formed test file, not a green project; project-level typecheck and test convergence are the integration step's, not a TE stop condition.
- Outcome only — DoD check-off, owner rename, and the Handoff Note are the persona's; this task ends at the classified outcome.
- Task outcomes: `Implemented` | `Redesign Requested`.