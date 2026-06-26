---
id: architect-story.task
level: 3
owner: or
templates: @~/{{ProjectRoot}}/agents/templates/agent/dev.tmp.md
rules:
  - @~/{{ProjectRoot}}/frontend/ar.ar.rules.md
  - @~/{{ProjectRoot}}/backend/ar.ar.rules.md
---

## Purpose

Validate story coherence as the last cheap gate before implementation, then draft contract-level dev documents that delegate body to DE while preserving cross-dev coordination. Dev documents are read by both DE (for implementation) and QA (for test design); contracts must include behavioral specification, not just signatures.

## Workflow

### 1. Story Read

1. Read the Story — AC, Domain Terms, External Dependencies, Testability Results, UI Test Coverage
2. Read UX spec when referenced
3. Read Glossary entries for Story Domain Terms; existing architecture spec; prior external-dependency references
4. Read prior contracts established by previous stories (locations per ar rules) — existing types, registered routes, active providers, other AR-owned entry points
5. When the Story modifies existing behavior (Story Context cites current behavior or affected production code), read the affected production code paths (per ar rules) to understand current structure, behavior, and downstream dependents
6. Check for concurrent stories touching the same paths or external dependencies via `.worktrees/story/`. Coordinate via Notes when conflicts exist

**Checklist:**

- [ ] All Story sections read
- [ ] UX spec read if referenced
- [ ] Glossary entries for every Domain Term identified
- [ ] Prior contracts reviewed
- [ ] Affected production code reviewed (or N/A for greenfield)
- [ ] Concurrent-story conflicts identified or N/A

### 2. Story Coherence Review

1. Inspect along four axes in order; stop on first blocking defect since later axes assume earlier ones hold:
   - **Domain model integrity** — Glossary entries vs Story Domain Terms. Kind correctness (entity / value-object / role / state / command / domain-event). Code ID appropriateness. Aggregate boundary judgments not yet registered.
   - **AC implementation coherence** — Conflicts between AC requirements when implemented together. Hidden assumptions (implementation needs not stated in AC). Implementation feasibility (whether the AC can be realized in code at all).
   - **External dependency presence** — External Dependencies table consistent with AC needs. An AC requiring external data must have its source registered.
   - **Behavioral testability** — Each AC behavior can be specified in terms observable from outside the implementation (return values, state changes visible through the contract, exceptions raised). Behaviors expressible only through internal mechanisms (private cache state, race timing without observable effect) cannot be tested and must be reformulated.
2. Classify defects:
   - **Refinement** — Definition tightening, example addition, scope clarification with AC interpretation preserved. AR applies to Glossary directly; continue.
   - **Structural change** — Code ID change, Kind change, term split/merge, definition change affecting AC interpretation. Stop. Outcome: `Revision Required`.
   - **AC defect** — AC conflict, missing AC, feasibility gap, AC too coarse to decompose into reviewable units, behavior not externally observable, or breaking change to a contract relied on by an already-merged story without AC acknowledging the break. Stop. Outcome: `Revision Required`.

**Checklist:**

- [ ] Four axes inspected
- [ ] Each defect classified
- [ ] Refinements applied to Glossary
- [ ] All AC behaviors are externally observable

### 3. External Dependency Inspection and Digestion

Skip if Story External Dependencies table is empty or all rows mark `none`.

For each external dependency row:

- **Reference exists** — Apply cheap check: credentials still valid, target still reachable, sample data shape unchanged. On pass, reuse. On failure, request PM to provide an updated context doc. Stop. Outcome: `Revision Required`.
- **No reference yet** — Read the context doc, probe the external system through allowed cheap probes (per ar rules), and record a new section in the shared reference store. A reference records: source identity, access constraints, business meaning, freshness expectation, schema notes, last-verified timestamp.

