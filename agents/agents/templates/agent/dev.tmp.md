---
id: dev.tmp
level: 3
owner: or
type: agent
---

<!--
  Frontmatter:
  ---
  id: dev-XXX-NN
  level: 7
  owner: {{AR | DE | QA | Complete}}
  parent: STORY-XXX
  covers-acs: [{{AC ID list}}]
  test-kind: {{business-logic | placeholder | ar-integration}}
  desc: {{one-line summary of what this unit delivers}}
  ---
-->

# [dev-XXX-NN]

{{One-line description of what this unit delivers}}

## Definition of Done

<!-- Progress is the set of checked items. Owner in the filename = who acts next.
     placeholder / ar-integration: Implemented and Reviewed are [N/A] at creation;
     Verified is still taken.
     Verified marks story-integration passed — set for every pair together when the
     story's Verification passes, not a per-unit check. -->

- [ ] Designed — AR
- [ ] Implemented — DE
- [ ] Reviewed — AR
- [ ] Verified — QA

## Position

| Field             | Value                                      |
| ----------------- | ------------------------------------------ |
| Zone              | {{ownership zone per ar rules}}            |
| Imports allowed   | {{allowed dependencies per ar spec}}       |

## Baseline

<!-- Redesign only: the commit holding this unit's prior implementation. DE diffs the
     current design against it to revert what the design drops; AR review checks the
     revert is complete. `—` when there is no prior implementation (initial design). -->

- {{commit hash / —}}

## Design

### Files

<!-- Boundary files only — what other units import, or system entry points.
     Internal helpers are DE's. A file the story removes from the boundary gets Action `remove`. -->

| Path                        | Role                                      | Action          |
| --------------------------- | ----------------------------------------- | --------------- |
| {{relative file path}}      | {{what this file exposes}}                | {{create / modify / remove}} |

### Public Signatures

<!-- Exported functions, classes, modules, or components.
     Code block per export in the target language. Body is forbidden — only signatures and types. -->

```
{{export signature}}
```

### Schemas

<!-- Cross-unit or external-facing data shapes. Reference cross-dev contract artifact
     files (created by AR per ar rules) instead of duplicating here. -->

| Schema             | Defined in                                | Used by                  |
| ------------------ | ----------------------------------------- | ------------------------ |
| {{schema name}}    | {{contract artifact file path}}           | {{unit / external party}} |

## Behavioral Contract

<!-- What another unit or AC relies on. Implementation-internal mechanisms stay out
     even when externally observable in principle. Body is forbidden. -->

### Pre/post conditions

- {{assumption that must hold on entry / state guaranteed on return}}

### Error and edge case behavior

- When {{trigger}} → {{observable response}}

### Observable state transitions

- After {{event}} → {{state visible through the contract}}

### Invariants

- {{property that always holds}}

<!-- Empty subsection: replace bullet content with `—` (em-dash). -->

## Notes

<!-- Cross-unit sequencing, foreseen edge cases not in Behavioral Contract,
     dependencies on other units in this Story. -->

- {{free-form note, list, or prose}}

## Handoff Notes

<!-- Each owner appends a note when passing the document to the next agent.
     Format: [DateTime] FROM → TO: note -->