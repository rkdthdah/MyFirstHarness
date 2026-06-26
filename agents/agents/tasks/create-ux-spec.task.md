---
id: create-ux-spec.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/ux-spec.tmp.md
rules: @~/{{ProjectRoot}}/frontend/ux.ar.rules.md
---

## Purpose

Analyze a story, build a working prototype through co-design with the user, and produce a UX spec

## Workflow

### 1. Story Analysis

1. Read the Story document
   - Note Domain Terms (use Glossary spelling in component naming) and External Dependencies (referenced data sources may shape UI states)
2. Review AC — each AC must map to at least one UI state
3. Scan existing assets for reusable components/layouts
4. Identify: affected screens, required UI states, navigation flows, Glossary terms, data requirements per component

**Checklist:**

- [ ] All AC mappable to UI states
- [ ] Affected screens identified
- [ ] Navigation points identified
- [ ] Glossary terms identified (using Story Domain Terms as starting set)
- [ ] Data requirements per component identified
- [ ] No open questions about story requirements

> Open questions → End task with outcome `Open Questions`

### 2. Prototype Build & Co-Design

> **CRITICAL:** Build iteratively WITH the user. Never build in isolation.

**Build → Show → Feedback → Iterate:**

1. Build prototype
2. Run prototype and review with user
3. Walk through each AC
4. Collect feedback → update → return to 1
5. User confirms → proceed to Step 3

**Checklist:**

- [ ] All AC scenarios visible
- [ ] Loading, error, empty states demonstrated
- [ ] Validation and messages demonstrated
- [ ] Navigation complete
- [ ] Glossary-consistent naming
- [ ] New patterns asked for catalog registration
- [ ] User confirmed
- [ ] UI patterns with a well-known library solution (skeleton loader, date picker, etc.) flagged for AR — hand-built in the prototype, introduction decided by AR

### 3. UX Spec & Validation

Write UX spec per template. Run validation checks.

**Checklist:**

- [ ] Every AC has testable UI state
- [ ] Data States cover all applicable states
- [ ] Component Interaction covers cross-component flows
- [ ] Validation Rules cover all fields
- [ ] Navigation covers all transitions
- [ ] Glossary terms only
- [ ] User confirmed UX spec

> On fail requiring design change → return to Step 2.

### 4. Post-Creation

1. Save UX spec to `~/{{ProjectRoot}}/agents/docs/stories/STORY-XXX/UX-SPEC-XXX.md`
2. Update Story `UX Spec` field: `[UX-SPEC-XXX](./UX-SPEC-XXX.md)`
3. Update catalogs
4. End task with outcome `Completed`

**Checklist:**

- [ ] UX spec saved
- [ ] Story `UX Spec` field linked
- [ ] Catalogs updated

## Instructions

- UX defines WHAT USER SEES — never HOW it's built
- Prototype IS the design — UX spec documents data requirements
- One UX spec per story, always co-design with user
- UX spec ID = Story ID (STORY-005 → UX-SPEC-005)
- Unclear requirements → return to PM
- Glossary terms for UI naming; flag new terms
- Hand-build UI patterns in the prototype; where a well-known library could supply one, flag it for AR — choosing build-vs-introduce is AR's, not UX's
- Task outcomes: `Open Questions` | `Completed`