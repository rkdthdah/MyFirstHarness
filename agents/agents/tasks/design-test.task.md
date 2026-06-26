---
id: design-test.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/test.tmp.md
rules:
  - @~/{{ProjectRoot}}/frontend/qa.qa.rules.md
  - @~/{{ProjectRoot}}/backend/qa.qa.rules.md
---

## Purpose

Transform a story's contract-level dev documents into outcome-level test designs that TE implements. Two products: a paired test per dev unit (closing the units that need no unit-level verification), and one story suite (`test-XXX-00`) designing the integration, contract, and E2E scenarios for the acceptance criteria no single unit can verify. Each paired design is read by TE in isolation; the suite is QA's own instrument. Designs specify observable outcomes, never implementation. Where a contract admits no observable scenario, that is reported as an observed block, never forced.

## Workflow

### 1. Story Intake

The whole story's dev units arrive together. Read them as a set, not one at a time.

1. Read every dev document — frontmatter (`test-kind`, `covers-acs`), Behavioral Contract, and Files / Public Signatures / Schemas for subject identification
2. Read the Story AC body for every AC referenced across the units — needed for the one-line restatements carried into the designs
3. Read the Story and UX spec as needed to validate the contracts' behaviors against requirements and to phrase scenarios — the dev documents distill both, but verifying outcomes against their sources is QA's own responsibility

**Checklist:**

- [ ] Every dev document's frontmatter and Behavioral Contract read
- [ ] AC body read for every referenced AC
- [ ] Story and UX spec consulted where outcome validation requires, or N/A

### 2. Per-Dev Paired Design

For each dev unit, produce its paired test document. Branch on `test-kind`:

- **`placeholder`** — Closing document recording why no unit-level design exists (no runtime logic to verify). No scenarios. DoD: `Designed`; `Implemented` / `Reviewed` `[N/A]`. Owner stays QA
- **`ar-integration`** — Closing document pointing to the dev document's verification surfaces; verification is deferred to integration, not performed at unit level. Do not restate the surfaces. Disposition each `covers-acs` ID `defer-to-verify` (or `N/A` with cause where a stack limit precludes verification, per qa rules), so the suite has the per-AC list. DoD: `Designed`; `Implemented` / `Reviewed` `[N/A]`. Owner stays QA
- **`business-logic`** — Map then design:

  1. **Disposition** every `covers-acs` ID:
     - `scenario` — unit-level behavior observable at this unit's boundary. An AC may have facets this unit does not own; disposition only the facet within this boundary, leave the rest to its owner
     - `covered-by-ui-test` — already fully verified at this boundary by a Phase 1 UI test (cross-reference the Story UI Test Coverage)
     - `defer-to-verify` — cross-unit behavior not observable at this boundary; verified at integration. This is the suite's input — collect it in Step 3. Excessive `defer-to-verify` on one unit signals a decomposition issue — record it as a note
     - `N/A` — contributes no verifiable behavior here. Record why
  2. **Design** each `scenario`-disposed behavior as **given** (precondition / entry state) → **when** (trigger) → **then** (observable terminal state). Link each to its AC ID(s) with a one-line restatement — the paired design stands alone for TE. Assert the observable terminal state, never timing, internal path, or call sequence

If a `scenario`-disposed behavior cannot be expressed observably against this contract — unobservable terminal state, signatures that cannot produce the *given*, or no behavior to design from — it is an observed block. Even one blocks this unit's design; scenarios drafted for its other ACs are discarded, and no paired test document is kept. Record the observed fact (which behavior, why unobservable) in the dev document's Handoff Notes — the unit returns to AR. This unit's outcome is `Design Blocked`. Scope is this unit only — a wider decomposition problem is AR's call. Other units in the story are unaffected; continue designing them

A closed unit still yields a paired document — stating *why no unit-level design exists*, not what verifies it instead (owned downstream).

**Forbidden in scenarios:** assertion code or exact mock implementation; timing or internal-path assertions; mock call-count assertions unless the contract mandates the count.

**Checklist:**

- [ ] Each dev unit has a paired document
- [ ] `placeholder` / `ar-integration`: closing document with reason; no scenarios; surfaces not restated; `ar-integration` ACs dispositioned `defer-to-verify` / `N/A`
- [ ] `business-logic`: every `covers-acs` ID dispositioned; each `scenario` behavior designed as given/when/then with AC restatement, or reported as an observed block
- [ ] Scenarios assert observable terminal state only; nothing forbidden present
- [ ] Each blocked unit recorded `Design Blocked` (no paired document kept; fact in the dev document); other units still designed

