---
id: de.ar.rules
level: 6
type: frontend-web-react
owner: ar
desc: Frontend implementation rules for DE agent
---

# DE Frontend Rules (Architecture)

Owner: AR. Consumed by DE per AGENTS.md rules-matching protocol.

## Global Constraints

### File Access

| Permission | Path | Description |
| ---------- | ---- | ----------- |
| rw | `frontend/features/**/*.ts` (non-test) | Domain logic — hooks, api, logic |
| rw | `frontend/app/pages/**` | Production pages |
| rw | `frontend/shared/constants/**` | Cross-package constants |
| rw | `frontend/shared/utils/**/*.ts` (non-test) | Cross-package utilities |

All other frontend paths are read-only — including every `*.test.ts(x)` file (the paired test is DE's gate, never DE's to edit) and the AR-owned entry-point and type files (`app/routes.tsx`, `app/providers.tsx`, `app/layout.tsx`, `app/main.tsx`, `shared/types/`). Read paths governed by `ar.spec.md` *Ownership Boundaries*.

### Dependency Rules

DE-written code obeys the architecture dependency graph in `ar.spec.md` *Dependency Rules* / *Forbidden Dependencies*; `depcheck` enforces it. The edges that bind DE's zones:

```text
features → shared
app      → ui/components, ui/layouts, ui/styles, features, shared
shared   → (neither app nor features)
```

A body that needs an import the graph forbids is a contract gap — surface it as the task outcome, never reach across the boundary.

### File Header Convention

Production page files DE creates or modifies carry:

```tsx
// @story STORY-XXX
// @owner de
// @page MappingRuleList
// @components FilterPanel, DataGrid
```

`@page —` for non-page logic files. Empty values use `—` (em-dash). `@story` is required.

### Path Aliases

| Alias         | Resolves to           |
| ------------- | --------------------- |
| `@app/*`      | `frontend/app/*`      |
| `@ui/*`       | `frontend/ui/*`       |
| `@features/*` | `frontend/features/*` |
| `@shared/*`   | `frontend/shared/*`   |

Cross-package imports use the alias; same-package siblings use relative paths.

## implement-dev.task

### Implementation

Zone per the dev document's Position:

| Zone | Path |
| ---- | ---- |
| Domain logic | `frontend/features/{domain}/` (hooks, api, logic) |
| Production page | `frontend/app/pages/` |
| Cross-package utility / constant | `frontend/shared/utils/`, `frontend/shared/constants/` |

- Import cross-package types from `frontend/shared/types/` (AR-owned, read-only) — never redeclare them.
- Generate from or import the cross-dev contract artifacts the dev document names; never duplicate a contract DE was handed.
- A production page transitions its prototype by replacing mock imports with hook imports only (per `ar.spec.md` *Prototype Convention*); `mirrorcheck` enforces the mirror.

**Checklist:**

- [ ] Body written only within the unit's ownership zone
- [ ] Imports obey the dependency graph (`depcheck` clean)
- [ ] Cross-package types imported from `shared/types/`, not redeclared
- [ ] Page files carry the `@owner de` header; prototype mirror preserved
