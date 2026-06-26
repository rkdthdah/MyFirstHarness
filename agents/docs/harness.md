---
id: harness
level: 1
owner: or
desc: Principles for designing multi-agent harness systems
---
# Harness Engineering Principles

This document and `development-principles` are the two canonical sources for
the whole system. Everything else — personas, tasks, rules, specs, templates —
is regenerable from these two: harness defines the **document system and
operating structure**, development-principles defines the **methodologies and
values**. Where a rule is methodological it lives there and is pointed to here;
where it is structural it lives here.

---

## 0. Document System Charter

The rules that govern how every other document is written, where each piece of
content belongs, and how the documents reference each other. This section is the
authority for those decisions; the per-type detail in §2 concretizes it.

### 0.1 What each document holds

The single test for placing any piece of content: **would it read identically
under a different technology stack?** Stack-invariant content (a concept, a
role split, a methodology, a gate principle) is universal and belongs in a
canonical document; stack-bound content (a path, a tool, a directory, a
convention) belongs in the stack document; procedure belongs in the task.

| Document | Holds | Does NOT hold |
| --- | --- | --- |
| **harness** (L1, human) | document-system rules, operating structure (file lifecycle, status, owner rotation, workflow skeleton) | methodology (→ principles); stack specifics (→ spec); procedure (→ task) |
| **development-principles** (L1, human) | adopted methodologies and cross-cutting values — the stack-invariant *why* | structure (→ harness); stack specifics (→ spec) |
| **persona** (L3) | WHEN and in what order an agent acts; owner transition, handoff, DoD check-off, rename at each step | HOW a job is done (→ task); stack specifics (→ rules) |
| **task** (L3) | HOW a job is done — procedure, steps, routing — technology-agnostic | WHEN/scheduling (→ persona); owner/handoff/DoD/rename (→ persona); stack specifics (→ rules) |
| **template** (L3) | WHAT an output document looks like — structure only | procedure (→ task); stack specifics (→ rules) |
| **spec** (L5, `type:` declared, owner) | stack-bound bindings only — paths, tools, structure tables, conventions | universal concepts/roles/gates (→ principles); procedure (→ task); extracted per-agent rules (→ rules) |
| **rules** (L6) | per-task extraction of spec bindings; project paths, commands, checklists | anything without a basis in spec (violation); procedure (→ task); identity/scheduling (→ persona) |

Two consequences worth stating because they are easy to violate:

- A concept that would appear in a backend spec verbatim (e.g. a test-pyramid
  level, a doer/evaluator role split, a "green gate" principle) is **not** a
  frontend-spec fact even though it shows up in frontend work. It is universal
  → principles. The spec carries only the binding that realizes it (the path,
  the tool).
- Duplicating a universal rule into a spec to "make it visible to the agent" is
  worse than referencing it — the agent never reads the spec directly anyway
  (it reads extracted rules), and the copy drifts. The binding the agent needs
  is already in the spec's structure (paths, owner); the universal rule stays
  canonical.

### 0.2 Reference direction (single, hard)

```text
workflow → task → rules → spec        (lower never references higher)
harness ↔ development-principles       (mutual; both human-canonical)
```

- Nothing — not spec, rules, task, persona — references **harness** or
  **development-principles**. They are human-facing canon; agents operate from
  the documents those two generate, not from the canon itself.
- **spec** does not reference harness either (same reason). rules reference
  spec (extraction relation); the reverse never holds.
- A task names no rules section directly — agents auto-match the current step
  name to the rules section via the AGENTS.md protocol (Global Constraints
  always apply; `## {task id}` → `### {step name}` applies when the step is
  running; an unmatched step-section is ignored; a missing rules file means
  proceed without).
- A task references a template only as "write per template," and contains no
  stack-specific path, command, or convention.

### 0.3 Authority is encoded in ownership