### 3. Story Suite Design

Collect every `defer-to-verify` AC from Step 2. These are the acceptance criteria no single unit verifies — design `test-XXX-00` to cover them.

1. **No deferred AC** — the story has no integration-level verification. Produce a closing suite: DoD `Designed`; `Implemented` / `Reviewed` `[N/A]`; reason recorded. Owner stays QA. Skip the rest of this step
2. **Scope** — state what the suite verifies that no unit can: the units composed and the boundaries where they meet, the contract artifacts both sides honor, and the journeys verified end to end. Not a single subject
3. **Scenarios** — for each deferred AC, design a given/when/then at integration or journey level (observable terminal state across the composed units, not within one), and tag its level:
   - `integration` — units composed; verified by execution at verification time
   - `contract` — the shape two sides exchange matches the agreed contract artifact; verified by execution
   - `e2e` — the user journey through the running stack; **scenario only** — the code is authored and run at verification, so design the journey, not its implementation
   Record each deferred AC in AC Coverage as `scenario`-disposed against its suite scenario(s)
4. **Fixtures & Environment** — real or stubbed boundaries, seeded state, environment the scenarios require; tooling and locations per rules

If a deferred behavior cannot be expressed observably at integration level — no terminal state observable across the composition — the integration itself is not designable, which is a decomposition problem, not a suite to force. The units whose composition the un-designable scenario spans are `Design Blocked` (uncheck their `Designed`, owner → AR); record the observed fact in each returning unit's Handoff Notes. Where the span is unclear, the whole story is blocked. The suite stays with QA, provisional. A unit blocked here is blocked for the same reason as in Step 2 — AR owns the contract

If a dev unit is `Design Blocked` from Step 2, its deferred ACs cannot be fully designed into the suite yet — design the rest, note the dependency on the blocked unit in the suite's Notes, and leave its portion for revision once AR returns the unit. The suite stays provisional with QA

**Checklist:**

- [ ] All `defer-to-verify` ACs collected
- [ ] No deferred AC → closing suite (`Implemented` / `Reviewed` `[N/A]`, reason); else Scope, Scenarios (level-tagged), Fixtures authored
- [ ] Every deferred AC covered by a suite scenario, or its portion deferred to a blocked unit's revision
- [ ] Un-designable integration reported; spanned units `Design Blocked` (or whole story where unclear)

### 4. Handoff

Finalize every document per template.

- **Baseline** — `—` on each; these are initial designs with no prior implementation
- **Paired Subject** — file / export under test, from the dev document's Files and Public Signatures
- **Suite ownership** — holds any `integration` / `contract` scenario → owner → TE; only `e2e` or closing → `Implemented` / `Reviewed` `[N/A]`, owner stays QA
- **TE self-containment** — each paired and the suite must not reference the Story, UX spec, or another dev unit by name without restating the content TE needs. On failure → return to the relevant step and supply the restatement

**Checklist:**

- [ ] Every paired Subject identifies the file / export under test
- [ ] Suite owner set by its scenario content (TE if any integration/contract; else QA)
- [ ] Fixtures stated at outcome level, no tooling specifics
- [ ] Every document self-contained for TE — no unrestated external references

> Per dev unit: scenarios designed or unit closed → `Designed`; a behavior un-expressable against the contract → `Design Blocked` (no paired document kept; the observed fact recorded in the dev document). Plus the story suite designed, closed, or left provisional pending a blocked unit.

## Instructions

- Designs validate outcomes, never implementation. A scenario asserting *how* the code works — internal path, timing, call sequence — leaves the implementer no freedom and yields brittle tests. The line is the same one the Behavioral Contract draws against implementation-internal mechanism.
- One paired document, one reader. Each paired design is implementable by TE without reading the Story or UX spec — those are QA's inputs, distilled in. An unrestated external reference is an incompleteness, not a TE lookup task. The suite's integration and contract scenarios are implemented by TE too, so restate what TE needs there as well; its E2E scenarios are QA's to author at verification.
- The suite is not a unit. Its scope is a composition or a journey, never a single subject; its ACs are precisely the ones the paired units could not verify alone. Designing the same behavior twice — once paired, once in the suite — means the paired disposition was wrong.
- QA observes a block, never diagnoses the contract. "No observable scenario fits this contract" — paired or at integration — is a fact within QA's domain; whether the contract is at fault is AR's judgment. Report `Design Blocked` and stop; do not redesign against a contract you cannot design to, and do not propose the fix.
- Task outcomes (per dev unit): `Designed` | `Design Blocked`