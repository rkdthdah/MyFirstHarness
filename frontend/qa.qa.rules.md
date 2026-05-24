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

Per qa.spec, all Component and Integration tests render through `frontend/test-utils/render.tsx`. Direct use of RTL's `render` is forbidden — bypasses Theme/Lang providers.

## test-ui.task

### Coverage Mapping

Test level decision (per qa.spec *Test Pyramid*):

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

Integration test:
- Mock data imported directly from `ui/prototype/mocks/` — no MSW (prototype is API-less)

### Run & Verify

```bash
npm run test
```