If decomposition surfaces an external dependency not in the Story table, AR adds the row directly when it leaves every AC's interpretation unchanged and needs no new context doc; otherwise stop, outcome `Revision Required` (PM supplies the context doc / AC change).

**Checklist:**

- [ ] Each row processed (skip / cheap check / new reference)
- [ ] All cheap checks passed or escalated
- [ ] New references include last-verified timestamp
- [ ] Newly surfaced dependency added directly (AC-neutral, no context doc) or routed to PM

### 4. Dev Unit Decomposition

1. Make-or-buy (before decomposing — introductions are settled first, per ar rules):
   - For story scope that an established library can satisfy, evaluate introduction rather than hand-building. Introduce only to satisfy a `covers-acs` AC.
   - Introduction that changes prototype render/interaction requires prototype rework → stop, outcome `Revision Required` (reason: library introduced). Decompose only after UX reworks the prototype.
   - Introduction with no prototype change → decompose against the introduced world; record the introduction in the depending unit's Notes so it is settled before implementation. A capability beyond this Story's AC also carries a promotion-candidate note for PM.
2. Decompose story scope into dev units along judgment axes:
   - **Cohesion** — Related responsibilities grouped in one unit
   - **Minimal cross-dev coupling** — Units that would import each other heavily should usually be one
   - **Reviewable implementation scope** — A unit's implementation surface small enough for a single coherent review
3. For each unit, determine `covers-acs` — set of Story AC IDs the unit contributes to. Resolve per AC too: an AC realized across a handler, its destination screen, and wiring is covered by each of those units. Empty list denotes cross-cutting infrastructure.
4. For each unit, determine `test-kind` — signals the unit's verification profile to QA:
   - `business-logic` — DE-implemented runtime logic requiring functional verification.
   - `placeholder` — Type/constant declarations or thin pass-through with no new runtime logic.
   - `ar-integration` — AR-written wiring across AR-rw files (scaffolding, providers, routes, layout shells, cross-cutting infrastructure) that DE does not implement. Create when this Story modifies AR-rw files outside any business-logic unit.
5. Assign unit sequence numbers `dev-XXX-NN` in dependency order — a unit that exports symbols consumed by another gets the lower number. Ties broken at AR's discretion.

**Checklist:**

- [ ] Make-or-buy evaluated; prototype-changing introduction routed as `Revision Required`, prototype-neutral introduction recorded in unit Notes
- [ ] Every Story AC appears in at least one unit's `covers-acs`
- [ ] `test-kind` determined per unit, applied at unit granularity
- [ ] ar-integration unit created when AR-rw files outside business-logic units are modified
- [ ] Unit numbers assigned in dependency order (depended-on units get lower numbers)
- [ ] No unit grouped purely for size reasons against cohesion

### 5. Dev Document Drafting

For each dev unit, write the dev document per dev template (the persona workflow sets the filename owner and checks `Designed`). Compose:

- **Frontmatter** — `id`, `covers-acs` (from Step 4), `test-kind` (from Step 4); additional fields per dev template
- **Baseline** — `—`; architecture is initial decomposition, so there is no prior implementation
- **Design section** — Contract-level only:
  - Files this unit creates, modifies, or removes at boundaries — files other units import from, files at system entry points. A story that deletes existing functionality removes the corresponding boundary files
  - Public signatures and types of exported functions, classes, or modules
  - Schemas visible across units or to external systems
  - Behavioral contract — pre/post conditions, error and edge case behavior, observable state transitions, invariants. Scope: behaviors another unit relies on or AC requires. Implementation-internal mechanisms (private cache state, exact algorithm, internal timing) stay out even when externally observable in principle. **Each entry is self-contained — when an AC ID is referenced, include a one-line restatement of the AC's intent. No "see Story" / "per UX spec" external references.**
  - Position within architecture spec (ownership zone, dependency boundary)
- **Notes** — Cross-unit sequencing, foreseen edge cases that don't fit into behavioral contract, dependencies on other units in this story. Read by DE for implementation and QA for test design.