A document's `owner` frontmatter plus its Ownership Boundaries table *are* the
authority statement — no prose needs to restate it. "The runtime/architecture
spec is owned by AR" follows from `ar.spec` declaring `owner: ar`; "the test
spec is owned by QA" from `qa.spec`. So a universal rule that delegates by
authority (e.g. *introduction is owned by the spec authority*) is stated once
in principles and **needs no per-spec sentence** — each spec satisfies it just
by declaring its owner.

### 0.4 Single source of truth

A rule lives in exactly one place; everywhere else points to it (see
development-principles → *Single Source of Truth*, harness §12). In particular,
the global file regime — filename convention, status lifecycle, owner rotation
(§2 *Implementation Documents*) — lives only here. An agent learns its own
transition from the specific status its persona workflow names, never by
copying the regime into a spec or rules file.

---

## 1. Agent Hierarchy

```text
Orchestration Agent (or)
├── manages: personas, tasks, templates, conventions
├── improves: reviews session logs, refines agent configuration
└── delegates to:
    ├── PM  — WHAT and WHY
    ├── UX  — WHAT USER SEES
    ├── QA  — OUTCOME VALIDATION
    ├── AR  — HOW (architecture, structure, rules)
    ├── DE  — HOW (implementation)
    └── TE  — HOW (test implementation)
```

- Orchestration agent owns agent configuration files and improves them based on session log feedback
- Working agents own story-level artifacts and execute within their defined boundaries

---

## 2. Document Types

Each type's placement rules are in §0; this section gives the file path, the
type-specific structure, and the lifecycle detail that §0 does not.

### Persona (`agents/personas/{agent}.md`)

Agent metadata, customization, persona (role/style/identity/focus), Core
Principles, Workflow, Tasks table, Data (startup reads), File & Folder Access
(explicit r/rw per path). Per §0.1, the Workflow carries owner transition,
handoff, and DoD check-off at each step.

### Task (`agents/tasks/{name}.task.md`)

Purpose (one sentence), Workflow (steps → actions → checklists → routing),
Instructions.

### Template (`agents/templates/{name}.tmp.md`)

Structure only — sections, tables, placeholders, minimal guidance comments.

### Specification (`{project-area}/{agent}.spec.md`)

Project-wide stack bindings for a domain: tool and technology decisions,
structural boundaries and dependency rules, directory/convention tables,
coverage criteria and quality-gate bindings. Owner: domain authority agent (AR
for architecture, QA for test). Agents never read specs directly — they read
extracted rules.

### Rules (`{project-area}/{consumer}.{owner}.rules.md`)

Per-agent extraction of spec bindings. Two top-level section types:

- `## Global Constraints` — applies to every task that links this rules file
  (cross-task conventions: file access, dependency rules, file headers, import
  conventions)
- `## {task id}` — applies only while the matching task runs; contains
  `### {step name}` sub-sections with step-scoped asset locations, commands,
  conventions, and a checklist verifying project-rule compliance

`{owner}` in the filename matches the spec authority; `{consumer}` matches the
reading agent. One consumer may have several rules files when multiple owners
govern its work (e.g. DE reads both `de.ar.rules.md` and `de.qa.rules.md`).

### Index (`{project-area}/{name}.idx.md`)

Lookup table for selective loading: a mapping table (story → files, or asset →
metadata) and a minimal header. Updated by the agent that creates/modifies the
tracked assets.

### Story (`agents/docs/stories/STORY-XXX/STORY-XXX_{Owner}.md`)

Lifecycle document for one story: AC definitions (source of truth), DoD
checkboxes for Phase 1 and Phase 2 milestones, Phase 1 outcome records
(Testability Results, UI Test Coverage), Handoff Notes, links to UX spec and
implementation documents in the same folder. Phase 2 implementation detail and
progress live in dev/test documents, not here. Owner rotates with the current
phase actor; created by PM; during Phase 2 the Owner field stays `AR`
(sleeping) until integration.

### Implementation Documents (`agents/docs/stories/STORY-XXX/{dev,test}/`)

