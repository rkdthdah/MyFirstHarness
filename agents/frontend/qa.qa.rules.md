---
id: qa.qa.rules
level: 6
type: frontend-web-react
owner: qa
desc: UI test rules for QA agent
---

# QA Frontend Rules

Owner: QA. Consumed by QA per AGENTS.md rules-matching protocol.

## Global Constraints

### File Access

| Permission | Path | Description |
| ---------- | ---- | ----------- |
| rw | `frontend/ui/components/**/*.test.tsx`, `frontend/ui/layouts/**/*.test.tsx` | Component tests |
| rw | `frontend/ui/prototype/pages/**/*.test.tsx` | Integration tests |
| rw | `frontend/test-utils/` | Test infrastructure |
| rw | `agents/docs/stories/STORY-XXX/STORY-XXX_*.md` | Story UI Test Coverage section |

All other frontend paths are read-only. Read paths governed by `ar.spec.md` *Ownership Boundaries*.

### Dependency Rules

Test files inherit host package dependency rules from `ar.spec.md` *Forbidden Dependencies*.

### Test File Convention

- Co-located: `Foo.tsx` → `Foo.test.tsx`
- `.tsx` when rendering React, `.ts` otherwise
- Fixtures: `{Subject}.fixtures.ts` sibling or `__fixtures__/` sibling folder

### Required File Header

```tsx
// @story STORY-XXX
// @ac AC-1, B3
// @subject {subject path relative to frontend/}
// @level component | integration
```

Empty values use `—`. `@ac` may be `—` for non-AC-driven tests. AC IDs follow Story numbering (`1`, `A1`, `B3`, `C1`, `D2`).

### Test Infrastructure

Per qa.spec, all Component and Integration tests render through `frontend/test-utils/render.tsx`. Direct use of RTL's `render` is forbidden — bypasses required providers (MemoryRouter, LangProvider, ThemeProvider).

## test-ui.task

### Coverage Mapping

Disposition each `covers-acs` ID:

| AC signal scope | Test Level | Location |
| --------------- | ---------- | -------- |
| Single component or layout render/event | Component | `ui/components/**/*.test.tsx`, `ui/layouts/**/*.test.tsx` |
| Page-level flow, multi-component composition, navigation | Integration | `ui/prototype/pages/**/*.test.tsx` |

UX spec anchor resolves to one row in: Data States, Component Interaction, Validation Rules, or Navigation.
Subject path in `@subject` header is relative to `frontend/` (e.g. `ui/components/Topbar.tsx`).

### Test Implementation

Tooling per level:

| Level | Tools |
| ----- | ----- |
| Component | RTL (`render` from `test-utils`), `user-event`, `jest-axe` |
| Integration | RTL (`render` from `test-utils`), `user-event`, prototype mock imports |

Component test must include:

- Accessibility assertion via `jest-axe`
- i18n key resolution check when AC covers display text (assert resolved text, not raw key)
`render` from `test-utils` returns `{ user, ...rtlResult }` with a pre-configured `userEvent.setup()` — prefer the bundled `user` over calling `userEvent.setup()` manually.
Integration test:

- Mock data imported directly from `ui/prototype/mocks/` — no MSW (prototype is API-less)
- If a test needs a `test-utils` helper or matcher not yet present, author the test assuming it exists and surface the introduction as a task outcome — never block on an unmet prerequisite

### Run & Verify

```bash
npm run test
```

## design-test.task

### Coverage Mapping

Disposition each `covers-acs` ID (per design-test W3):

| Disposition | When | Note |
| ----------- | ---- | ---- |
| `scenario` | Behavior observable at the unit's boundary (Feature level) | Designed in Scenario Design |
| `covered-by-ui-test` | Behavior verified by a Phase 1 UI test | Cross-reference Story *UI Test Coverage* |
| `defer-to-verify` | Cross-unit, not observable at this boundary | Verified at integration |
| `N/A` | No verifiable behavior in this unit | State the cause — incl. jsdom limits (e.g. "jsdom computes no layout") for geometry/visual ACs |

Business-logic units test at **Feature** level (`features/**/*.test.ts`). `@subject` is the unit's exported entry relative to `frontend/`.

### Scenario Design

| Hook layer | Strategy |
| ---------- | -------- |
| No backend call | Seed real jsdom surface (`localStorage`) in *given*; supply router/theme/lang context through a provider wrapper. No MSW |
| Backend call | Mock the API boundary via MSW; never mock the hook's own logic |

- Hooks that read provider context must render through a `test-utils` wrapper, never RTL's `renderHook` directly. If a needed wrapper is not yet in `test-utils/`, design the scenarios assuming it exists and surface the introduction as a task outcome — never record it as an unmet prerequisite in the document.
- `setup.ts` resets DOM and storage after each test; scenarios assume isolation and arrange state in *given*.
- File header `@level feature` (or `unit` for pure functions).

### Handoff Composition

- Subject path is the dev unit's exported entry relative to `frontend/`.
- Test files inherit import/dependency limits from `ar.spec.md` *Forbidden Dependencies* — no per-design list.