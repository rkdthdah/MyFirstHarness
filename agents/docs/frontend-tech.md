---
id: frontend-tech
level: 4
owner: ar
---

# Frontend Tech Stack

> Technical choices, directory structure, and quality policies for the
> Web frontend.
> For agent collaboration, see `frontend-agents.md`.

---

## 1. Overview — 3-Way Development Environment

The frontend runs three independent entry points from a single codebase,
each serving a different validation concern.

| Entry Point          | Command               | URL                                  | Purpose                                  |
| -------------------- | --------------------- | ------------------------------------ | ---------------------------------------- |
| **Storybook**  | `npm run storybook` | `http://localhost:6006/`           | Isolated component validation            |
| **Prototype**  | `npm run proto`     | `http://localhost:5173/proto.html` | Screen-level flow validation (mock data) |
| **Production** | `npm run dev`       | `http://localhost:5173/`           | Live API integration                     |

### Why Three Entry Points

- **Storybook** validates that components behave correctly across all props
  combinations, isolated from user flows.
- **Prototype** validates screen-level flows with hardcoded mock data,
  enabling UX review without a backend.
- **Production** connects to real APIs. Prototype page structure is reused
  — only the data source changes from mock to hook.

This separation enforces **code ownership boundaries** across agents
(`frontend-agents.md`).

---

## 2. Stack

### 2.1 Core Runtime / Build

| Area              | Choice                                | Notes                                                      |
| ----------------- | ------------------------------------- | ---------------------------------------------------------- |
| Bundler           | **Vite 6**                      | ESM native, dev HMR, prod tree-shaking                     |
| Language          | **TypeScript 5.7**              | `strict`, `noUnusedLocals`, `verbatimModuleSyntax`   |
| UI Library        | **React 19**                    | Standard SPA, no Server Components                         |
| Routing           | **react-router-dom v7**         | `BrowserRouter` (production), `HashRouter` (prototype) |
| Module Resolution | TS `paths` + Vite `resolve.alias` | Both share the same mapping                                |

Build output is generated via `tsc -b && vite build`. `index.html`
(production) and `proto.html` (prototype) produce separate bundles.
Storybook uses its own bundler to produce `storybook-static/`.

### 2.2 Styling — Token-Based Design System

| Area          | Choice                                                                      |
| ------------- | --------------------------------------------------------------------------- |
| CSS Framework | **Tailwind CSS v4** (`@tailwindcss/vite` plugin)                    |
| Token Model   | CSS Custom Properties, light / dark sets                                    |
| Theme Toggle  | `ThemeProvider` swaps `data-mode` attribute and token values at runtime |

All colors, radii, shadows, and fonts pass through a single token layer.
Tailwind v4's `@theme inline` directive maps tokens to utility classes.
Dark mode requires no `dark:` prefix in component code — `ThemeProvider`
swaps `:root` CSS variables.

Status colors are restricted to **Forest (pass) / Amber (warn) /
Crimson (fail)** plus process-group accent colors. Emphasis is expressed
through hierarchy, not color.

### 2.3 Internationalization (i18n)

| Item                | Choice                                                                         |
| ------------------- | ------------------------------------------------------------------------------ |
| Dictionary format   | Flat `Record<string, string>`                                                |
| Supported languages | Korean (default), English                                                      |
| Key miss fallback   | en → ko → key itself                                                         |
| Domain separation   | `shared/i18n/dict/{domain}.ts` slices spread-merged into `ko.ts`/`en.ts` |
| API                 | `<LangProvider>` + `useT()` hook (`{ lang, setLang, t, fmt }`)           |

### 2.4 Component Catalog — Storybook

| Item                    | Choice                                                                                   |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| Tool                    | **Storybook 8** (`@storybook/react-vite`)                                        |
| Theme / Language toggle | Storybook manager toolbar wraps `ThemeProvider` · `LangProvider` in controlled mode |

Storybook serves as the **UI Pattern Library**. Agents check
`frontend/storybook.idx.md` for available components before creating new ones.

### 2.5 Dependency Graph Enforcement — dependency-cruiser

The *Forbidden Dependencies* section in `ar.spec.md` is encoded as
`dependency-cruiser` rules (`.dependency-cruiser.cjs`). `npm run depcheck`
exits non-zero on violation. `npm run depgraph` outputs a Graphviz dot
diagram.

### 2.6 Testing

Testing stack (Vitest + Testing Library + MSW + jest-axe + coverage-v8) and
commands are defined in `frontend/qa.spec.md` as the single source of truth.
QA owns these decisions; this document does not duplicate them.

---

## 3. Directory Structure

> Precise ownership and dependency rules live in `frontend/ar.spec.md`
> as the single source of truth. This section explains **why** the
> structure exists.

```
frontend/
├── app/             # Production entry — pages, providers, routes
├── ui/              # Design system — no production dependency
│   ├── components/  # Reusable atoms and organisms
│   ├── layouts/     # Page-level shells
│   ├── styles/      # Tokens, theme, global CSS
│   ├── storybook/   # *.stories.tsx only
│   └── prototype/   # Mock-data screen flow validation
├── features/        # Domain business logic (hooks, api, logic)
└── shared/          # Cross-layer shared contracts
    ├── i18n/        # Dictionaries, LangProvider, useT
    ├── icons.tsx    # SVG icon set
    ├── types/
    ├── constants/
    └── utils/
```

### Dependency Direction

```
app  →  ui (components, layouts, styles)  →  shared
       ↑
  features  →  shared
```

- `ui` CANNOT import `features` or `app`
- `features` CANNOT import `ui/prototype` or `app`
- `shared` CANNOT import any other package
- All rules enforced by `dependency-cruiser`

### Prototype ↔ Production Mirror

`ui/prototype/pages/X.tsx` and `app/pages/X.tsx` share identical component
trees. Only the data source differs — mock imports become hook imports.

---

## 4. Quality Policies

### 4.1 Automated Verification

| Command                     | Verification                         |
| --------------------------- | ------------------------------------ |
| `npm run typecheck`       | TypeScript type correctness          |
| `npm run depcheck`        | Zero forbidden dependency violations |
| `npm run build`           | typecheck + production bundle        |
| `npm run build-storybook` | Catalog static site                  |

Test commands (`npm run test*`) live in `frontend/qa.spec.md` *Runtime Commands*.

### 4.2 File Header Convention

Page files must include a 4-line comment header (`@story`, `@owner`,
`@page`, `@components`). Empty values use `—` (em-dash).

### 4.3 Import Convention (Path Aliases)

- Cross-package: `@ui/*`, `@shared/*`, `@features/*`, `@app/*`
- Same-package siblings: relative paths

### 4.4 Design Token Usage

Colors, radii, shadows, and fonts must use token utilities only.
Raw hex or `rgb(...)` values are rejected — they break dark mode.

---

## 5. Operational Notes

### 5.1 Prototype Uses HashRouter

`proto.html` is a static entry, so `BrowserRouter` cannot match in-app
routes. Prototype uses `HashRouter`; production uses `BrowserRouter`.

### 5.2 dependency-cruiser and MDX

Storybook `Foundations/*.mdx` files are outside the module graph.
Some `_components` files trigger false-positive `no-orphans` warnings
(`severity: warn`, does not fail build).

### 5.3 Tailwind v4 Token Mapping

Token-to-utility mapping lives in `ui/styles/global.css` only.
New tokens must be added there to become available as utility classes.

### 5.4 Storybook Static Output

`frontend/storybook-static/` is in `.gitignore` and not tracked by git.

---

*Last updated: 2026-05-05*