Per-work-unit documents produced during Phase 2. Each unit is a **pair** — one
`dev` document (contract + implementation) and one `test` document (its
verification design), matched by sequence number `dev-XXX-NN` ↔ `test-XXX-NN`.
They route like the Story document (§2 *Story*): the filename carries the
**owner**, progress is tracked by **DoD checkboxes inside the document**, handoff
is an owner rename plus a Handoff Note, and there is no status in the filename.

Filename convention:

```
{type}-{STORY}-{NN}_{Owner}.md
```

- `type`: `dev` | `test`
- `STORY`: 3-digit zero-padded story id (e.g. `001`)
- `NN`: 2-digit zero-padded sequence within the Story (`01`, `02`, ...)
- `Owner`: the agent who acts next — `AR` | `DE` | `QA` | `TE` | `Complete`. The Session Startup scan queues a document to whoever its filename names; `Owner` is not a write-permission gate (permissions come from persona File Access and the spec)

Frontmatter carries `owner` (matching the filename), `covers-acs`, and
`test-kind`; it carries no status field.

Definition of Done — progress is the set of checked DoD items, each owned by one agent:

```
dev:   - [ ] Designed — AR        - [ ] Implemented — DE        - [ ] Reviewed — AR        - [ ] Verified — QA
test:  - [ ] Designed — QA        - [ ] Implemented — TE        - [ ] Reviewed — QA
```

`placeholder` and `ar-integration` units (set by AR at decomposition, see below)
have no implementation or review phase: their `Implemented` and `Reviewed`
items are marked `[N/A]` at creation. A `dev` placeholder/ar-integration still
takes `Verified` — QA confirms the unit truly has no runtime behaviour and no
AC is left uncovered. A `test` placeholder/ar-integration is created with only
`Designed` checked (recording why no unit-level design exists) and the rest
`[N/A]`; it has no scenarios to implement, review, or verify.

#### Pair Camp

A pair is produced and evaluated by opposite role lines (dev: AR→DE→AR→QA;
test: QA→TE→QA), so at any moment it belongs to one **camp** holding the
authority to move and redesign both documents:

- **AR camp** — senior `AR`, junior `DE`. AR designs/reviews the contract; DE
  implements the body.
- **QA camp** — senior `QA`, junior `TE`. QA designs/reviews the test; TE
  implements the test code.

Two invariants keep ownership single (development-principles → *Ownership
Rotates, Never Shares*) while letting a cross-document redesign happen without
any agent editing a document it does not own:

- **Invariant 1 — Pair Camp.** dev and test are always in the same camp.
- **Invariant 2 — Transition Gate.** A camp transition (QA↔AR) occurs only when
  **both** documents are held by that camp's senior (both `QA`, or both `AR`).
  While a junior (`DE`/`TE`) holds either, the pair cannot change camp — so a
  cross-document redesign is always performed by one senior owning both.

Routing — camp + which DoD items are checked. Each row is one senior's action; juniors implement and hand back up.

**AR camp** (dev: AR↔DE, test: AR-held):

| Doc | Owner | DoD state | Action → result |
| --- | --- | --- | --- |
| dev | AR | none | architect → check `Designed`; pair leaves for QA camp (test authored there). placeholder/ar-integration: `Implemented`/`Reviewed` `[N/A]`, pair goes to QA camp for verify |
| dev | DE | Designed | implement → check `Implemented`, owner → AR; cannot implement (contract gap, or contract met but paired test unpassable) → uncheck `Designed`, owner → AR, note which |
| dev | AR | Implemented | review → check `Reviewed`, owner → AR (pass); code flaw → uncheck `Implemented`, owner → DE |
| dev | AR | Reviewed | both docs AR-senior → transition pair to QA camp for verification |
| test | AR | (held) | held while DE implements, and after verify until integration; see *Redesign* if needed |
| dev,test | AR | Verified (all story pairs) | integrate (merge-hook + E2E review; no re-run) → dev `Complete`, test → QA (W8 closes); merge-hook fail → re-arch; E2E unsound → re-verify |

