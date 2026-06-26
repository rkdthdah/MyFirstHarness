---
id: te.qa.rules
level: 6
type: frontend-web-react
owner: qa
desc: Business-logic and unit test rules for TE agent
---

# TE Frontend Rules

Owner: QA. Consumed by TE per AGENTS.md rules-matching protocol.

## Global Constraints

### File Access

| Permission | Path | Description |
| ---------- | ---- | ----------- |
| rw | `frontend/features/**/*.test.ts` | Feature tests (hooks, domain logic, API contracts) |
| rw | `frontend/shared/utils/**/*.test.ts` | Unit tests (pure functions) |

All other frontend paths are read-only. Read paths governed by `ar.spec.md` *Ownership Boundaries*.

### Dependency Rules

Test files inherit host package dependency rules from `ar.spec.md` *Forbidden Dependencies*.

### Test File Convention

- Co-located: `useFoo.ts` → `useFoo.test.ts`, `foo.ts` → `foo.test.ts`
- `.ts` — Feature and Unit tests render no React component tree
- Fixtures: `{Subject}.fixtures.ts` sibling or `__fixtures__/` sibling folder

### Required File Header

```ts
// @story STORY-XXX
// @ac AC-1, B3
// @subject {subject path relative to frontend/}
// @level feature | unit
```

Empty values use `—`. `@ac` may be `—` for non-AC-driven unit tests.

### Test Infrastructure

- Hooks that read provider context (router, theme, language) render through `renderHook` from `test-utils`. Direct use of RTL's `renderHook` is forbidden — it bypasses the provider stack.
- `setup.ts` resets the DOM and clears storage after each test. Arrange state in the scenario's *given*; never carry it across tests.

## implement-test.task

### Test Implementation

Tooling per level:

| Level   | Subject                          | Location                    | Tools                                                    |
| ------- | -------------------------------- | --------------------------- | -------------------------------------------------------- |
| Feature | hook, domain logic, API contract | `features/**/*.test.ts`     | Vitest, `renderHook` (from `test-utils`), MSW, `vi.mock` |
| Unit    | pure function                    | `shared/utils/**/*.test.ts` | Vitest                                                   |

Implement the strategy the design's *Fixtures & Mock Strategy* already states — TE does not re-choose it. Implementation discipline:

- jsdom-surface strategy — seed the real surface (e.g. `localStorage`) in *given*; never mock the platform surface itself. Supply router/theme/language context via `renderHook` options.
- MSW strategy — mock at the API boundary (handlers under `test-utils/msw/handlers/`); never mock the hook's own logic.
- Feature tests render no full component tree (component level, outside TE scope). Unit tests take no external dependency — no network, no DOM, no real timers (`vi.useFakeTimers`).

`@subject` is the unit's exported entry relative to `frontend/` (e.g. `features/navigation/hooks/useGlobalNav.ts`).

### Run & Classify

```bash
npm run test
```

Runs once (`--passWithNoTests`). A subject whose implementation file does not yet exist surfaces as an import-resolution failure before any test body runs — per `qa.spec.md` *Development Gate Binding*, that is the *unexecuted* signal, not a failure to fix.