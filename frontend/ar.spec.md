---
id: ar.spec
level: 5
owner: ar
type: frontend-web-react
desc: Frontend architecture boundaries, convention, ownership rules, and prototype-to-production transition flow
---

# Frontend Architecture Specification

This document defines frontend architecture boundaries, ownership rules, and transition flow from UX prototype → production implementation.

## Directory Structure

```text
frontend/
├── .storybook/                     ← Storybook config (main, manager, preview)
├── ui/
│   ├── components/
│   ├── layouts/
│   ├── styles/
│   │   ├── theme.ts
│   │   ├── ThemeProvider.tsx
│   │   └── global.css
│   ├── storybook/                  ← .stories.tsx files only
│   └── prototype/
│       ├── pages/
│       ├── routes.tsx
│       ├── layout.tsx              ← mirrors app/layout.tsx (Topbar/sidebar shell)
│       ├── mocks/
│       └── main.tsx
├── features/
│   └── {domain}/
│       ├── hooks/
│       ├── api/
│       └── logic/
├── app/
│   ├── pages/
│   ├── routes.tsx
│   ├── providers.tsx
│   ├── layout.tsx
│   └── main.tsx
└── shared/
    ├── types/
    ├── constants/
    ├── utils/
    ├── i18n/                       ← LangProvider, useT, dictionaries
    │   ├── dict.ts                 ← auto-collects ./dict/*.ts via import.meta.glob
    │   └── dict/                   ← domain-scoped slices, just drop a new file
    ├── icons.tsx                   ← shared SVG icon set (Icon, Icons)
    ├── iconPaths.ts                ← raw SVG path data shared with .storybook/manager.ts
    └── vite-env.d.ts               ← Vite client ambient types
```

`*.spec.md`, `*.rules.md`, `*.idx.md` live directly under `frontend/` — see *Document Ownership*.

### Ownership Boundaries

| Glob                          | UX | QA | TE | DE | AR |
| ----------------------------- | -- | -- | -- | -- | -- |
| ui/**/*.tsx                   | rw | r  | r  | r  | r  |
| ui/**/*.test.tsx              | r  | rw | r  | r  | r  |
| features/**/*.ts (non-test)   | r  | r  | r  | rw | rw |
| features/**/*.test.ts         | r  | r  | rw | r  | r  |
| app/pages/**                  | r  | r  | r  | rw | rw |
| app/routes.tsx                | r  | r  | r  | r  | rw |
| app/providers.tsx             | r  | r  | r  | r  | rw |
| app/layout.tsx                | r  | r  | r  | r  | rw |
| app/main.tsx                  | r  | r  | r  | r  | rw |
| shared/types/**               | r  | r  | r  | r  | rw |
| shared/constants/**           | r  | r  | r  | rw | rw |
| shared/utils/**/*.ts (non-test) | r | r | r  | rw | rw |
| shared/utils/**/*.test.ts     | r  | r  | rw | r  | r  |
| shared/i18n/**                | rw | r  | r  | r  | r  |
| shared/icons.tsx              | rw | r  | r  | r  | r  |
| shared/iconPaths.ts           | rw | r  | r  | r  | r  |
| shared/vite-env.d.ts          | r  | r  | r  | r  | rw |
| .storybook/**                 | rw | r  | r  | r  | r  |
| test-utils/**                 | r  | rw | r  | r  | r  |

### Document Ownership

| File                | Owner | Readers              |
| ------------------- | ----- | -------------------- |
| ar.spec.md          | AR    | all                  |
| ar.ar.rules.md      | AR    | AR                   |
| ux.ar.rules.md      | AR    | UX                   |
| de.ar.rules.md      | AR    | DE                   |
| qa.spec.md          | QA    | all                  |
| qa.qa.rules.md      | QA    | QA                   |
| te.qa.rules.md      | QA    | TE                   |
| storybook.idx.md    | UX    | UX                   |
| prototype.idx.md    | UX    | UX                   |

### Dependency Rules

```text
ui/components → shared
ui/layouts    → ui/components, shared
ui/storybook  → ui/components, ui/layouts, ui/styles, shared
ui/prototype  → ui/components, ui/layouts, ui/styles, shared
features      → shared
app           → ui/components, ui/layouts, ui/styles, features, shared
test-utils    → ui, features, shared
```

### Forbidden Dependencies

```text
ui                        → features, app
features                  → ui/prototype, app
shared                    → app, features
app, ui, features, shared → test-utils, **/*.test.*
test-utils                → app
```

## Library Introduction

Stack bindings for introducing a runtime/architecture library (introduction is AR-authored — AR owns this spec):

- An introduction is an edit to this spec (a new entry under *Dependency Rules*, a new package, a new directory).
- A library consumed by `ui/` must be made `ui/`-consumable — wrapped in a `ui/components` component, or granted an explicit *Dependency Rules* entry. `ui/` never imports an unwrapped third-party module directly.
- A new UI capability is a prototype defect only when the prototype omitted it; when a library can supply it, it is a make-or-buy candidate.
- AR may add a Story **External Dependencies** row directly when it leaves every AC's interpretation unchanged and needs no new context doc; otherwise it routes to PM.

## Prototype Convention

Prototype pages mirror production page structure. Only the data source changes.

Allowed: ui/components, ui/layouts, ui/styles, shared, prototype/mocks
Forbidden: features, app

**Rules:**
- Prototype page component structure = Production page component structure
- Transition to production by replacing mock imports with hook imports only
- No business logic in prototype pages

Enforced by `npm run mirrorcheck`.

### Prototype Production Transition Example

Prototype:

```tsx
// ui/prototype/pages/MappingRuleList.tsx
import { DataGrid } from '../../components/DataGrid'
import { mockColumns, mockRows } from '../mocks/mapping-rule'

