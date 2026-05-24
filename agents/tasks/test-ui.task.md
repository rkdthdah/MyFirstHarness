---
id: test-ui.task
level: 3
owner: or
---

## Purpose

Verify the prototype reflects AC and surface UX spec gaps before AR architecture. UI tests are the verification artifact.

## Workflow

### 1. Coverage Mapping

1. Read every AC body, Domain Terms, External Dependencies, User-Facing Notes
2. Read UX spec — data states, interactions, validation, navigation, prototype reference
3. For each AC:
   - **Test level** —
     - Component when the signal is within a single UI unit's render or event handling
     - Integration when the signal requires multi-unit composition or view-level navigation
   - **Subject** — UI unit at file granularity (one component, one view), not a region
   - **State/flow** — concrete UX spec row or interaction sequence
   - **UX spec anchor** — trigger and signal both resolve to specific UX spec rows
4. AC outside UI scope → mark `N/A` with reason
5. Mapping defects → carry into Step 4 as `Revision Required` triggers:
   - **No anchor** — AC trigger or signal not locatable in UX spec
   - **Contradiction** — AC signal diverges from UX spec for the same trigger
   - **Ambiguous mapping** — AC resolves to multiple unrelated UX spec rows
   - **Prototype divergence** — UX spec row exists, prototype does not match
   - **Excess UX spec** — UX spec defines a state/flow with no AC mapping and not declared out of story scope

**Checklist:**

- [ ] Every UI-relevant AC mapped to level, subject, state/flow, UX spec anchor
- [ ] N/A AC has a reason
- [ ] Mapping defects identified and recorded

### 2. Test Implementation

1. For each mapping row, implement test(s) per qa rules:
   - **Component test** verifies: rendered output for the mapped state, user-event handling, accessibility
   - **Integration test** verifies: view-level user flow across composed units, navigation, mock-data-driven state transitions
2. Defect surfaced during implementation → return to Step 1

**Checklist:**

- [ ] One or more tests per mapping row
- [ ] No defect unresolved

### 3. Run & Verify

1. Execute the test suite per qa rules
2. Classify each failure:
   - **Test code defect** — assertion or setup error from test author → return to Step 2
   - **Spec defect** — test correctly exercises the mapped state but the AC outcome can't be observed (source is AC, UX spec, or prototype) → return to Step 1
3. All tests pass → proceed

**Checklist:**

- [ ] All UI tests pass

### 4. Coverage Documentation

Fill Story `UI Test Coverage`. One row per AC:

- `AC ID` — AC identifier, or `—` for gap rows not tied to a single AC
- `Test File` — test file path(s), or `—` for N/A AC and defect rows
- `Coverage` — what the test verifies for this AC; for N/A AC or defect rows, the reason or defect criterion

**Checklist:**

- [ ] Every AC has a row
- [ ] N/A rows carry reason in `Coverage`
- [ ] Defect rows cite a specific criterion in `Coverage`
- [ ] Story file saved

> Any unresolved spec defect → outcome `Revision Required`
> All AC covered, all tests pass → outcome `Completed`

## Instructions

- Bias toward `Revision Required` — one PM revision is cheaper than AR redesigning off a contradictory UX spec or insufficient prototype.
- Test code defect is self-correction. Any AC / UX spec / prototype defect is `Revision Required`.
- Task outcomes: `Revision Required` | `Completed`