---
id: manage-qa-spec.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/qa.qa.rules.md
  - @~/{{ProjectRoot}}/backend/qa.qa.rules.md
---

## Purpose

Make one test-infrastructure increment and record it across `qa.spec.md` and the affected rules together — a test library, a `test-utils/` helper, a tooling or environment binding the spec does not yet describe — so every downstream test author works against the fulfilled world. Baseline test infrastructure is set at init (a separate OR line); this adds increments only. Entered inline when a test workflow cannot proceed without the increment, or standalone to record a new pattern or satisfy a user/OR request.

## Workflow

### 1. Increment Intake

1. Identify the increment: inline, the helper / tool / fixture a calling workflow signalled it could not proceed without; standalone, the pattern to record or the request to satisfy.
2. Read the current `qa.spec.md` and the affected `*.qa.rules.md`, plus any prior `test-utils/` helper or fixture the increment touches.

**Checklist:**

- [ ] Increment identified (inline signal or standalone request)
- [ ] Current spec and affected rules read

### 2. Make the Change

Perform the infrastructure per rules — add the test library, author the `test-utils/` helper, register the tooling or environment binding. Test infrastructure does not touch the prototype, so it is added in place and never bounces upstream. Settle it fully before any dependent test or design is produced, so it exists by the time the calling workflow resumes.

**Checklist:**

- [ ] Infrastructure change performed per rules
- [ ] A `test-utils/` helper added where shared use requires it, per rules
- [ ] Change settled before dependent artifacts (no unmet prerequisite left)

### 3. Record and Extract

1. Update `qa.spec.md` so it is the single source — the Tooling, Test Pyramid, `test-utils/` structure, or convention table the change touches. Record the binding only; never copy a universal rule into the spec.
2. Update each affected `*.qa.rules.md` so its consumer sees the increment — agents read rules, not the spec.

**Checklist:**

- [ ] `qa.spec.md` updated where the change lands; no universal rule duplicated
- [ ] Every affected `*.qa.rules.md` updated for its consumer
- [ ] Reference direction intact (rules → spec; nothing references harness)

### 4. Validate

Confirm the increment is coherent per rules — the helper imports within its boundary, the tooling runs, no convention contradicts another.

**Checklist:**

- [ ] Spec-bound checks pass per rules (e.g. the helper's own contract test, dependency check)
- [ ] No contradiction introduced across spec and rules

> One test-infrastructure increment performed and recorded across spec and rules → outcome `Incremented`

## Instructions

- Infrastructure is precisely a change that alters the spec; work within the spec's granted scope is the calling task's to do directly and never reaches here.
- Spec and rules move together — a binding in the spec but not extracted to the consuming rules is invisible to the test author who needs it; the two are updated in one pass.
- Record the binding, not the principle — the universal rule stays canonical and is referenced, never copied.
- Test infrastructure adds in place — it touches no prototype and never bounces upstream (distinct from a runtime-library introduction).
- The increment is settled before the calling workflow resumes — the test or design it then writes assumes the fulfilled world.
- Outcome only — the commit (inline → story worktree; standalone → agents worktree) and session log are the persona's; this task ends at the outcome.
- Task outcome: `Incremented`
