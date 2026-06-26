---
id: ar.spec.tmp
level: 3
owner: or
type: frontend-web-react
---

<!--
  Frontmatter: 
  ---
  id: ar.spec
  level: 5
  owner: ar
  type: frontend-web-react
  desc: {{one-line purpose of this MANIFEST}}
  ---
-->

# Frontend Specification

{{One-line purpose of this document}}

## Directory Structure

```text
{{project directory tree with ownership annotations}}
```

### Ownership Boundaries

| Glob      | {{Agent 1}} | {{Agent 2}} | {{Agent 3}} |
| --------- | ----------- | ----------- | ----------- |
| {{glob}}   | {{r/rw}}    | {{r/rw}}    | {{r/rw}}    |

### Document Ownership

| File          | Owner       | Consumed by |
| ------------- | ----------- | ----------- |
| {{file.md}}   | {{agent}}   | {{agent}}   |

### Dependency Rules

```text
{{layer → allowed dependencies}}
```

### Forbidden Dependencies

```text
{{layer → forbidden dependencies}}
```

## Prototype Convention

{{Prototype purpose and scope}}

Allowed: {{allowed imports}}
Forbidden: {{forbidden imports}}

**Rules:**
- {{rule}}

## Storybook Convention

{{Storybook purpose and scope}}

Allowed: {{allowed imports}}
Forbidden: {{forbidden imports}}

**Rules:**
- {{rule}}

## Path Aliases

| Alias | Resolves to |
| ----- | ----------- |
| {{alias}} | {{path}} |

## File Header Convention

```tsx
// @story {{STORY-XXX}}
// @owner {{agent}}
// @page {{PageName}}
// @components {{Component1, Component2}}
```

## Runtime Commands

| Command | Purpose | Entry |
| ------- | ------- | ----- |
| {{command}} | {{purpose}} | {{entry point}} |