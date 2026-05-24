---
id: frontend-agents
level: 4
owner: ar
---
# Frontend Agent Collaboration

> How agents divide responsibilities and collaborate on the Project
> frontend.
> For tech stack details, see `frontend-tech.md`.
> The authoritative rules document for agents is `frontend/ar.spec.md`.

---

## 1. Agent Roles

Five agent types collaborate on the frontend. Code-level permissions are defined in `ar.spec.md` Ownership Boundaries.
Document-level permissions are defined in `ar.spec.md` Document Ownership.

| Agent        | Role                                              | rw Directories                                                         |
| ------------ | ------------------------------------------------- | ---------------------------------------------------------------------- |
| **UX** | UI Designer / Pattern Library maintainer          | `ui/` (*.tsx), `shared/i18n/`, `shared/icons.tsx`                |
| **QA** | UI test authoring, test specification maintenance | `ui/**/*.test.tsx`, `qa.spec.md`, `qa.qa.rules.md`, `te.qa.rules.md`, `de.qa.rules.md` |
| **TE** | Business logic test implementation                | `features/**/*.test.ts`, `shared/utils/**/*.test.ts`               |
| **DE** | Developer (domain logic implementation)           | `features/`, `app/pages/`, `shared/{constants,utils}`            |
| **AR** | Architect (app skeleton, infra, rules)            | `app/` (all), `shared/types/`, `ar.spec.md`                      |

---

## 2. Responsibility Matrix

### Design

| Area                                          | UX | QA | TE | AR | DE |
| --------------------------------------------- | :-: | :-: | :-: | :-: | :-: |
| UI design (prototype)                         | ✓ |    |    |    |    |
| UX Spec authoring                             | ✓ |    |    |    |    |
| Directory structure design                    |    |    |    | ✓ |    |
| Dependency rules / ownership                  |    |    |    | ✓ |    |
| Component interface definition (shared/types) |    |    |    | ✓ |    |
| Test scenario design (Given-When-Then)        |    | ✓ |    |    |    |

### Implementation

| Area                                | UX | QA | TE | AR | DE |
| ----------------------------------- | :-: | :-: | :-: | :-: | :-: |
| ui/components, layouts              | ✓ |    |    |    |    |
| ui/styles (theme, tokens)           | ✓ |    |    |    |    |
| ui/prototype (pages, mocks, routes) | ✓ |    |    |    |    |
| ui/storybook (.stories.tsx)         | ✓ |    |    |    |    |
| shared/i18n, shared/icons           | ✓ |    |    |    |    |
| features/ (hooks, api, logic)       |    |    |    |    | ✓ |
| app/pages (production assembly)     |    |    |    |    | ✓ |
| app/routes, providers, layout, main |    |    |    | ✓ |    |
| shared/types                        |    |    |    | ✓ |    |
| shared/constants, utils             |    |    |    |    | ✓ |

### Testing

| Area                                          | UX | QA | TE | AR | DE |
| --------------------------------------------- | :-: | :-: | :-: | :-: | :-: |
| Prototype smoke check (`npm run proto`)     | ✓ |    |    |    |    |
| Type check (`npm run typecheck`)            | ✓ |    |    | ✓ |    |
| Dependency check (`npm run depcheck`)       |    |    |    | ✓ |    |
| UI component tests (a11y, i18n, states)       |    | ✓ |    |    |    |
| UI integration tests (page-level flows)       |    | ✓ |    |    |    |
| Unit / Feature test design (Given-When-Then)  |    | ✓ |    |    |    |
| Unit / Feature test implementation            |    |    | ✓ |    |    |
| Test review & approval                        |    | ✓ |    |    |    |
| DE development gate (tests pass on impl)      |    |    |    |    | ✓ |
| Final verification & regression               |    | ✓ |    |    |    |
| E2E                                           | — | — | — | — | — |

### Infrastructure / Management

| Area                                                  | UX | QA | TE | AR | DE |
| ----------------------------------------------------- | :-: | :-: | :-: | :-: | :-: |
| Architecture Specification (ar.spec.md)               |    |    |    | ✓ |    |
| Test Specification (qa.spec.md)                       |    | ✓ |    |    |    |
| depcheck / typecheck tooling                          |    |    |    | ✓ |    |
| Vitest / coverage tooling                             |    | ✓ |    |    |    |
| AR-owned rules (ux.ar, de.ar)                         |    |    |    | ✓ |    |
| QA-owned rules (qa.qa, te.qa, de.qa)                  |    | ✓ |    |    |    |
| Storybook catalog (storybook.idx.md)                  | ✓ |    |    |    |    |
| Story-file mapping (prototype.idx.md)                 | ✓ |    |    |    |    |

