---
id: storybook.idx.tmp
level: 3
owner: or
type: frontend-web-react
---

<!--
  Frontmatter:
  ---
  id: storybook.idx
  level: 5
  owner: ux
  type: frontend-web-react
  desc: {{one-line summary of this project's UI asset catalog}}
  ---
-->

# Storybook Catalog {{stack}}

Index of reusable UI assets registered in the project's Storybook.
Agents reference this document to identify existing components before creating new ones.

Path alias: {{list path aliases, e.g. `@ui/*` → `frontend/ui/*`}}

## Foundations (`ui/storybook/Foundations/`)

Visual token catalogs and design rules — not code components.

| Document | Content |
| -------- | ------- |
| {{doc}}  | {{content summary}} |

## Atoms (`@ui/components/`)

Single-purpose presentational components.

| Component | Import | Description | Key Props |
| --------- | ------ | ----------- | --------- |
| {{name}}  | {{import path}} | {{one-line description}} | {{key props}} |

## Organisms (`@ui/components/`)

Composite components combining multiple atoms.

| Component | Import | Description | Data Shape |
| --------- | ------ | ----------- | ---------- |
| {{name}}  | {{import path}} | {{one-line description}} | {{data shape}} |

## Layouts (`@ui/layouts/`)

Page-level structural shells.

| Layout | Import | Description | Sub-exports |
| ------ | ------ | ----------- | ----------- |
| {{name}} | {{import path}} | {{one-line description}} | {{sub-exports or —}} |

## Shared Utilities

Assets from `shared/` that UX references during component implementation.

| Asset | Import | Purpose |
| ----- | ------ | ------- |
| {{name}} | {{import path}} | {{purpose}} |

## Theme & Styling

{{Describe theme provider import path and runtime token injection mechanism.}}
{{Describe Storybook decorator wrapping (e.g. ThemeProvider + LangProvider on stories and MDX Docs) and how the manager toolbar reuses shared preference controls via context wiring.}}

| Control | Behavior |
| ------- | -------- |
| {{control name}} | {{toggle behavior and visual state}} |

| Category | Classes | Tokens |
| -------- | ------- | ------ |
| {{category}} | {{utility class names}} | {{CSS variable tokens}} |