**QA camp** (dev: QA-held, test: QA↔TE):

| Doc | Owner | DoD state | Action → result |
| --- | --- | --- | --- |
| test | QA | none (business-logic) | design → check `Designed`, owner → TE |
| test | QA | none (placeholder/ar-integration) | design → closing doc, `Designed` checked, rest `[N/A]`; pair stays QA-held and goes straight to verify (no implement/review, no AR camp) |
| test | TE | Designed | implement → check `Implemented`, owner → QA; design unimplementable → uncheck `Designed`, owner → QA |
| test | QA | Implemented | review → check `Reviewed`, owner → QA (pass); code flaw → uncheck `Implemented`, owner → TE; design flaw → uncheck `Designed`, owner → QA (redesign) |
| dev | QA | Designed | held while the pair's test runs design→implement→review |
| test | QA | Reviewed | both docs QA-senior → transition pair to AR camp: route dev → DE (implement), test → AR (hold) |
| dev,test | QA | (verifying) | verify integration → check dev `Verified`, both → AR (pass); finding → both → AR (redesign wf) |

**Redesign.** A redesign is any change to a unit's **code** that is not a review
rework — a changed signature/behavior, or a changed file name/path. Editing a
document's Notes or Handoff is not a redesign. Review and verification steps do
not classify a redesign; they signal "cannot proceed here" and route to a
redesign workflow, where the judgment lives:

- **QA's test-design workflow** — initial design, or revision after a **QA Test Review** finding — is the test-side judge: design within QA (contract intact), or route the pair to AR's **redesign** workflow when the contract blocks the design (*Design Blocked*).
- **AR Dev Review** (design-origin finding), a **DE block** (DE cannot implement the contract), **QA Verification**, and **AR Integration** → AR's **redesign** workflow.

**AR's redesign workflow is the single judge of a dev-side redesign**, deciding per pair:

- **dev redesign** (contract changed) → dev re-`Designed`; paired test fully unchecked.
- **test-only redesign** (contract intact) → test fully unchecked; dev's DoD untouched, but its owner moves to `QA` with the test.
- **re-architecture** (impact spreads beyond the pair, or the decomposition is unsound) → §12 reset.

Either non-re-architecture outcome returns the pair to QA camp **together** (dev
→ `QA` held, test → `QA`); normal camp flow carries it from there. Reset touches
only **active** DoD items; `[N/A]` items stay `[N/A]`. At Integration a failing
pair is still `_AR` (not yet `Complete`), so AR reactivates it normally — dev
`Designed` unchecked → `AR`, paired test → `AR`. AR decomposition prefers
write-disjoint units and evaluates impact, so spread stays rare.

**rework** (uncheck `Implemented`, owner → junior) occurs **only** at the two
review steps — AR Review (dev code flaw → DE), QA Test Review (test code flaw →
TE). After review, a fault is a design fault routed to a redesign workflow.
Missed review is detected by the OR from session logs (§15).

#### Story Suite (`test-XXX-00`)

The Pair Camp model above governs **pairs**. A story also carries one **suite**
test, `test-XXX-00`, with **no paired dev** — a story-level instrument for what
no single unit verifies: the integration and contract scenarios across units, and
the end-to-end journeys. It sits outside the camp model:

- **QA-owned throughout** — it never rotates through an AR camp (no dev body to
  implement or review). DoD is `Designed — QA` / `Implemented — TE` /
  `Reviewed — QA`, with **no `Verified`** — the suite is verification's
  instrument, not its subject.
- **`covers-acs`** is the story's deferred ACs — the facets paired tests
  dispositioned `defer-to-verify`, collected at design, not inherited.