export function MappingRuleListPage() {
  const loading = false
  return 
}
```

Production:

```tsx
// app/pages/MappingRuleList.tsx
import { DataGrid } from '../../ui/components/DataGrid'
import { useMappingRuleList } from '../../features/mapping-rule/hooks/useMappingRuleList'

export function MappingRuleListPage() {
  const { columns, rows, loading } = useMappingRuleList()
  return 
}
```

## Storybook Convention

UI Pattern Library. Runs independently of prototype/production.

Validates: reusable components, layouts, theme variations.
Does NOT validate: full user flows, API integrations, business workflows.

Allowed: ui/components, ui/layouts, ui/styles, shared
Forbidden: ui/prototype, features, app

## i18n Dictionary Slices

`shared/i18n/dict/*.ts` is auto-collected by `dict.ts` via `import.meta.glob`.

- Add a slice: create `dict/<domain>.ts` exporting `{ ko, en }`. No registration.
- Use a stable prefix per slice (`tb.*`, `frm.*`). Keys must be globally unique — collisions silently last-wins.
- ko and en key sets must match (enforced by convention, not types).

## Storybook Manager Boundary

`.storybook/manager.ts` is built by esbuild without Vite alias support.

- Relative imports only (no `@shared/*`, `@ui/*`).
- Do not import user components — transitive `@`-aliases will fail to resolve.
- For UI duplicated between manager and a user component, share data via plain-string modules (e.g. `shared/iconPaths.ts`) and mark both sites with `KEEP IN SYNC`.

## Path Aliases

TypeScript and Vite share the same alias map (also read by `dependency-cruiser`).

| Alias            | Resolves to               |
| ---------------- | ------------------------- |
| `@app/*`         | `frontend/app/*`          |
| `@ui/*`          | `frontend/ui/*`           |
| `@features/*`    | `frontend/features/*`     |
| `@shared/*`      | `frontend/shared/*`       |
| `@test-utils/*`  | `frontend/test-utils/*`   |

- Cross-package imports use the alias.
- Same-package siblings use relative paths.

## File Header Convention

All page files must include a comment header:

```tsx
// @story STORY-003
// @owner ux
// @page MappingRuleList
// @components FilterPanel, DataGrid, StatusBadge
```

Empty values use `—` (not `none` / `N/A` / blank). `@story` is required — use `—` if the story is not yet registered in `prototype.idx.md`.

## Runtime Commands

| Command           | Purpose                | Entry                 |
| ----------------- | ---------------------- | --------------------- |
| npm run storybook | isolated UI validation | ui/storybook          |
| npm run proto     | prototype validation   | ui/prototype/main.tsx |
| npm run dev       | production app         | app/main.tsx          |
| npm run typecheck | type validation        | tsc --noEmit          |
| npm run depcheck  | enforce *Forbidden Dependencies* (above) | dependency-cruiser |
| npm run depgraph  | print Graphviz dot of the module graph   | dependency-cruiser |
| npm run mirrorcheck | enforce *Prototype Convention* mirror invariant | scripts/mirror-check.ts |

`depcheck` and `mirrorcheck` exit non-zero on violation — wire into pre-commit / CI.

Test-related commands: see `qa.spec.md` *Runtime Commands*.