---
id: ar.ar.rules
level: 6
type: frontend-web-react
owner: ar
desc: Frontend architecture rules for AR agent
---

# AR Frontend Rules

Owner: AR. Consumed by AR per AGENTS.md rules-matching protocol.

## Global Constraints

### File Access

| Permission | Path | Description |
| ---------- | ---- | ----------- |
| rw | `frontend/app/routes.tsx`, `frontend/app/providers.tsx`, `frontend/app/layout.tsx`, `frontend/app/main.tsx` | Entry-point files — contract artifacts |
| rw | `frontend/shared/types/` | Cross-package types |
| rw | `frontend/shared/vite-env.d.ts` | Vite client ambient types |
| rw | `frontend/ar.spec.md` | Frontend architecture specification |
| rw | `frontend/*.ar.rules.md` | AR-owned rules files (ar.ar, ux.ar, de.ar) |

All other frontend paths are read-only.

### Dependency Rules

Allowed:

```text
ui/components → shared
ui/layouts    → ui/components, shared
ui/storybook  → ui/components, ui/layouts, ui/styles, shared
ui/prototype  → ui/components, ui/layouts, ui/styles, shared
features      → shared
app           → ui/components, ui/layouts, ui/styles, features, shared
test-utils    → ui, features, shared
```

Forbidden:

```text
ui                        → features, app
features                  → ui/prototype, app
shared                    → app, features
app, ui, features, shared → test-utils, **/*.test.*
test-utils                → app
```

### File Header Convention

Entry-point files AR creates or modifies must include:

```tsx
// @story STORY-XXX
// @owner ar
// @page —
// @components Component1, Component2
```

`@page —` for non-page entry files. `@components` lists imported UI components when applicable. Empty values use `—` (em-dash). `@story` is required.

### Path Aliases

| Alias            | Resolves to               |
| ---------------- | ------------------------- |
| `@app/*`         | `frontend/app/*`          |
| `@ui/*`          | `frontend/ui/*`           |
| `@features/*`    | `frontend/features/*`     |
| `@shared/*`      | `frontend/shared/*`       |
| `@test-utils/*`  | `frontend/test-utils/*`   |

Cross-package imports use the alias. Same-package siblings use relative paths.

## architect-story.task

### Story Read

Read prior contracts and affected production code from committed sources:

| Source | Read when |
| ------ | --------- |
| `frontend/shared/types/` | Always — existing cross-package types |
| `frontend/app/routes.tsx` | Always — registered routes |
| `frontend/app/providers.tsx` | Always — active providers |
| `frontend/features/{domain}/` | When Story modifies existing domain — read affected hooks, api, logic |
| `frontend/app/pages/{page}` | When Story modifies existing page — read structure and prop usage |
| `frontend/shared/utils/`, `frontend/shared/constants/` | When Story relates to utilities the new unit might reuse |

**Checklist:**

- [ ] Existing types, route table, and providers reviewed
- [ ] Affected production code reviewed when Story modifies existing behavior

### External Dependency Inspection and Digestion

References are recorded in the shared store `agents/docs/external-refs.md`, one section per external system (read prior sections before probing — a system may already be digested).

Allowed cheap probes (closed network) — non-mutating, single-shot only:

| Allowed | Forbidden |
| ------- | --------- |
| Read-only single-row / single-record query | Any write, update, or delete |
| File header or sample-row read | Bulk export or full-table scan |
| Schema/metadata introspection | Schema modification |
| Connectivity/credential liveness check | Load or stress probing |

A probe needing anything in the Forbidden column is not cheap — record the gap and request PM for a context doc that supplies it instead.

### Dev Unit Decomposition

Dev units inhabit one of these zones. UI reuse and entry-point updates are not dev units themselves but inform decomposition:

| Zone | Path | Primary writer |
| ---- | ---- | -------------- |
| Domain logic | `frontend/features/{domain}/` | DE |
| Production pages | `frontend/app/pages/` | DE |
| Cross-package utilities | `frontend/shared/utils/`, `frontend/shared/constants/` | DE |
| Cross-package types | `frontend/shared/types/` | AR (contract artifact) |
| UI reuse (read-only) | `frontend/ui/components/`, `frontend/ui/layouts/` | UX (out of dev scope) |

A dev unit lives in one zone; cross-zone units require AC justification.

A unit needing something not yet in the codebase (a new UI component, a runtime library) is evaluated before being hand-built (make-or-buy):

- A needed new UI component is an AC defect (design incompleteness, Step 2) only when the prototype omitted it. When an established library can supply it, it is a make-or-buy candidate.
- Introduce a library only to satisfy a `covers-acs` AC — never for convenience. Settle it before drafting units that depend on it; draft the dev document assuming it exists, and surface the introduction as a task outcome — never write an unmet prerequisite into the document.
- If the introduction changes what the prototype renders or how the user interacts, it requires prototype rework: surface it as a task outcome for the Story to route back to UX, and the library must be made `ui/`-consumable (a wrapping component, or an opened dependency rule). A library with no prototype change is drafted against in place.
- If the introduction creates capability beyond this Story's AC (logging, auth, monitoring), record a promotion-candidate note for PM in the dev document Notes.

AR may add an **External Dependencies** row to the Story directly (Story is AR-owned in Phase 2) when it (a) leaves every AC's interpretation unchanged and (b) needs no new context doc. Otherwise surface it as a task outcome for the Story to route to PM.

**Checklist:**

- [ ] Each dev unit's primary zone identified
- [ ] Cross-zone units justified by AC
- [ ] Anything not yet in the codebase resolved as make-or-buy (prototype-omission defect, or AC-bound introduction surfaced as task outcome)

### Dev Document Drafting

Contract artifact medium and location:

| Contract kind | Medium | Location |
| ------------- | ------ | -------- |
| Cross-package TypeScript types and interfaces | `*.ts` | `frontend/shared/types/` |
| Route registration | TSX | `frontend/app/routes.tsx` |
| Provider composition | TSX | `frontend/app/providers.tsx` |
| Layout shell | TSX | `frontend/app/layout.tsx` |
| App entry | TSX | `frontend/app/main.tsx` |

The five files listed above are AR-owned — updating them counts as contract artifact creation.

**Checklist:**

- [ ] Each cross-dev contract materialized in the listed medium and location
- [ ] No contract expressed only in markdown
- [ ] AR-owned entry-point files updated where contracts require it