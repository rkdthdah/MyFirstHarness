---
id: harness
level: 1
owner: or
desc: Principles for designing multi-agent harness systems
---
# Harness Engineering Principles

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

### Persona (`agents/personas/{agent}.md`)

WHO the agent is and WHEN it acts

Contains:

- Agent metadata (name, ID, title, model)
- Customization (project context, domain)
- Persona (role, style, identity, focus)
- Core Principles
- Workflow (WHEN and in WHAT ORDER — not HOW)
- Handoff and DoD check-off procedures at each workflow step
- Tasks table (file references)
- Data (files to read at session startup)
- File & Folder Access (explicit r/rw per path)

Does NOT contain:

- HOW to execute tasks
- Technology-specific rules
- Other agents' details

### Task (`agents/tasks/{name}.task.md`)

HOW the agent performs a specific job

Contains:

- Purpose (one sentence)
- Workflow (steps → actions → checklists → routing)
- Instructions (rules and constraints)

Does NOT contain:

- Persona or workflow scheduling
- Template structure
- Technology-specific paths, commands, or conventions
- Handoff Notes, DoD check-off, or Owner rotation (belongs in persona workflow)

### Template (`agents/templates/{name}.tmp.md`)

WHAT the output document looks like

Contains:

- Structure only (sections, tables, placeholders)
- Minimal guidance comments

Does NOT contain:

- Procedural instructions
- Technology-specific paths or examples
- Downstream consumer guidance

### Specification (`{project-area}/{agent}.spec.md`)

Project-specific source of truth from which rules are extracted.

Contains:

- Project-wide policies, standards, and conventions for a domain
- Tool and technology decisions
- Structural boundaries and dependency rules
- Coverage criteria and quality gates

Does NOT contain:

- Task procedures (belongs in task)
- Agent-specific extracted rules (belongs in rules)

Owner: Domain authority agent (AR for architecture, QA for test)
Agents do not read specifications directly — they read extracted rules files

### Rules (`{project-area}/{consumer}.{owner}.rules.md`)

Project-specific concretization of abstract task instructions

Structure:

- Two top-level section types:
  - `## Global Constraints` — applies to every task that links this rules file
  - `## {task id}` — applies only when the matching task is executing; contains `### {step name}` sub-sections matched against the current task step
- Step sub-sections contain project-specific paths, commands, conventions, and a checklist verifying project rules compliance
- Matching protocol is defined in AGENTS.md

Contains:

- Cross-task conventions (file access, dependency rules, file headers, import conventions, etc.) under Global Constraints
- Step-scoped asset locations, commands, and checklists under `## {task id}` → `### {step name}`

Does NOT contain:

- Workflow procedures (belongs in task)
- Agent identity or scheduling (belongs in persona)
- Git or commit conventions (belongs in persona workflow)

Owner: domain authority agent — AR for architecture, QA for test. The `{owner}` segment in the filename matches.
Consumer: the agent that reads and follows the file. The `{consumer}` segment in the filename matches. A single consumer may have multiple rules files when multiple owners govern its work (e.g. DE reads both `de.ar.rules.md` and `de.qa.rules.md`)

### Index (`{project-area}/{name}.idx.md`)

Lookup table for selective file loading

Contains:

- Mapping table (story → files, or asset → metadata)
- Minimal header describing purpose

Updated by the agent that creates/modifies the tracked assets

### Story (`agents/docs/stories/STORY-XXX/STORY-XXX_{Owner}.md`)

Lifecycle document for a single story. Tracks requirements, phase progress, and handoffs

Contains:

- AC definitions (source of truth)
- DoD checkboxes covering both Phase 1 and Phase 2 milestones
- Phase 1 outcome records (Testability Results, UI Test Coverage)
- Handoff Notes
- Links to UX spec and AC documents within the same Story folder

Does NOT contain:

- Phase 2 implementation details (live in dev / test documents within the same Story folder)
- Phase 2 progress tracking (derived from dev/test filenames; see *Implementation Documents* below)

Owner: rotates with current phase actor. Created by PM. During Phase 2 the Owner field remains `AR` (sleeping) until integration

### Implementation Documents (`agents/docs/stories/STORY-XXX/{dev,test}/`)

Per-work-unit documents produced during Phase 2.

Filename convention:

```
{type}-{STORY}-{NN}_{status}.md
```

- `type`: `dev` | `test`
- `STORY`: 3-digit zero-padded story id (e.g. `001`)
- `NN`: 2-digit zero-padded sequence within the Story (`01`, `02`, ...)
- `status`: `designed` | `implement` | `complete` | `verified` | `redesign-request` | `rework`

Status flow:

```
designed → implement → complete → verified     (dev only; complete = AR review pass, verified = QA verification pass)
              ↓
       redesign-request → designed              (implementer → designer: design flaw found during impl)

implement | complete → rework → implement      (reviewer → implementer: code flaw found during review)
```

Filename is renamed on every status transition.

