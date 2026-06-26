---
id: ux.ar.rules
level: 6
type: frontend-web-react
owner: ar
desc: Frontend rules and conventions for UX agent
---

# UX Frontend Rules

Owner: AR. Consumed by UX per AGENTS.md rules-matching protocol.

## Global Constraints

### File Access

| Permission | Path | Description |
| ---------- | ---- | ----------- |
| rw | `frontend/ui/` | Components, layouts, styles, storybook, prototype |
| rw | `frontend/storybook.idx.md` | UI asset catalog |
| rw | `frontend/prototype.idx.md` | Story-to-file index |
| rw | `frontend/shared/i18n/` | LangProvider, useT, ko/en dictionaries |
| rw | `frontend/shared/icons.tsx` | Shared SVG icon set |
| rw | `frontend/shared/iconPaths.ts` | Raw SVG path data shared with .storybook/manager.ts |
| rw | `frontend/.storybook/` | Storybook config (main, manager, preview) |

All other frontend paths are read-only

### Dependency Rules

Allowed:

```text
ui/components → shared
ui/layouts    → ui/components, shared
ui/storybook  → ui/components, ui/layouts, ui/styles, shared
ui/prototype  → ui/components, ui/layouts, ui/styles, shared
```

Forbidden:

```text
ui → features, app
```

### Component Rules

- Components must be pure UI — props only, no data fetching.
- Reuse existing components from `storybook.idx.md` before creating new ones.
- New components require user approval before Storybook registration.
- Do not add external libraries. Where a UI pattern has a well-known library solution, hand-build it in the prototype and flag it for AR. If AR introduces one, consume it only through the `ui/`-consumable form AR provides (a wrapping component or an opened dependency rule) — never import the library directly into `ui/prototype`.

### File Header

All page files must include:

```tsx
// @story STORY-XXX
// @owner ux
// @page PageName
// @components Component1, Component2
```

Empty values use `—` (em-dash, not `none` / `N/A` / blank).

### Import Convention

- Cross-package: use alias (`@ui/*`, `@shared/*`)
- Same-package siblings: use relative path

### i18n Dictionary Slices

`shared/i18n/dict/*.ts` is auto-collected by `dict.ts` via `import.meta.glob`.

- Add a slice: create `dict/<domain>.ts` exporting `{ ko, en }`. No registration.
- Use a stable prefix per slice (`tb.*`, `frm.*`). Keys must be globally unique — collisions silently last-wins.
- ko and en key sets must match (enforced by convention, not types).

### Storybook Manager Boundary

`.storybook/manager.ts` is built by esbuild without Vite alias support.

- Relative imports only (no `@shared/*`, `@ui/*`).
- Do not import user components — transitive `@`-aliases will fail to resolve.
- For UI duplicated between manager and a user component, share data via plain-string modules (e.g. `shared/iconPaths.ts`) and mark both sites with `KEEP IN SYNC`.

## create-ux-spec

### Story Analysis

Scan these indexes to identify reusable assets:

1. `frontend/prototype.idx.md` — components/layouts used by existing stories
2. `frontend/storybook.idx.md` — established patterns for identified components

**Checklist:**

- [ ] prototype.idx.md reviewed for reusable components/layouts
- [ ] storybook.idx.md checked for established patterns
- [ ] Reusable components listed with intended usage
- [ ] New components identified only where no existing fit

### Prototype Build & Co-Design

#### Locations

| Asset | Path |
| ----- | ---- |
| Components | `frontend/ui/components/` |
| Layouts | `frontend/ui/layouts/` |
| Storybook stories | `frontend/ui/storybook/` |
| Prototype pages | `frontend/ui/prototype/pages/` |
| Prototype routing | `frontend/ui/prototype/routes.tsx` |
| Mock data | `frontend/ui/prototype/mocks/` |
| i18n dictionaries | `frontend/shared/i18n/dict/<domain>.ts` (auto-collected) |

#### Run Prototype

```bash
npm run proto
```

Run, open browser, review with user.

### UX Spec & Validation

```bash
npm run proto        # Prototype runs without errors
npm run typecheck    # TypeScript type correctness
npm run depcheck     # No forbidden dependency violations
```

**Checklist:**

- [ ] Prototype runs without errors
- [ ] Type check passed
- [ ] Dependency check passed
- [ ] File headers present on all new page files
- [ ] Import conventions followed
- [ ] No forbidden imports

### Post-Creation

Update catalogs after creating or modifying assets:

#### `prototype.idx.md`

Add/update one row per story:

| Column | Content |
| ------ | ------- |
| Story | `STORY-XXX` |
| Components | All component files created/modified |
| Layouts | All layout files created/modified |
| Prototype Page | All prototype page files created/modified |
| Mock Data | All mock data files created/modified |
| Storybook | All `.stories.tsx` files created/modified |

#### `storybook.idx.md`

If new components or layouts were registered in Storybook, add a row to the appropriate table (Atoms, Organisms, or Layouts).

**Checklist:**

- [ ] `prototype.idx.md` updated (all new/modified files)
- [ ] `storybook.idx.md` updated (if new components/layouts registered)

## revise-ux-spec

### Change Triage

Scan these indexes to locate currently registered assets for this story:

1. `frontend/prototype.idx.md` — current row for this story (existing components/layouts/pages)
2. `frontend/storybook.idx.md` — current components/layouts owned by this story

**Checklist:**

- [ ] Current prototype.idx.md row for this story read
- [ ] storybook.idx.md entries owned by this story identified

### Revision

When prototype edit is required, use the same locations and run command as create-ux-spec.

#### Run Prototype

```bash
npm run proto
```

### Validation

```bash
npm run proto         # Prototype runs without errors
npm run typecheck     # TypeScript type correctness
npm run depcheck      # No forbidden dependency violations
```

**Checklist:**

- [ ] Prototype runs without errors
- [ ] Type check passed
- [ ] Dependency check passed
- [ ] File headers updated where component lists changed
- [ ] Import conventions followed on edited files

### Post-Update

Update catalogs to reflect changes:

#### `prototype.idx.md`

Update the existing row for this story:
- Add newly created files
- Remove files that are no longer used by the story
- Update component lists for modified pages

#### `storybook.idx.md`

- Add rows for any new components/layouts registered
- Remove rows for components/layouts no longer registered

**Checklist:**

- [ ] `prototype.idx.md` row updated (additions and removals)
- [ ] `storybook.idx.md` updated if component/layout set changed