For `ar-integration` units, Design composition differs: list the AR-rw files modified, the dev unit exports this wiring composes, and the wiring surfaces QA verifies. No public signatures or Behavioral Contract — ar-integration exposes no contract to other dev units; its verification target is the surfaces listed.

Cross-dev contract artifacts — touchpoints where two units must agree on a shared shape. Heuristic: if a change on one side would break the other, it's a contract. Common kinds include cross-process APIs, in-process module interfaces, asynchronous message formats, and shared configuration; the heuristic governs new kinds. Materialize each contract as a code or machine-readable schema/contract file (not markdown) so DE can import or generate from it. Medium and location per ar rules.

**Forbidden in dev documents:**

- Function bodies, pseudocode, internal algorithms and control flow
- Private helper decisions (existence and shape)
- Internal state shape (component state, private fields)

Decision heuristic for any item in question: would changing this break another dev unit, or change behavior another unit relies on, or change behavior AC requires? If yes, it's contract — AR decides. If no, it's body — DE decides.

**Checklist:**

- [ ] Each dev unit has a dev document with complete frontmatter
- [ ] Design content is contract-level only — no bodies, pseudocode, private helpers, internal state
- [ ] Behavioral contract included for every business-logic unit
- [ ] ar-integration units list AR-rw files modified, dev exports composed, and surfaces to verify
- [ ] Cross-dev contracts materialized as code/schema files where applicable
- [ ] Notes included where cross-unit context exists
- [ ] Each unit's `covers-acs` reconciles with its body — every AC cited in Design/Behavioral Contract is listed, and every listed AC appears in the body
- [ ] Dev document is self-contained for DE — implementable without reading Story or UX spec

### 6. Placeholder / AR-Integration Implementation

For each `placeholder` and `ar-integration` unit (business-logic units are implemented by DE later, not here):

- **placeholder** — write the type/constant declarations or thin pass-through the unit specifies, in the files its Design lists, per ar rules.
- **ar-integration** — write the wiring across the AR-rw files its Design lists (scaffolding, providers, routes, layout shells), composing the dev exports it integrates, per ar rules.

These units carry no DE implementation phase; AR implements them here so they are ready for QA verification. Implement only what the unit's Design specifies — no behavior beyond it. If implementing surfaces a flaw in this unit's own design, correct the design and its document in place and re-implement — these units are AR's throughout, so there is no redesign cycle; the fix is part of this step. A flaw that breaks the decomposition itself (not just this unit) is the rare exception → outcome `Revision Required`.

**Checklist:**

- [ ] Every placeholder / ar-integration unit implemented in the files its Design lists
- [ ] No runtime behavior written beyond what the unit specifies
- [ ] A design flaw found while implementing is corrected in place and re-implemented (document updated)
- [ ] business-logic units left for DE (not implemented here)

> Structural model defect, AC defect, behavioral testability failure, external dependency currency failure, or a prototype-changing library introduction → outcome `Revision Required`
> All dev units drafted (with consistent Glossary, contracts, external references) and all placeholder / ar-integration units implemented → outcome `Completed`

## Instructions

- Architecture is the last cheap gate before implementation. Flaws caught here are absorbed in design; flaws caught later force rework. Phase 1 gates (testability, UI test) are not responsible for model or implementation coherence — do not retroactively blame them when defects surface here.
- Contract before body. Decisions omitted from contracts will be made implicitly by DE; that's incomplete architecture, not DE overreach. Conversely, decisions belonging to DE (body, private helpers, internal flow) must not appear in dev documents — pre-determining the body invalidates AR Review.
- Dev documents are implementable by DE without reading Story or UX spec — Story and UX spec are AR's inputs, and AR distills them into the dev document. QA reads Story and UX spec separately for test design. Internal mechanism descriptions serve no reader.
- Task outcomes: `Revision Required` | `Completed`