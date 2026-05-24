---
id: review-testability.task
level: 3
owner: or
---

## Purpose

Apply QA's lens to a Story before AR architecture. Last cheap gate before AR design cost is paid. Verify each AC can be turned into a deterministic, executable test, and identify what the Story should specify but does not.

## Workflow

### 1. Source Reading

1. Read the full body of each AC, classification, context, User-Facing Notes, Domain Terms, External Dependencies
2. Read the UX spec if linked, and parent/related stories if cited
3. Open the Glossary; for each Domain Terms entry resolve spelling, kind, definition
4. For each External Dependencies row with Context Doc set, read the referenced doc in `docs/context/`

**Checklist:**

- [ ] Every AC body read in full
- [ ] Domain Terms resolved in Glossary (spelling, kind, definition)
- [ ] External Dependencies read; Context Docs read where set
- [ ] UX spec and parent/related stories read if applicable

### 2. Per-AC Testability Evaluation

For each AC, simulate writing the test. The AC fails if any of:

- **Observable signal** — a concrete surface to assert against (UI state, returned value, emitted event, persisted record, log line), specific enough to assert without prescribing implementation.
  - Insufficient: "validates input correctly", "is rejected"
  - Sufficient: "an error message is shown and the form remains editable"
  - Over-specified (do not require): "returns HTTP 400 with body `{code: E_INVALID}`" — protocol detail is AR territory.
- **Trigger specification** — the input/action/condition is named within the AC. Checks AC self-containment, not variant coverage (Step 3).
- **Boundary precision** — exact values and inclusivity for ranges, thresholds, limits.
- **Measurable threshold** (NFR only) — quantitative target with measurement condition.
- **Ubiquitous Language compliance** — every domain term in the AC appears in Domain Terms with Glossary-matching spelling. Domain terms = nouns/verbs carrying business meaning (entities, value objects, states, roles, commands, domain events). Excluded: UI interactions, generic system actions (save/load/display) unless domain-meaningful, generic English, Story meta-language (user/system/screen). Substitution test: if a synonym loses business meaning, it is a domain term.
- **Kind consistency** — AC usage matches the term's registered kind (e.g. a `command` not used as a noun result; a `domain-event` not used as a trigger). Flag the inconsistency only — whether the kind itself is correct is AR's call.
- **UX consistency** (UI-involved AC only) — AC trigger and signal resolvable to specific UX spec states.

Mark each AC `[x]` testable or `[ ]` not, with a reason naming the failed criterion and AC location. Never propose registration or wording.

**Checklist:**

- [ ] Every AC evaluated against applicable criteria
- [ ] Each non-testable AC has a reason naming the failed criterion
- [ ] UI-involved AC cross-checked against UX spec

### 3. Cross-AC Gap Analysis

Catches what is absent. For each dimension: populated OR explicitly N/A → pass. Do not judge N/A reason quality (PM Discovery's responsibility).

- **Category coverage** — Normal / Alternative / Edge / Validation / Non-functional
- **Lifecycle completeness** — when an entity/state is introduced or modified, CRUD-equivalents addressed or scoped out
- **Failure paths** — failure of each external dependency or precondition covered or scoped out
- **Concurrency & idempotency** — repeated action behavior specified or scoped out
- **Permission & role variants** — each user type's behavior specified or scoped out
- **Regression surface** — existing behavior that must NOT break is named, or "no regression surface" stated
- **AC interaction** — no two AC conflict, ambiguous in priority, or depend on undefined sequencing
- **Implicit domain dependency** — domain concepts required for an AC to make sense but not directly named must appear in Domain Terms
- **Action coverage** — for every domain entity in Domain Terms, verbs the AC uses on it are either registered as commands/events or fall under Step 2's non-domain exclusions
- **External Dependency completeness** — every external system the AC actually touches appears in External Dependencies; rows without sufficient detail (and no Context Doc) flagged

Each gap → row in Testability Results with `AC ID` = `—`, `Testable` = `—`, `Notes` = missing dimension and what PM must add.

**Checklist:**

- [ ] All dimensions reviewed (presence-only)
- [ ] Each gap recorded as a row

### 4. Test Feasibility Analysis

A spec that reads well but can't be exercised in test is not testable.

- **External dependencies** — each external system (explicit OR implicit: auth, time/clock, permission, external API, message broker) has a stub/mock strategy or is out of scope. Use External Dependencies + Context Docs as primary input.
- **Determinism** — time, randomness, ordering, latency: each non-deterministic input has a control surface
- **State setup** — preconditions reachable without manual factory data
- **Verification surface** — each observable signal has a corresponding test API

Infeasibility treated as non-testability: AC marked `[ ]` with reason.

**Checklist:**

- [ ] Each dependency (explicit and implicit) has a stubbing path
- [ ] Each non-deterministic input has a control surface
- [ ] Each precondition reachable in test
- [ ] Each observable signal has a verification API

### 5. Results Documentation

Fill Story `Testability Results`. Each defect under the single most specific criterion — no defect recorded twice.

- AC row: `AC ID` = AC id, `Testable` = `[x]` / `[ ]`, `Notes` = empty or reason
- Gap row: `AC ID` = `—`, `Testable` = `—`, `Notes` = missing dimension

**Checklist:**

- [ ] Every AC and every gap has a row
- [ ] Every non-testable / gap row cites a specific criterion
- [ ] No defect recorded twice
- [ ] Story file saved

> Any non-testable AC OR any gap → outcome `Revision Required`
> All testable AND no gaps → outcome `Completed`

## Instructions

- Last cheap gate before AR. Bias toward `Revision Required` when in doubt — one PM round-trip costs less than one AR redesign.
- `Revision Required` signals model precision, not PM work quality. PM Minor Revision absorbs most cases cheaply. Do not soften criteria to avoid triggering it.
- The Glossary embodies the project's Ubiquitous Language (DDD). QA never registers terms, never proposes wording — flag with exact location and let PM decide register / rename / rewrite / declare non-domain.
- Domain model correctness (entity vs value object, splitting/merging concepts, aggregate boundaries) is AR's. QA verifies AC is internally consistent with registered kinds, not whether the kinds are modeled correctly.
- QA reviews testability, not desirability. Don't redesign the Story; identify what blocks testing it.
- Reasons must cite a specific criterion — "vague" is not a reason; "no trigger named in B1" is.
- Task outcomes: `Revision Required` | `Completed`