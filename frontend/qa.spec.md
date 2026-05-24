---
id: qa.spec
level: 5
owner: qa
type: frontend-web-react
desc: Frontend test pyramid, tooling, conventions, coverage, and DE gate
---

# Frontend Test Specification

Single source of truth for frontend testing.

Test code ownership and directory boundaries: see `ar.spec.md`

## Test Pyramid

| Level       | Scope                              | Location                                                            | Owner |
| ----------- | ---------------------------------- | ------------------------------------------------------------------- | ----- |
| Unit        | Pure functions                     | `shared/utils/**/*.test.ts`                                         | TE    |
| Component   | Single React component             | `ui/components/**/*.test.tsx`, `ui/layouts/**/*.test.tsx`           | QA    |
| Integration | Page-level flows (mock data)       | `ui/prototype/pages/**/*.test.tsx`                                  | QA    |
| Feature     | Hooks, API contracts, domain logic | `features/**/*.test.ts`                                             | TE    |
| E2E         | Browser + backend, key journeys    | TBD                                                                 | TBD   |

**Authoring rule:**
- QA writes UI tests (Component, Integration) directly.
- TE implements Unit/Feature tests from QA's test design documents.

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
├── render.tsx              ← custom render with providers
├── msw/
│   ├── server.ts
│   └── handlers/
└── builders/
```

All Component and Integration tests render through `test-utils/render.tsx`. Direct use of RTL's `render` bypasses Theme/Lang providers and is forbidden.

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
Forbidden: rendering full component trees. Use `renderHook` for hooks.

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

## Test Design Document Lifecycle

Business logic tests only (Unit, Feature). UI tests have no design document.

```
test-{STORY}-{AC}-{NN}_{status}.md
```

- Fields per `harness.md` §2 *Implementation Documents*.
- Each `test-XXX-NN` pairs 1:1 with `dev-XXX-NN`. If a dev document has no runtime business logic to verify (type-only, mock→hook swap, constants), QA creates a placeholder test document at `_complete` status with rationale and skips test code.
- Filename renamed on every transition.

## DE Development Gate

QA's gate on DE work for a `dev-XXX-NN`:

All tests in the matching `test-XXX-NN` set pass against DE's implementation.

## Runtime Commands

| Command                 | Purpose                              |
| ----------------------- | ------------------------------------ |
| `npm run test`          | run all tests once                   |
| `npm run test:watch`    | watch mode for active development    |
| `npm run test:cov`      | full coverage report                 |