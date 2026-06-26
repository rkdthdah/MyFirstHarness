---
id: qa.spec
level: 5
owner: qa
type: frontend-web-react
desc: Frontend test pyramid, tooling, conventions, coverage, and gate bindings
---

# Frontend Test Specification

Single source of truth for frontend test stack bindings.

Test code ownership and directory boundaries: see `ar.spec.md` *Ownership Boundaries*.

## Test Pyramid

| Level       | Scope                              | Location                                                   |
| ----------- | ---------------------------------- | --------------------------------------------------------- |
| Unit        | Pure functions                     | `shared/utils/**/*.test.ts`                               |
| Component   | Single React component             | `ui/components/**/*.test.tsx`, `ui/layouts/**/*.test.tsx` |
| Integration | Page-level flows (mock data)       | `ui/prototype/pages/**/*.test.tsx`                        |
| Feature     | Hooks, API contracts, domain logic | `features/**/*.test.ts`                                   |
| E2E         | Browser + backend, key journeys    | TBD                                                       |

Per-level verification scope is in *Test Scope per Type*; per-level ownership follows `ar.spec.md` *Ownership Boundaries* (`ui/**/*.test.tsx` QA-authored, `features/**` and `shared/utils/**` test files TE-authored).

## Tooling

| Concern              | Tool                          |
| -------------------- | ----------------------------- |
| Test runner          | Vitest                        |
| DOM environment      | jsdom                         |
| Component rendering  | @testing-library/react        |
| DOM matchers         | @testing-library/jest-dom     |
| User interaction     | @testing-library/user-event   |
| Accessibility        | jest-axe (Vitest compatible)  |
| API mocking          | MSW (Mock Service Worker)     |
| Module mocking       | `vi.mock`                     |
| Coverage             | @vitest/coverage-v8           |
| E2E                  | TBD                           |

## Test File Convention

- Co-located: `Foo.tsx` → `Foo.test.tsx`, `useFoo.ts` → `useFoo.test.ts`.
- `.tsx` when rendering React, `.ts` otherwise.
- Fixtures: `{Subject}.fixtures.ts` sibling (local) or `__fixtures__/` sibling folder (shared in same package).
- Test files inherit host package dependency rules from ar.spec *Forbidden Dependencies* — no per-level forbidden lists.

### Test infrastructure — `frontend/test-utils/`

```text
frontend/test-utils/
├── setup.ts                ← Vitest setupFiles entry (jest-dom + jest-axe matchers, RTL cleanup)
├── render.tsx              ← custom render with providers
├── renderHook.tsx          ← custom renderHook with providers
├── msw/
│   ├── server.ts
│   └── handlers/
│       └── index.ts        ← exported HttpHandler[] (initially empty)
└── builders/
```

QA-owned, outside the QA-designs / TE-implements split — its correctness rests on cross-use (a wrong wrapper breaks every dependent test) plus co-located contract tests (e.g. `render.test.tsx`). Import boundaries fixed by `ar.spec.md` *Forbidden Dependencies*, enforced by `depcheck`.

All Component and Integration tests render through `test-utils/render.tsx`. Direct use of RTL's `render` bypasses required providers and is forbidden.

`render.tsx` wraps subjects in (outer → inner): `MemoryRouter` → `LangProvider` → `ThemeProvider`. Options accept `route` (string | string[], default `["/"]`), `lang` (default `"ko"`), `mode` (default `"light"`). Returns `{ user, ...rtlResult }` with a `userEvent.setup()` instance.

`renderHook.tsx` wraps the hook under test in the same provider stack with the same options — for Feature tests of hooks that read router, theme, or language context. Direct use of RTL's `renderHook` bypasses these providers and is forbidden.

`setup.ts` runs after each test: RTL `cleanup`, `localStorage.clear()`, and removal of the `data-mode` / `lang` attributes on the document root. Tests therefore start from a clean DOM and empty storage without per-test teardown — arrange state in *given*, never carry it across tests.

### Introducing test tooling

Stack bindings for introducing a test library or `test-utils/` helper (introduction is QA-authored — QA owns this spec):

- An introduction is an edit to this spec (a new entry in *Tooling*, a new `test-utils/` module) or to `test-utils/` itself.
- A test or test design needing a helper not yet present is authored assuming it exists; the need surfaces as a task outcome, never as an unmet prerequisite in the document.
- Test tooling does not touch the prototype, so an introduction is added in place and never bounces upstream.

### jsdom limits

jsdom computes no layout and honours no media queries. ACs needing rendered geometry (text overlap, responsive breakpoints, visual regression) cannot be checked here and are dispositioned `N/A` with the cause stated. A future browser-level E2E layer (TBD) covers this class going forward.

### Required file header

```tsx
// @story STORY-XXX
// @ac AC-1, AC-3
// @subject {subject path relative to frontend/}
// @level component | integration | unit | feature
```

Empty values use `—` (mirrors ar.spec header convention). `@ac` may be `—` for non-AC-driven tests (e.g. utility unit tests).

## Test Scope per Type

### Component test

Verifies: rendered output, accessibility (axe), i18n key application, state transitions, user-event handlers, prop variations.

### Integration test

Verifies: page-level user flows against prototype mock data, component composition, prototype-router navigation.

Production pages (`app/pages/`) are covered transitively via `mirrorcheck` (per `ar.spec.md`) — no separate production page tests are written.

### Feature test

Verifies: hook behavior (state, side effects), API contracts via MSW, domain logic, error paths.
Forbidden: rendering full component trees. Use `renderHook` (from `test-utils`) for hooks.

Mock strategy by hook layer:
- Hook with no backend call — seed the real jsdom platform surface it reads (e.g. `localStorage`) in *given*; supply router/theme/language context via `renderHook` options. No MSW, no mocking of `localStorage` itself.
- Hook with a backend call — mock the API boundary via MSW; do not mock the hook's own logic.

Production pages (`app/pages/`) are covered transitively via `mirrorcheck` (per `ar.spec.md`) — no separate production page tests are written.

### Unit test

Verifies: pure function input/output, edge cases, error conditions.
Forbidden: any external dependency — no network, no DOM, no real timers (`vi.useFakeTimers`).

## Coverage Policy

Mode: **report-only** (no CI gate). Metrics: line, branch, function. Numeric gates added once baselines stabilize.

Excluded paths:
- `**/*.test.{ts,tsx}`
- `**/*.stories.tsx`
- `**/__fixtures__/**`, `**/*.fixtures.ts`
- `**/*.d.ts`
- `**/main.tsx`
- `app/routes.tsx`, `app/providers.tsx`, `app/layout.tsx`
- `shared/types/**`
- `ui/prototype/mocks/**`
- `frontend/test-utils/**`

## Development Gate Binding

The gate on a `dev-XXX-NN` is decided by the paired `test-XXX-NN` — located from the dev unit's subject via the *Test File Convention* — run against DE's implementation.

A subject whose implementation file does not yet exist surfaces as a Vitest import-resolution failure before any test body runs — the test is authored but unexecuted, not failing. This pending state is the expected pre-implementation position; the gate is reached only once the subject exists and the paired test runs green.

## Runtime Commands

| Command                 | Purpose                              |
| ----------------------- | ------------------------------------ |
| `npm run test`          | run all tests once (`--passWithNoTests`) |
| `npm run test:watch`    | watch mode for active development    |
| `npm run test:cov`      | full coverage report (`--passWithNoTests`) |