Dev documents declare AC coverage via frontmatter `covers-acs: [<AC IDs>]` (e.g. `[1, 3, B3]`); empty list means cross-cutting infrastructure. AR ensures every Story AC is covered by at least one dev document. Test documents inherit AC mapping from their paired dev.

Phase 2 progress is read directly from the directory listing (`ls dev/`, `ls test/`); no separate progress document is maintained. Aggregating status produces the Story DoD `Development` mark — all dev/test files reaching `complete` flips Story DoD.

Document body accumulates over the lifecycle:

- Each status author writes their own section (design, implementation notes, etc.)
- When transitioning status, the author appends a `Notes for {target}` section per the table below
- The receiving agent reads these sections as the Phase 2 handoff. No Story Handoff Notes are written during Phase 2

| Transition                  | Author | Notes for | Purpose                       |
| --------------------------- | ------ | --------- | ----------------------------- |
| → `_designed` (test)        | QA     | TE        | Implementation guidance       |
| → `_implement` (dev)        | DE     | QA        | Surfaces for verification     |
| → `_implement` (test)       | TE     | QA        | Surfaces for review           |
| → `_complete` (dev)         | AR     | QA        | Review pass, ready to verify  |
| → `_verified` (dev)         | QA     | AR        | Verification pass             |
| → `_redesign-request` (dev) | DE     | AR        | Design flaw found during impl |
| → `_redesign-request` (test)| TE     | QA        | Design flaw found during impl |
| → `_rework` (dev)           | QA     | DE        | Verification findings         |
| → `_rework` (test)          | QA     | TE        | Review findings               |

Owner: rotates by status — the actor whose action triggers the next status owns the rename.

### Story Phases

Stories progress through two phases with different document orchestration:

- **Phase 1 — Requirements:** Story document is the unit of work. PM → UX → QA workflow rotates through Story Owner. QA hands off directly to AR at Phase 1 exit (no PM round-trip)
- **Phase 2 — Implementation:** dev / test documents within the Story folder are the units of work. AR generates dev documents at Phase 2 entry; QA generates test documents per qa.spec and dev documents. Story Owner stays `AR` (sleeping) and Story document is read-only until integration.
- **Phase boundary:** AR architecture step (Phase 2 entry) and AR integration step (Phase 2 exit) are the only Phase 2 touchpoints on the Story document itself.

---

## 3. Document Separation Principle

```text
Persona  → WHEN to do it
Task     → WHAT to do (technology-agnostic)
Rules    → WHERE and HOW to verify (technology-specific)
Template → WHAT the output looks like (technology-agnostic)
Index    → WHERE to find things (project-specific data)
```

Task never references rules sections directly — agents auto-match via the protocol defined in AGENTS.md (Global Constraints + `## {task id}` → `### {step name}`).
Task references Template: "write per template"
Task never contains technology-specific paths, commands, or conventions.
Step-scoped sections without a matching task step are ignored. Global Constraints always apply when the rules file is linked. Missing rules file → proceed without.

This separation enables:

- Tasks and templates reusable across projects
- Rules swappable per project/tech stack
- Indexes grow with project without changing tasks

---

## 4. Document Level System

Documents are assigned levels based on their communication direction:

| Level | Direction             | Examples                                                |
| ----- | --------------------- | ------------------------------------------------------- |
| 1     | or → user            | harness, README                                         |
| 2     | or → or              | orchestration internal docs                             |
| 3     | or → agent           | AGENTS.md, personas, templates, tasks, git-convention |
| 4     | agent → user         | project artifacts                                       |
| 5     | agent → itself       | session-logs, latest-state, specification               |
| 6     | agent → other agent  | UX specs                                                |
| 7     | multi-agent ownership | stories                                                 |

---

## 5. Document Frontmatter

Every document includes frontmatter for identification:

```
---
id: {{unique identifier}}
level: {{1-7}}
owner: {{agent-id}}
desc: {{one-line summary}} ← optional, when purpose is not obvious from id
---
```

Templates may define additional or different frontmatter fields for generated documents like:

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

## 6. Separation of Concerns

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

- If expressible in code → it belongs in code
- Documents describe what code CANNOT express
- Never maintain both a code artifact and a document describing the same thing

---

## 7. Inter-Agent Communication

### Handoff Notes

- During Phase 1, agents communicate through Handoff Notes in Story documents
  - Format: `[DateTime] FROM → TO: note`
  - Handoff includes renaming Story Owner to the receiving agent
- During Phase 2, agents communicate through `Notes for {target}` sections in Implementation Documents (see §2 *Implementation Documents*)
  - Story document is read-only
  - Handoff is signaled by filename status rename
- No direct agent-to-agent communication outside the two channels above

### Shared Documents

- Glossary — shared domain terms, all agents read
- Stories — lifecycle documents, owner rotates
- Indexes — lookup tables, updated by asset creators
- External Context — user-provided docs in `docs/context/` for external dependencies; written by PM during story creation, read by AR on first integration, archived on story completion

