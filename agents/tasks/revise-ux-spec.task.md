---
id: revise-ux-spec.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/ux-spec.tmp.md
rules: @~/{{ProjectRoot}}/frontend/ux.ar.rules.md
---

## Purpose

Revise an existing UX spec and its prototype against PM's revision intent. Apply targeted edits — never rebuild what is unchanged.

## Workflow

### 1. Change Triage

1. Read the most recent inbound Handoff Note from PM — identify revision intent and called-out items
2. Read the Story document in full — current AC, Domain Terms, External Dependencies, User-Facing Notes
3. Read the existing UX spec
4. Identify changed Story items by comparing against UX spec assumptions:
   - AC added / removed / reworded
   - Domain Term added / renamed / re-kinded
   - User-Facing Notes shifted in scope or constraint
   - A library introduced by AR (Handoff Note) that should now back a prototype surface previously hand-built

**Checklist:**

- [ ] Handoff Note read; revision intent understood
- [ ] Story read; changed items identified
- [ ] Existing UX spec read
- [ ] No open questions about revision intent

> Open questions → End task with outcome `Open Questions`

### 2. Impact Analysis

For each changed Story item, locate affected UX spec sections and prototype surfaces:

- AC removed → UX spec rows mapped to that AC become orphaned (remove or re-scope)
- AC reworded affecting UI behavior → corresponding UX spec rows and prototype interactions need update
- AC reworded without UI behavior change → no UX spec edit needed (audit only)
- New AC → new UX spec rows + prototype interactions required
- Domain Term change → naming updates across UX spec and prototype
- Library introduced → replace the hand-built prototype surface with the introduced library (AC unchanged); UX spec rows updated only where the surface's described behavior shifts

Decide prototype impact:

- UI behavior changes, or a library now backs a surface → prototype edit required (Step 3 prototype path)
- Text-only UX spec cleanup (orphaned rows, naming) → prototype edit not required (Step 3 text-only path)

**Checklist:**

- [ ] Each changed Story item mapped to UX spec / prototype surfaces
- [ ] Prototype impact determined

### 3. Revision

**If prototype edit required:**

Build → Show → Feedback → Iterate, scoped to affected surfaces:

1. Edit prototype on affected surfaces only
2. Run and review with user, walking through changed AC
3. Collect feedback → update → return to 1
4. User confirms → proceed

**If prototype edit not required:**

Apply UX spec text edits directly. Confirm with user before saving.

If a new ambiguity in PM revision intent surfaces during revision, bring prototype and UX spec to a typecheck/depcheck-passing state (complete the partial edit safely or revert it), then end task with outcome `Open Questions`. Partial work in a stable state is preserved for the next pass.

**Checklist:**

- [ ] Changes applied to affected surfaces only
- [ ] Unchanged surfaces preserved
- [ ] Glossary-consistent naming maintained
- [ ] User confirmed

### 4. Validation

Run validation against the changed scope plus its immediate neighbors:

**Checklist:**

- [ ] Every changed/new AC has testable UI state
- [ ] Orphaned rows removed
- [ ] Data States still cover all applicable states for affected components
- [ ] Validation Rules consistent with current AC
- [ ] Navigation transitions still complete
- [ ] Glossary terms only
- [ ] User confirmed revised UX spec

> On fail requiring design change → return to Step 3.

### 5. Post-Update

1. Save updated UX spec
2. Update catalogs if patterns changed
3. End task with outcome `Completed`

**Checklist:**

- [ ] UX spec saved
- [ ] Catalogs updated if applicable

## Instructions

- Apply targeted edits — never rebuild unchanged surfaces.
- UX defines WHAT USER SEES — never HOW it's built.
- Prototype IS the design when UI behavior changes; co-design with user remains mandatory in that path.
- Glossary terms for UI naming; flag new terms.
- Unclear PM revision intent → return to PM.
- Task outcomes: `Open Questions` | `Completed`