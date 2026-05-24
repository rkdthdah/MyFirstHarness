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

# Frontend Architecture Specification

{{One-line purpose, e.g.: defines frontend architecture boundaries, ownership rules, and transition flow from UX prototype → production implementation.}}

## Directory Structure

```text
{{project directory tree with inline role/ownership annotations}}
```

{{Note where governance docs (*.spec.md, *.rules.md, *.idx.md) live in the tree.}}

### Ownership Boundaries

| Glob      | {{Agent 1}} | {{Agent 2}} | {{Agent 3}} |
| --------- | ----------- | ----------- | ----------- |
| {{glob}}  | {{r/rw}}    | {{r/rw}}    | {{r/rw}}    |

### Document Ownership

| File         | Owner     | Readers      |
| ------------ | --------- | ------------ |
| {{file.md}}  | {{agent}} | {{agent(s)}} |

{{Note on read scope, e.g.: "Agents read only their own rules file — not spec files, not other agents' rules."}}

### Dependency Rules

```text
{{layer → allowed dependencies}}
```

### Forbidden Dependencies

```text
{{layer → forbidden dependencies}}
```

## Prototype Convention

{{Prototype purpose and its structural relationship to production.}}

Allowed: {{allowed imports}}
Forbidden: {{forbidden imports}}

**Rules:**
- {{rule}}

{{Note on enforcement command, if any.}}

### Prototype Production Transition Example

Prototype:

```tsx
{{prototype page sample using mock data source}}
```

Production:

```tsx
{{production page sample showing the minimal diff — mock imports replaced with hook imports, same component structure}}
```

## Storybook Convention

{{Storybook purpose and scope.}}

Validates: {{what Storybook validates}}
Does NOT validate: {{what Storybook does not validate}}

Allowed: {{allowed imports}}
Forbidden: {{forbidden imports}}

## i18n Dictionary Slices

{{Describe auto-collection mechanism (e.g. `import.meta.glob`), slice naming/prefix conventions, key uniqueness rules, and locale parity requirements.}}

## Storybook Manager Boundary

{{Describe constraints of the Storybook manager build (e.g. no Vite alias support, no user-component imports) and the strategy for sharing data between manager and user components (e.g. plain-string modules marked `KEEP IN SYNC`).}}

## Path Aliases

{{Note on alias map source of truth — e.g., shared between tsconfig, bundler config, and dependency analysis tooling.}}

| Alias     | Resolves to |
| --------- | ----------- |
| {{alias}} | {{path}}    |

- {{rule on cross-package vs same-package imports}}

## File Header Convention

All page files must include a comment header:

```tsx
// @story {{STORY-XXX}}
// @owner {{agent}}
// @page {{PageName}}
// @components {{Component1, Component2}}
```

{{Note on empty-value placeholder (e.g. `—`, not `none`/`N/A`/blank) and which fields are required.}}

## Runtime Commands

| Command     | Purpose      | Entry            |
| ----------- | ------------ | ---------------- |
| {{command}} | {{purpose}}  | {{entry point}}  |

{{Note on which commands exit non-zero on violation and are wired into pre-commit / CI.}}
{{Pointer to test-related commands in qa.spec.md or equivalent.}}