- **Lifecycle by level** — `integration` / `contract` scenarios are QA-designed,
  TE-implemented, QA-reviewed, first executed at **Verification**; `e2e`
  scenarios are designed as journeys only, then authored and run by QA at
  Verification with their code reviewed by AR at **Integration**. A suite with no
  deferred AC, or only `e2e` scenarios, is `Designed` with `Implemented` /
  `Reviewed` `[N/A]`, owner QA.
- **Two touchpoints** — at Verification the suite's scenarios run and the pairs'
  `Verified` is the result; at Integration AR reviews the E2E code and gates the
  merge, never re-running it. On an Integration pass the suite renames to QA to
  close (W8); on a Verification fail it stays QA for the returning pairs.

Dev documents declare AC coverage via frontmatter `covers-acs: [<AC IDs>]`
(e.g. `[1, 3, B3]`); empty list means cross-cutting infrastructure. AR ensures
every Story AC is covered by at least one dev document. Test documents inherit
AC mapping from their paired dev.

Phase 2 progress is read directly from the directory listing (`ls dev/`,
`ls test/`) — the filename owner tells the scan who acts next; the document's
DoD tells how far it has progressed. The Story DoD `Development` mark flips at Integration when
every dev document is `Complete` and every test document is owned by `QA` (for
Workflow 8 to close) or already `Complete`.

Document body and handoff accumulate over the lifecycle exactly as the Story
does: each owner writes its own section (design, implementation notes,
verification result), and on every owner change appends a Handoff Note
(`[DateTime] FROM → TO: note`) — the same channel and format as Phase 1, no
separate `Notes for` mechanism. The receiving agent reads the Handoff Notes as
the Phase 2 handoff.

`test-kind` (`business-logic` | `placeholder` | `ar-integration`) is set by AR
at decomposition and recorded in frontmatter. `business-logic` runs the full
DoD; `placeholder` and `ar-integration` are created with `Implemented`/`Reviewed`
`[N/A]` as above. The `ar-integration` dev variant is authored and owned by AR
through `Designed` (AR is both designer and implementer of the wiring) and then
follows the normal `Verified` path.

### Story Phases

- **Phase 1 — Requirements:** Story document is the unit. PM → UX → QA rotates through Story Owner. QA hands off directly to AR at Phase 1 exit (no PM round-trip).
- **Phase 2 — Implementation:** dev / test documents are the units. AR generates dev documents at Phase 2 entry; QA generates test documents per qa.spec and the dev documents. Story Owner stays `AR` (sleeping) and the Story document is read-only until integration.
- **Phase boundary:** AR architecture step (Phase 2 entry) and AR integration step (Phase 2 exit) are the only Phase 2 touchpoints on the Story document.

---

## 3. Document Level System

Documents are assigned levels by communication direction:

| Level | Direction             | Examples                                                |
| ----- | --------------------- | ------------------------------------------------------- |
| 1     | or → user            | harness, development-principles, README                 |
| 2     | or → or              | orchestration internal docs                             |
| 3     | or → agent           | AGENTS.md, personas, templates, tasks, git-convention |
| 4     | agent → user         | project artifacts                                       |
| 5     | agent → itself       | session-logs, latest-state, specification               |
| 6     | agent → other agent  | UX specs, rules                                         |
| 7     | multi-agent ownership | stories                                                 |

---

## 4. Document Frontmatter

Every document includes frontmatter for identification:

```
---
id: {{unique identifier}}
level: {{1-7}}
owner: {{agent-id}}
type: {{stack identifier}}   ← spec/rules only, e.g. frontend-web-react
desc: {{one-line summary}}   ← optional, when purpose is not obvious from id
---
```

Templates may define additional or different frontmatter fields for generated
documents:

```
<!--
  Frontmatter:
  ---
  id: STORY-XXX
  level: 4
  owner: PM
  desc: {{one-line summary}} ← optional
  {{additional fields as needed}}
  ---
-->
```

---

## 5. Separation of Concerns

### Role Boundaries

