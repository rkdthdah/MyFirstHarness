---
id: story.tmp
level: 3
owner: or
type: agent
---

<!-- 
  Frontmatter:
  ---
  id: STORY-XXX
  level: 7
  owner: {{Owner}}
  desc: {{one-line summary of what this story delivers}}
  ---

  `Owner` at file name: PM | UX | AR | QA | Complete | Discarded
  Owner Flow:
    Phase 1: PM → [UX → PM →] QA → AR
    Phase 2: AR (architecture → sleeping while dev/test progress → integration) → PM

  PM/UX/AR/QA labels = who fills {{}} or checks [ ].
-->

# [STORY-XXX_{{Owner}}]

{{Concise description of the enhancement}}

## User Story — `PM`

As a {{type of user}},
I want {{specific action/capability}},
So that {{clear benefit/value to the user}}.

## Story Context — `PM`

<!-- CURRENT state — before this story is applied -->

**Classification:** {{Iteration | Feature}}
**Parent Story:** {{[STORY-XXX] or "N/A" if Feature}}
**UX Spec:** N/A

- Current behavior: {{how the system handles this today, or "not supported"}}
- Affected area: {{which part of the system the user interacts with}}
- Related user workflows: {{existing workflows that may be affected}}

## Domain Terms — `PM`

<!-- Domain nouns/verbs the AC depends on. Each must be in the Glossary with matching spelling.
     Excludes UI interactions, generic system actions, Story meta-language. None → row with Term = "—". -->

| Term | Kind | Status |
| ---- | ---- | ------ |
| {{term}} | {{entity / value-object / state / role / command / domain-event}} | {{existing / new}} |

## External Dependencies — `PM`

<!-- External systems/data sources this Story depends on. PM provides facts AR cannot infer;
     not implementation detail. Context Doc = `docs/context/{file}.md` when user provided detail.
     None → row with Source = "none". -->

| Source | Type | Access / Constraints | Context Doc |
| ------ | ---- | -------------------- | ----------- |
| {{system or data source}} | {{external DB / API / file / queue / system}} | {{endpoint, auth kind, freshness, retention, permission, limits}} | {{docs/context/{file}.md or —}} |


## Acceptance Criteria — `PM`

<!-- Add as many as needed. Every criterion must be pass/fail testable.
     Define WHAT and WHY only — never HOW (no implementation decisions).
     Every domain noun/verb here must appear in Domain Terms with Glossary-matching spelling.
     UX translates edge/validation/alternative cases into UI states;
     QA translates all criteria into test scenarios.
     For project-wide non-functional requirements, see Requirements Specification. -->

**Normal flow:**
1. {{expected behavior under valid conditions}}

**Alternative flow:**
A1. {{valid but non-primary path}} → {{expected behavior}}

**Edge cases:**
B1. {{exception condition}} → {{expected system response}}

**Validation:**
C1. {{invalid input/state}} → {{expected system response}}

**Non-functional:**
D1. {{performance, security, usability, or other quality requirement specific to this story}}

## User-Facing Notes — `PM`

<!-- CHANGE — what is different for the user after this story -->

- **Workflow change:** {{how the user's workflow changes, or "no change"}}
- **Constraints:** {{any operational or environmental constraints}}

## Testability Results — `QA`

| AC ID | Testable | Notes |
| ----- | -------- | ----- |
| {{1}} | {{[x]/[ ]}} | {{revision reason if not testable}} |

## UI Test Coverage — `QA`

> When UI not required, PM delete this section

| AC ID | Test File | Coverage |
| ----- | --------- | -------- |
| {{1}} | {{frontend/ui/.../X.test.tsx}} | {{which state/flow}} |

## Definition of Done

> When UI not required, PM marks all UI item as [N/A]

- [ ] Requirements — `PM`
- [ ] Design (UI item) — `UX`
- [ ] Testability — `QA`
- [ ] UI Test (UI item) — `QA`
- [ ] Development — `AR`
- [ ] Delivery — `PM`

## Handoff Notes

<!-- Each agent appends a note when passing the story to the next agent. -->
<!-- Format: [DateTime] FROM → TO: note -->