### Delegation Documents

- During Phase 2, all inter-agent handoffs occur via Implementation Documents (dev / test)
- File name encodes current status (per Implementation Documents convention in §2)
- Body contains dedicated handoff sections (e.g. "Notes for DE", "Notes for QA")
- Agent scan logic finds files matching `{type}-*_{status}.md` to determine work queue

---

## 8. Token Optimization

### Index-First Access

- Maintain index files as lookup tables
- Agents read the index first, then load only related files
- Never scan entire directories when an index exists

### Rules-First Convention

- Agents read their rules file for project-specific paths and commands
- Tasks remain technology-agnostic and compact
- Rules change per project; tasks do not

### Template Minimalism

- Templates contain structure only
- Usage guidance belongs in the task
- Consumer guidance belongs in the consumer's own task

### Catalog Documents

- When agents cannot access a UI, maintain a text catalog
- Catalogs list available assets with enough detail for reuse decisions
- Updated as part of post-creation actions

---

## 9. Checklist Design

### Types

- **Analysis** — verify understanding before starting work
- **Reuse** — verify existing assets were considered
- **Co-design** — verify user confirmed the output
- **Validation** — verify completeness and correctness (task-level)
- **Rules validation** — verify project rules compliance (rules-level)
- **Post-completion** — verify all artifacts saved and linked

### Placement

- Every workflow step must have a checklist — in task always, and in the matching `## {task id} → ### {step name}` rules sub-section if present
- Task-level checklists: "WHAT was done" (AC coverage, user confirmation)
- Rules-level checklists: "project rules were followed" (file locations, dependencies, conventions, commits)
- Consolidation checklists (e.g. total validation step) should be distributed into their respective steps
- Minimize overlap between task and rules checklists

### Routing

- On fail → specify exact step to return to
- On pass → specify what to update (DoD, Owner, etc.)

---

## 10. Co-Design Principle

- Never finalize without user confirmation
- Build iteratively WITH the user
- Show working artifacts during co-design, not descriptions
- Collect feedback → iterate → confirm → proceed

---

## 11. Ownership & Boundaries

### File-Level Ownership

- Every file/directory has one owner with write access
- Other agents read only
- Ownership changes require explicit approval

### Boundary Enforcement

- Boundaries are physical (directory-level), not logical
- Define allowed and forbidden dependencies
- When two agents share a concern, split by file — not by "areas of the same file"
- Exception: lifecycle documents (Story, dev/test) rotate ownership through phases. File owner is the *current* phase actor — sequential, not concurrent

### Rules as Boundary Contract

- Domain authority agents define boundaries in specification documents (AR: ar.spec, QA: qa.spec)
- Domain authority agents extract agent-specific boundaries into per-agent rules files
- Each agent reads only its own rules file — not the central document

---

## 12. Deduplication

- A rule appears in exactly ONE place
- Tasks reference rules files — never duplicate their content
- If multiple tasks share an instruction, extract to persona or rules
- "Per rules `Section Name`" = follow rules defined there — do not restate

---

## 13. Git Convention

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
- PM creates worktree when creating a story, removes on story completion

### Branch Lifecycle

- Story: PM creates branch and worktree from develop → agents work in worktree → PM merges to develop, removes worktree and deletes branch
 - Agents: or edits configuration → merge to develop when no active story branches
 - Release: develop → main

### Commit Convention

```text
[agent][workflow name] summary
- detail (optional, as many as needed)
```

Story work includes story ID and may include status extras:

```text
[STORY-XXX][agent][workflow name][optional extras] summary
- detail (optional, as many as needed)
```

`optional extras`: status modifiers like `Discarded`, `Open Questions`, `Major Rework`, `Minor Revision`, `dev-NN`, `pass`, `fail`, etc

- Handoff first, then commit in the story worktree
- Commit at each workflow step completion

### Ownership Enforcement

- Each agent commits only within rw directories
- Agent ID in commit message for traceability

---

## 14. Session Lifecycle

### Startup

1. Load all files in Data section
2. Check for interrupted work → resume first
3. Scan `.worktrees/story/*/` for queued work:
   - Phase 1 — Story documents with matching Owner
   - Phase 2 — Implementation Documents with matching status (see §2)
4. Process queued in order found
5. Nothing queued → wait for input

### End

1. Write session log (with self-evaluation)
2. Update latest-state

---

## 15. Orchestration Agent

### Responsibilities

- Manage agent configuration files (personas, tasks, templates)
- Review session logs for improvement opportunities
- Refine agent definitions based on feedback
- Maintain harness-level conventions (this document, git-convention, AGENTS.md)

### Configuration Change Policy

- Changes merge to develop only when no active story branches exist
- All changes tracked on the `agents` branch
- Changes are based on session log self-evaluations and observed patterns

### Does NOT

- Execute story-level work
- Communicate with users directly during story execution
- Override agent decisions within their defined boundaries
