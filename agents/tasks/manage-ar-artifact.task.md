---
id: manage-ar-artifact.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/ar.ar.rules.md
  - @~/{{ProjectRoot}}/backend/ar.ar.rules.md
---

## Purpose

Bring the AR-owned project-level artifacts a story touched up to date once it integrates — the cross-story indexes, catalogs, and shared references that let the next architect look up what exists instead of scanning. Project-level, not story-level: these live in the agents worktree and accumulate across stories. Entered at integration (W5 Pass) or on a user/OR request.

## Workflow

### 1. Scope Intake

1. Identify what the integrated story added or changed that an AR-owned project artifact tracks — new contracts and types, registered routes or providers, entry-points, digested external-dependency references.
2. Read the project artifacts that track them (indexes, catalogs, the external-reference store), located per rules.

**Checklist:**

- [ ] Story's AR-tracked additions / changes identified
- [ ] The project artifacts that track them read (per rules)

### 2. Update

Bring each affected artifact current per rules — add the new entries, update changed ones, drop what the story removed. Each stays a lookup table or catalog: enough to decide reuse, never a copy of what the code or contract already states.

**Checklist:**

- [ ] Each affected index / catalog / reference updated per rules
- [ ] Entries are lookup-level — no content the code or contract already holds
- [ ] Removed assets dropped from their trackers

### 3. Validate

Confirm each artifact is internally consistent and points at assets that exist.

**Checklist:**

- [ ] Every tracked entry resolves to an existing asset
- [ ] No stale entry left from this story's changes

> AR-owned project artifacts brought current → outcome `Maintained`

## Instructions

- Project-level, not story-level — these artifacts span stories and live in the agents worktree; the story's own dev / test documents are not touched here.
- A tracker is a lookup, never a second source of truth — record what is needed to find and reuse an asset, never a copy of what the code or contract already expresses.
- Index-first — the next architect reads these instead of scanning directories, so currency is the whole value; a stale tracker is worse than none.
- Outcome only — the commit (agents worktree) and session log are the persona's; this task ends at the outcome.
- Task outcome: `Maintained`
