---
id: manage-ar-spec.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/ar.ar.rules.md
  - @~/{{ProjectRoot}}/backend/ar.ar.rules.md
---

## Purpose

Make one architecture-infrastructure increment and record it across `ar.spec.md` and the affected rules together — a dependency or library, a tool, a directory rule, or an entry-point structure the spec does not yet describe — so every downstream agent operates against the fulfilled world. Baseline infrastructure is set at init (a separate OR line); this adds increments only. Entered inline when a story workflow cannot proceed without the increment, or standalone to record a new pattern or satisfy a user/OR request.

## Workflow

### 1. Increment Intake

1. Identify the increment: inline, the tool / rule / directory a calling workflow signalled it could not proceed without; standalone, the pattern to record or the request to satisfy.
2. Read the current `ar.spec.md` and the affected `*.ar.rules.md`, plus any prior contract or external reference the increment touches.

**Checklist:**

- [ ] Increment identified (inline signal or standalone request)
- [ ] Current spec and affected rules read

### 2. Make the Change

Perform the infrastructure per rules — introduce the package and make it consumable (wrap it, or open a dependency-rule edge), add the directory, register the entry-point, open the dependency edge. Settle it fully before any dependent artifact is produced, so the increment exists by the time the calling workflow resumes.

**Checklist:**

- [ ] Infrastructure change performed per rules
- [ ] A library made consumable per rules (wrapped, or an explicit dependency-rule edge), never imported raw
- [ ] Change settled before dependent artifacts (no unmet prerequisite left)

### 3. Record and Extract

1. Update `ar.spec.md` so it is the single source — the Directory, Dependency Rules, Tooling, Path Alias, or Ownership table the change touches. Record the binding only; never copy a universal rule into the spec.
2. Update each affected `*.ar.rules.md` so its consumer sees the increment — agents read rules, not the spec.

**Checklist:**

- [ ] `ar.spec.md` updated where the change lands; no universal rule duplicated
- [ ] Every affected `*.ar.rules.md` updated for its consumer
- [ ] Reference direction intact (rules → spec; nothing references harness)

### 4. Validate

Confirm the increment is coherent per rules — types resolve, dependency checks pass, no rule contradicts another.

**Checklist:**

- [ ] Spec-bound checks pass per rules (e.g. typecheck, dependency check)
- [ ] No contradiction introduced across spec and rules

> One architecture-infrastructure increment performed and recorded across spec and rules → outcome `Incremented`

## Instructions

- Infrastructure is precisely a change that alters the spec; work within the spec's granted scope is the calling task's to do directly and never reaches here.
- Spec and rules move together — a binding recorded in the spec but not extracted to the consuming rules is invisible to the agent that needs it; the two are updated in one pass.
- Record the binding, not the principle — the universal rule stays canonical and is referenced, never copied into the spec.
- The increment is settled before the calling workflow resumes — the artifact it then writes assumes the fulfilled world, carrying no unmet-prerequisite note.
- Outcome only — the commit (inline → story worktree; standalone → agents worktree) and session log are the persona's; this task ends at the outcome.
- Task outcome: `Incremented`
