---
id: ux-spec.tmp
level: 3
owner: or
type: agent
---

<!--
  Frontmatter:
  ---
  id: UX-SPEC-XXX
  level: 6
  owner: ux
  parent: {{Story}}
  desc: {{one-line summary of what this spec covers}}
  ---

  Subordinate to its Story. No independent Owner.
  The prototype IS the design — this spec describes what data drives it.
-->

# [UX-SPEC-XXX]

**Story:** [STORY-XXX](../stories/STORY-XXX/STORY-XXX_*.md)

## Prototype Reference

<!-- Refer to the rules file for path conventions. -->

| Type           | Path      |
| -------------- | --------- |
| Prototype Page | {{paths}} |
| Mock Data      | {{paths}} |
| Components     | {{paths}} |
| Layouts        | {{paths}} |

## Data Requirements

### {{PageName}}

#### {{ComponentName}}

**Props:**

| Prop     | Type (conceptual) | Description              | Source                         |
| -------- | ----------------- | ------------------------ | ------------------------------ |
| {{prop}} | {{type}}          | {{what this data means}} | {{API / user input / derived}} |

**Data States:**

| State     | Data Condition     | AC Reference |
| --------- | ------------------ | ------------ |
| Default   | {{data condition}} | {{AC #}}     |
| Loading   | {{data condition}} | {{AC #}}     |
| Empty     | {{data condition}} | {{AC #}}     |
| Error     | {{data condition}} | {{AC #}}     |
| {{state}} | {{data condition}} | {{AC #}}     |

<!-- Repeat per component within the page -->

## Component Interaction

| Source Component | Event           | Target Component | Effect           |
| ---------------- | --------------- | ---------------- | ---------------- |
| {{component}}    | {{user action}} | {{component}}    | {{what changes}} |

## Validation Rules

| Field     | Rule     | Trigger Condition         | AC Reference |
| --------- | -------- | ------------------------- | ------------ |
| {{field}} | {{rule}} | {{when validation fires}} | {{AC #}}     |

## Navigation

| Action     | Source          | Destination   | Trigger Condition |
| ---------- | --------------- | ------------- | ----------------- |
| {{action}} | {{from screen}} | {{to screen}} | {{condition}}     |

## Component Mapping

| UI Element  | Domain Term (Glossary) |
| ----------- | ---------------------- |
| {{element}} | {{glossary term}}      |