- PM defines WHAT and WHY — never HOW
- UX defines WHAT USER SEES — never how it's built
- AR defines HOW — architecture, interfaces, structure, rules
- AR composes implementation completion gates from QA, build, and structural checks
- QA validates OUTCOMES — not implementation
- QA owns test specification and extracts test rules
- DE implements HOW — within AR's architecture
- TE implements test code — within QA's test design
- OR refines agent configuration — based on feedback

### Document vs Code

See development-principles → *Document versus Code*. If expressible in code, it
belongs in code; documents describe only what code cannot; never maintain both
a code artifact and a document describing the same thing.

---

## 6. Inter-Agent Communication

### Handoff Notes

- During Phase 1, agents communicate through Handoff Notes in Story documents
  - Format: `[DateTime] FROM → TO: note`
  - Handoff includes renaming Story Owner to the receiving agent
- During Phase 2, agents communicate through Handoff Notes in the dev/test documents (§2), in the same format
  - The Story document is read-only; handoff is signaled by renaming the dev/test document owner and appending a Handoff Note
- No direct agent-to-agent communication outside these two channels

### Shared Documents

- Glossary — shared domain terms, all agents read
- Stories — lifecycle documents, owner rotates
- Indexes — lookup tables, updated by asset creators
- External Context — user-provided docs in `docs/context/` for external dependencies; written by PM during story creation, read by AR on first integration, archived on story completion
- External Dependency References — `agents/docs/external-refs.md`, one section per external system, recording what AR digested from a context doc and probe (source identity, access constraints, business meaning, freshness expectation, schema notes, last-verified). Stack-neutral and shared like the Glossary: written and updated by the AR that first integrates or re-verifies a system, read by any later AR. Accumulates across stories — not archived with the story that created it (distinct from External Context, which is the transient PM input that AR digests into a reference)

---

## 7. Token Optimization

### Index-First Access

- Maintain index files as lookup tables; read the index first, then load only related files; never scan whole directories when an index exists

### Rules-First Convention

- Agents read their rules file for project-specific paths and commands; tasks stay technology-agnostic and compact; rules change per project, tasks do not

### Template Minimalism

- Templates contain structure only; usage guidance belongs in the task; consumer guidance belongs in the consumer's own task

### Catalog Documents

- When agents cannot access a UI, maintain a text catalog listing assets with enough detail for reuse decisions; updated as a post-creation action

---

## 8. Checklist Design

### Types

- **Analysis** — verify understanding before starting
- **Reuse** — verify existing assets were considered
- **Co-design** — verify user confirmed the output
- **Validation** — verify completeness and correctness (task-level)
- **Rules validation** — verify project-rule compliance (rules-level)
- **Post-completion** — verify all artifacts saved and linked

### Placement

- Every workflow step has a checklist — in the task always, and in the matching `## {task id} → ### {step name}` rules sub-section if present
- Task-level: "WHAT was done" (AC coverage, user confirmation). Rules-level: "project rules followed" (locations, dependencies, conventions, commits)
- Distribute consolidation checklists into their respective steps; minimize task/rules overlap

### Routing

- On fail → specify the exact step to return to. On pass → specify what to update (DoD, Owner, etc.)

---

## 9. Co-Design Principle

Implements *Co-Design Boundary* (development-principles). Never finalize without
user confirmation; build iteratively WITH the user; show working artifacts, not
descriptions; collect feedback → iterate → confirm → proceed. Value decisions
are taken with the user, technical decisions by the agents — each is asked only
what it can meaningfully answer.

---

## 10. Ownership & Boundaries

Implements *Ownership Rotates, Never Shares* (development-principles).

### File-Level Ownership

- Every file/directory has one owner with write access; others read only; ownership changes require explicit approval

### Boundary Enforcement

- Boundaries are physical (directory-level), not logical; define allowed and forbidden dependencies
- When two agents share a concern, split by file — not by "areas of the same file"
- Exception: lifecycle documents (Story, dev/test) rotate ownership through phases. The owner is the *current* phase actor — sequential, not concurrent

