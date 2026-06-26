---
id: refine-story.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/story.tmp.md
---

## Purpose

Process a Story returned from a downstream agent (UX/QA/AR), classify the defect, apply targeted edits when appropriate, and report what changed so the persona can rebuild downstream state.

## Workflow

### 1. Handoff Note Triage

Read the most recent inbound Handoff Note. Identify:

- **Source agent** — `UX` | `QA` | `AR`
- **Reason label** by source:
  - UX → `open questions`
  - QA → `testability defect` | `UI test defect`
  - AR → `structural glossary change` | `AC defect` | `external dependency currency failure` | `library introduced`
- **Defect location** — which artifact(s) carry the actual flaw: `AC` | `Domain Terms` | `External Deps` | `UX spec` | `Glossary` | `context doc` | `composite`

Defect location is independent of source. A UX spec defect can surface from QA UI test or AR coherence review.

**Checklist:**

- [ ] Handoff Note read; source identified
- [ ] Reason label assigned
- [ ] Initial defect location identified

### 2. Source Reading

Read the Story in full. Add inbound-specific context:

| Source / reason | Additional reading |
| --- | --- |
| QA `testability defect` | Testability Results rows, Glossary entries for cited terms |
| QA `UI test defect` | UI Test Coverage rows, UX spec, prototype reference |
| UX `open questions` | UX spec draft (if any), parent/related stories |
| AR `structural glossary change` | Glossary entries cited by AR, affected AC |
| AR `AC defect` | AC under question, cited Glossary entries, External Deps |
| AR `external dependency currency failure` | External Deps row, existing context doc, AR's currency-check finding |
| AR `library introduced` | AR's Handoff Note (library introduced, prototype surfaces needing rework), UX spec, prototype reference |

Confirm defect location against source materials; the Triage label may be revised here.

**Checklist:**

- [ ] Story read in full
- [ ] Inbound-specific context read
- [ ] Defect location confirmed (Triage label revised if needed)

### 3. Change Scope Assessment

Classify the change required:

| Category | Trigger | Outcome candidate |
| --- | --- | --- |
| Story-level invalidation | Story value no longer justifies cost, AC majority invalidated, scope fundamentally redefined | `Discarded` or `Major Rework` |
| Meaning change | AC meaning/scope/priority change, new AC required, undefined behavior to be defined | `Minor Revision` (interaction required) |
| Meaning-preserving | Wording polish, Glossary spelling/registration correction, Tier 2 Glossary change applied to AC without altering interpretation, or a library introduced by AR requiring prototype rework (AC unchanged) | `Minor Revision` (interaction may be skipped) |

**Checklist:**

- [ ] Change scope classified
- [ ] Outcome candidate identified

### 4. User Interaction (Conditional)

Per Step 3 classification:

- **Story-level invalidation** — Present findings; confirm `Discarded` vs `Major Rework`. Discarded requires explicit reason.
- **Meaning change** — Confirm user intent for new or changed AC. PM does not infer behavior.
- **Meaning-preserving** — Skip.
- **New context doc required** (AR `external dependency currency failure`) — Request updated context doc from the user; receive and place under `~/{{ProjectRoot}}/docs/context/`.

**Checklist:**

- [ ] Required interactions completed
- [ ] User intent confirmed for meaning changes
- [ ] Context doc received and placed (if applicable)

### 5. Apply Changes

Per outcome:

- **Discarded** — Record reason; no artifact edits.
- **Major Rework** — No artifact edits.
- **Minor Revision** — Apply targeted edits to the defect location:
  - AC / Domain Terms / External Deps in the Story
  - Glossary entries (Tier 2 structural change reflection)
  - Context doc placement or update
  - Cross-consistency: Domain Terms ↔ Glossary spelling/kind, AC ↔ UX spec for UI stories, External Deps ↔ context doc

**Checklist:**

- [ ] Edits applied per outcome
- [ ] For Minor Revision: Story-internal consistency verified (Domain Terms, AC, External Deps, DoD)
- [ ] For Minor Revision: Glossary consistency verified

### 6. Outcome and Change Set

Outcome plus, for `Minor Revision`, the change set — categories of artifacts actually edited. The change set tells the persona which downstream DoD claims are no longer valid.

Change set categories:

- `story-meta` — Glossary spelling/registration, External Deps row metadata; AC text unchanged
- `context-doc` — context doc added or replaced; AC text unchanged
- `ac` — AC text or meaning changed (including new AC)
- `ux-spec` — UX spec needs revision (PM does not edit UX spec; flagged for downstream). Add when an `ac` change affects UI-mapped behavior (trigger/signal resolvable to a UX spec row), when the inbound defect itself was a UX spec defect, or when AR introduced a library requiring prototype rework. PM checks `UI Test Coverage` rows of changed AC to determine UI mapping.

Multiple categories may apply.

**Checklist:**

- [ ] Outcome label set
- [ ] For Minor Revision: change set categories listed

> Task outcomes: `Discarded` | `Major Rework` | `Minor Revision`. Minor Revision is paired with the change set.

## Instructions

- PM defines WHAT and WHY. AC edits do not encode HOW.
- Interaction gate is scope-driven: meaning-preserving edits skip interaction; meaning changes require user confirmation.
- Tier 1 Glossary refinements are AR-direct and never reach PM. Only Tier 2 structural changes arrive from AR.
- External dependency content negotiation is not PM's job — placement of the context doc is.
- PM does not edit UX spec. UX spec changes flag the `ux-spec` change set category for the persona to act on.
- Task outcomes: `Discarded` | `Major Rework` | `Minor Revision`