---

## 3. Story Workflow

Every feature receives a **STORY-XXX** identifier and a dedicated branch
`story/STORY-XXX` created from `develop` by PM. All agents commit
sequentially to this branch. See `docs/git-convention.md` for full rules.

### Story Flow

```
PM → UX (design + prototype) → PM → QA (testability + UI test)
   → AR (architecture design) → QA (test design)
   → TE (test implementation) → QA (test review)
   → DE (dev, tests pass) → AR (route integration) → QA (verification)
   → AR (merge readiness) → PM (complete + merge)
```

### What Happens at Each Stage

1. **UX** creates reusable components in `ui/components` or `ui/layouts`,
   registers them in Storybook, and builds mock-based prototype pages
   in `ui/prototype/pages` through co-design with the user.
2. **UX** writes the UX Spec and hands off to PM.
3. **QA** verifies AC testability and writes/runs UI component tests and
   integration tests against the prototype. Failures return to UX
   (before AR, minimizing rollback cost).
4. **AR** designs architecture, defines interfaces in `shared/types/`,
   and produces implementation design documents (`dev-XXX-XX.md`).
5. **QA** designs Given-When-Then test scenarios and produces test
   design documents (`test-XXX-XX_designed.md`), one per AR design document.
6. **TE** implements test code based on test design documents.
   QA reviews and approves (`complete`) or requests revision (`revision-requested`).
7. **DE** implements hooks and APIs in `features/{domain}` and creates
    production pages in `app/pages` by replacing mock imports with hook
    imports.
8. **AR** registers routes in `app/routes.tsx` and validates the
    integrated build.
9. **QA** performs final verification — runs all tests, regression check,
   and updates Test Coverage Map.
10. **AR** runs merge readiness checks (typecheck, depcheck, build, tests).
11. **PM** completes the story, merges story branch to `develop`, removes
   worktree and deletes the story branch.

### Prototype → Production Transition

Production pages mirror prototype pages — identical component trees,
different data sources. Transitioning requires replacing mock imports
with hook imports only.

---

## 4. Conflict Prevention

- **Ownership table** defines who can modify which directories.
- **Forbidden Dependencies** enforce import direction, preventing
  cross-boundary coupling.
- **STORY identifiers** isolate concurrent work.
- `npm run depcheck` automatically verifies that agents only touched
  their permitted directories.

---

## 5. Agent Reference Documents

| Document         | Location                      | Purpose                                                |
| ---------------- | ----------------------------- | ------------------------------------------------------ |
| ar.spec.md       | `frontend/ar.spec.md`       | Architecture rules, ownership, dependencies (AR owned) |
| qa.spec.md       | `frontend/qa.spec.md`       | Test specification (QA owned)                          |
| ux.ar.rules.md   | `frontend/ux.ar.rules.md`   | UX agent rules (AR owned)                              |
| de.ar.rules.md   | `frontend/de.ar.rules.md`   | DE agent build rules (AR owned)                        |
| qa.qa.rules.md   | `frontend/qa.qa.rules.md`   | QA agent rules (QA owned)                              |
| te.qa.rules.md   | `frontend/te.qa.rules.md`   | TE agent rules (QA owned)                              |
| de.qa.rules.md   | `frontend/de.qa.rules.md`   | DE agent test-gate rules (QA owned)                    |
| storybook.idx.md | `frontend/storybook.idx.md` | UI asset catalog (UX owned)                            |
| prototype.idx.md | `frontend/prototype.idx.md` | Story-to-file index (UX owned)                         |

ar.spec.md is the single source of truth for architecture. AR extracts
agent-specific rules into AR-owned rules files (`ux.ar.rules.md`, `de.ar.rules.md`).
qa.spec.md is the single source of truth for testing. QA extracts
test rules into QA-owned rules files (`qa.qa.rules.md`, `te.qa.rules.md`, `de.qa.rules.md`).

Rules files follow the `{consumer}.{owner}.rules.md` convention.
A single consumer may read multiple rules files when multiple owners govern its work
(e.g. DE reads both `de.ar.rules.md` for build and `de.qa.rules.md` for test gates).
Agents read only their own rules files — never spec files directly.

---

*Last updated: 2026-05-15*
