---
id: manage-qa-artifact.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/qa.qa.rules.md
  - @~/{{ProjectRoot}}/backend/qa.qa.rules.md
---

## Purpose

Bring the QA-owned project-level artifacts a story touched up to date once its tests close — the cross-story coverage map and catalogs that let the next test author see what is already verified instead of re-deriving it. Project-level, not story-level: these live in the agents worktree and accumulate across stories. Entered at story completion (every dev `Complete`, every test `Reviewed` or `[N/A]`) or on a user/OR request.

## Workflow

### 1. Scope Intake

1. Identify what the completed story added or changed that a QA-owned project artifact tracks — newly covered ACs, the units and journeys now verified, deferred-then-verified coverage.
2. Read the project artifacts that track them (the coverage map, any test catalog), located per rules.

**Checklist:**

- [ ] Story's QA-tracked additions / changes identified
- [ ] The project artifacts that track them read (per rules)

### 2. Update

Bring each affected artifact current per rules — record the newly covered ACs and verified surfaces, update changed entries, drop what the story removed. Each stays a lookup table: enough to decide what is already covered, never a copy of the test code.

**Checklist:**

- [ ] Coverage map / catalog updated per rules
- [ ] Entries are lookup-level — no content the test code already holds
- [ ] Removed coverage dropped from its tracker

### 3. Validate

Confirm each artifact is internally consistent and points at tests that exist.

**Checklist:**

- [ ] Every tracked entry resolves to an existing test or covered AC
- [ ] No stale entry left from this story's changes

> QA-owned project artifacts brought current → outcome `Maintained`

## Instructions

- Project-level, not story-level — these artifacts span stories and live in the agents worktree; the story's own test documents are marked `Complete` by the persona, not here.
- A tracker is a lookup, never a second source of truth — record what is needed to see and reuse coverage, never a copy of the test code.
- Coverage is reported, not gated, unless the spec says otherwise — this maintains the map; it does not impose a numeric gate the spec has not set.
- Outcome only — the commit (agents worktree) and session log are the persona's; this task ends at the outcome.
- Task outcome: `Maintained`