### Rules as Boundary Contract

- Domain authority agents define boundaries in specs (AR: ar.spec, QA: qa.spec) and extract per-agent boundaries into rules files; each agent reads only its own rules file

---

## 11. Deduplication

Implements *Single Source of Truth* (development-principles).

- A rule appears in exactly ONE place; tasks reference rules files, never duplicate them
- If multiple tasks share an instruction, extract to persona or rules
- "Per rules `Section Name`" = follow rules defined there — do not restate

---

## 12. Git Convention

### Branch Model

```text
main                            ← Release-ready
├── agents                      ← Agent configuration (or-managed)
└── develop                     ← Story integration
    └── story/STORY-XXX         ← Per-story work
```

### Worktrees

```text
.worktrees/
├── agents/                     ← agents branch worktree
└── story/
    ├── STORY-XXX/              ← story/STORY-XXX branch worktree
    └── STORY-YYY/              ← story/STORY-YYY branch worktree
```

- Agents work in `.worktrees/story/STORY-XXX/` instead of switching branches
- PM creates the worktree when creating a story, removes it on completion

### Branch Lifecycle

- Story: PM creates branch and worktree from develop → agents work in worktree → PM merges to develop, removes worktree and deletes branch
- Agents: or edits configuration → merge to develop when no active story branches
- Release: develop → main

### Redesign Rollback

Phase 2 redesign (§2) resets documents; this is the code rollback that
accompanies it.

- **Per-unit redesign** rolls back by **re-implementation, not git reset.** The
  reset documents (`Designed`-only) send DE/TE to implement afresh against the
  new design: re-writing a kept file replaces the old body, and a file the new
  design drops is deleted by its owner. A unit's own files are disjoint from
  other units', so this disturbs no one else.
- **Re-architecture** — when a redesign's impact spreads beyond its own pair to
  other units, the decomposition itself is reworked. Reset the story branch to
  the **Phase 2 entry** — the commit where the Story has just become AR-owned
  and no dev document yet exists (distinct from the post-architect state where
  dev documents are drafted). Order matters:
  1. `git reset --hard {Phase 2 entry}` — discards all dev/test documents and all DE/TE code.
  2. **Then** append a Handoff Note to the Story recording why the previous decomposition failed (a note written before the reset would be discarded by it; it survives only on top of the reset point).
  3. architect-story re-enters (its `no dev files` trigger now holds) and reads the note in Story Read, so the new decomposition does not repeat the prior mistake.

### Commit Convention

```text
[agent][workflow name] summary
- detail (optional, as many as needed)
```

Story work includes the story ID and may include status extras:

```text
[STORY-XXX][agent][workflow name][optional extras] summary
- detail (optional, as many as needed)
```

`optional extras`: status modifiers like `Discarded`, `Open Questions`, `Major Rework`, `Minor Revision`, `dev-NN`, `pass`, `fail`.

- Handoff first, then commit in the story worktree; commit at each workflow step completion
- An infrastructure workflow commits only its own spec/infrastructure change; the calling workflow commits its documents separately — the two never share a commit

### Ownership Enforcement

- Each agent commits only within rw directories; agent ID in the commit message for traceability

---

## 13. Session Lifecycle

### Startup

1. Load all files in Data section
2. Check for interrupted work → resume first
3. Scan `.worktrees/story/*/` for queued work:
   - Phase 1 — Story documents with matching Owner
   - Phase 2 — Implementation Documents with matching status (§2)
4. Process queued in order found
5. Nothing queued → wait for input

### End

1. Write session log (with self-evaluation)
2. Update latest-state

### Resumability

- Work leaves a stable (typecheck/depcheck-passing) state at every stop, never a half-applied edit
- A task interrupted mid-way is resumed by the next session, not restarted; partial work in a stable state is preserved
- Workflows that re-evaluate (e.g. spec-gap triage) re-judge the world each pass — not once — so a condition unmet now is picked up when later met

