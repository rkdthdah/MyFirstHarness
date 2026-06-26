---
id: create-story.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/story.tmp.md
---

## Purpose

Create a user story through requirements discovery, producing a scoped, unambiguous story ready for AR handoff. PM defines **what** and **why** — never **how**

## Workflow

### 1. Requirements Discovery

> **CRITICAL:** Do NOT skip this step. Engage the user before writing anything

Before discovery: scan `~/{{ProjectRoot}}/docs/` (service planning, user scenarios) to ground discovery in current system state.

**Discovery Questions (adapt to context):**

1. What problem are you trying to solve?
2. Which user type is affected?
3. How does the system handle this today?
4. What does "done" look like from the user's perspective?
5. Any operational or environmental constraints?
6. What can go wrong? Any limits or boundaries? (max size, max count, invalid input, empty results, permission denied, timeout, etc.)
7. Are there alternative paths the user might take? (cancel, skip, back, etc.)
8. Any specific performance, security, or usability expectations for this feature?
9. What domain concepts (entities, states, actions, roles, events) does this story touch? Which exist in Glossary, which are new?
10. Any external systems/data sources? For each: what policy/access/structure detail must PM or AR know? If large, place under `docs/context/` and share the filename.

**Checklist:**

- [ ] The "why" behind the request is clearly understood
- [ ] Target user type identified
- [ ] Current vs. desired behavior gap defined
- [ ] Scope boundaries agreed with user
- [ ] AC categories covered (Normal / Alternative / Edge / Validation / Non-functional)
- [ ] Domain concepts identified
- [ ] External dependencies identified; context docs collected if provided
- [ ] Traces back to concrete user value
- [ ] No open questions
- [ ] All findings based on user dialogue — no assumptions
- [ ] Discovery summary confirmed by user

### 2. Scope Classification & Overlap Check

1. Scan `~/{{ProjectRoot}}/agents/docs/stories/` (including `archived/`) for OTHER stories touching the same affected area or external dependencies
2. For each match:
   - **Complete duplicate** — same requirement → outcome `Discarded`
   - **Iteration** — improves or extends an existing story → record parent
   - **Separate** — continue
3. For active stories sharing an external dependency: confirm with user — wait, or proceed in parallel with explicit approval (record in Handoff Notes)
4. Classify:

| Classification | Criteria |
| -------------- | -------- |
| **Iteration** | Small change within existing behavior |
| **Feature** | New capability or significant behavior change |
| **Epic candidate** | Multiple stories needed |

**Checklist:**

- [ ] Existing stories scanned for area and dependency overlap
- [ ] Parallel-dependency conflicts resolved with user
- [ ] Classification determined
- [ ] Completable as a single unit of work
- [ ] MVP scope — minimum required value only
- [ ] If Epic candidate, split plan approved by user
- [ ] If duplicate, end task with outcome `Discarded`

### 3. Context Assessment

- [ ] Relevant existing functionality identified
- [ ] Current user workflow understood
- [ ] Affected screens or interactions identified
- [ ] Known system constraints identified
- [ ] Dependencies on other stories identified (Feature only)
- [ ] UI change required: Yes / No

### 4. Story Creation

Write the story using the template.

> Follow the template exactly. PM fills all `PM`-marked sections except `Delivery` in `Definition of Done`
> If no UI change → Mark all UX item DoD as `[N/A]` and delete the `UI Test Coverage` section
> Before saving: register every `new` Domain Terms entry in Glossary with Kind. Place user-provided context docs in `~/{{ProjectRoot}}/docs/context/` and reference in External Dependencies.

**Checklist:**

- [ ] All PM-marked sections filled
- [ ] UX DoD marked `[N/A]` if no UI change
- [ ] AC are pass/fail testable
- [ ] All AC categories covered (empty marked N/A with reason)
- [ ] Domain Terms lists every domain noun/verb in AC; `new` entries registered in Glossary
- [ ] AC uses Glossary spelling consistently
- [ ] External Dependencies filled (or single "none" row); context docs placed if provided
- [ ] No technical implementation decisions
- [ ] Requirements are unambiguous and actionable by AR with no further PM clarification needed

### 5. Post-Creation

1. **Save:**
   - Create folder: `~/{{ProjectRoot}}/agents/docs/stories/STORY-XXX/`
   - Save story file: `STORY-XXX/STORY-XXX_PM.md`
2. **Present** to user/agent for final approval
   - Approved → end task
   - Rejected → return to Step 1 with feedback
   - Discarded → end task with outcome `Discarded`, record reason

**Checklist:**

- [ ] Story file saved
- [ ] User approved

## Instructions

- ALWAYS start with Requirements Discovery
- PM defines WHAT and WHY — never HOW
- Glossary is the project's Ubiquitous Language (DDD). PM is the entry point — name during discovery, register before handoff, use Glossary spelling in AC.
- External Dependencies: facts AR cannot infer (source, access, business meaning, freshness, permission). Detail beyond a cell → `docs/context/`. Implementation detail (schema, query, library) is AR's.
- One story at a time, end-to-end
- If complexity grows, split rather than inflate scope
- When Epic candidate, present split plan to user. Proceed only after approval, then execute this task per story.
- Task outcomes: `Discarded` | `Approved`