---

## 14. Workflow Execution

How a workflow invokes another workflow, and how infrastructure work is reached.

### Inline versus terminal invocation

- **Inline** — caller invokes a sub-workflow to resolve a need, then continues ("blocked → means → return"). Both caller and sub-workflow log. The infrastructure calls (→ §15) are inline.
- **Terminal** — caller invokes a sub-workflow as a non-returning phase transition ("completion → next phase"). Only the sub-workflow logs; the caller hands off and ends. AR W5→W6 is the terminal call.
- **Commit separation is absolute** (§12): a sub-workflow commits only its own change; the caller commits its own documents separately.

### Infrastructure: definition and reach

- **Infrastructure is any change that requires a spec-file change.** Work inside a spec's already-allowed scope (a contract edit, a decomposition, composing an entry-point the spec already grants) is **not** infrastructure — the task performs it directly. A change that alters the spec itself (a new dependency, a tool, a directory rule, a test-util the structure does not yet describe) is infrastructure → it routes to the spec authority's infrastructure workflow (AR: W7 / QA: W9).
- **Baseline is an init precondition; these workflows handle increments only.** That a spec exists means baseline infrastructure was set up at init (a separate OR-driven line, → §15). W7/W9 add to that baseline; they do not create it.
- An infrastructure workflow is invoked **inline** by a story workflow that discovers the need, **or** runs **standalone** on user/OR request. Inline, it commits to the story worktree and the caller writes the session log; standalone, it commits to the agents worktree and writes its own log.

### Fulfilled-world authoring

Implements *Separation of Doer and Evaluator → Signal versus judgment*
(development-principles). An artifact that needs an infrastructure increment is
written **assuming the increment exists** (the fulfilled world). The need is
raised as a signal from the task to its workflow — never recorded as an unmet
prerequisite in the document body — and the workflow resolves it via the inline
infrastructure call before handing off. The document therefore never tells a
downstream reader "this does not exist yet"; by the time it is handed off, it
does.

---

## 15. Orchestration Agent

### Responsibilities

- Manage agent configuration files (personas, tasks, templates)
- Review session logs for improvement opportunities; refine agent definitions based on feedback
- Maintain harness-level conventions (this document, development-principles, git-convention, AGENTS.md)
- Run the init line: collect environment parameters (closed/open network, stack, scale, user count) through user dialogue, hand them to QA/AR, who create baseline spec + infrastructure through a dedicated init task — distinct from W7/W9, which handle increments only
- Detect cross-session failure patterns (e.g. a unit repeatedly bounced back for redesign) and raise a standalone infrastructure workflow

### Configuration Change Policy

- Changes merge to develop only when no active story branches exist
- All changes tracked on the `agents` branch; driven by session-log self-evaluations and observed patterns

### Does NOT

- Execute story-level work
- Override agent decisions within their boundaries
- (User communication is the OR's channel — working agents minimize it; see development-principles → *Co-Design Boundary*)

---

## 16. Make-or-Buy

Implements *Make-or-Buy* (development-principles). When a library can satisfy a
requirement, prefer introducing it over hand-building. Routing within the
harness:

- Introduction is a **spec change** (§14), owned by the spec authority — AR for runtime/architecture (ar.spec), QA for test (qa.spec). Authority is encoded in spec ownership (§0.3); agents without it signal the need and never introduce
- Bound to a specific AC — the AC is the user-agreed value, so it is the approval; no separate technical gate
- Settle the library before producing dependent artifacts; artifacts assume the introduced world and carry no unmet-prerequisite note (§14 fulfilled-world)
- If the introduction changes UI representation (component render/interaction), the work bounces upstream — AR introduces it (W7), then returns the story via PM for UX to rework the prototype; an introduction that does not change UI (e.g. a date library) proceeds in place
- An introduction reaching beyond its AC (project-wide capability) is performed but noted in Handoff Notes for PM to weigh as a project-wide NFR at story completion