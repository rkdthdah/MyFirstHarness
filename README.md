---
id: README
level: 1
owner: or
---

# MyFirstHarness

*English · [한국어](README.ko.md)*

> A file-based multi-agent harness for software engineering. Six specialized agents
> collaborate under a strict protocol — no runtime, no orchestration server. The
> filesystem is the state machine, git is the audit trail.

---

## About

Stories are written, designed, tested, implemented, and shipped by separate agents,
each confined to specific directories and following a defined workflow. Every
artifact — story, spec, prototype, test design, session log — is a markdown file
with frontmatter declaring its owner and intended reader. Coordination happens
through file renames, directory listings, and handoff notes embedded in documents.

The protocol is described in **[`docs/harness.md`](docs/harness.md)** (15 sections,
~500 lines). This README is the entry door.

---

## Features

- **Filesystem as state machine.** Status is encoded in filenames
  (`dev-001-02_designed.md` → `_implement.md` → `_complete.md` → `_verified.md`).
  Each agent's work queue is computed by `ls` and a status filter. No queue
  service, no shared mutable state.
- **Five document types, one question each.** Persona (WHO and WHEN), Task (WHAT,
  technology-agnostic), Rules (WHERE and HOW, project-specific), Template (output
  shape), Index (lookup). Tasks and templates port across projects; rules swap per
  stack.
- **Seven document levels by communication direction.** Every file declares in
  frontmatter who wrote it for whom — `or→user`, `or→agent`, `agent→itself`,
  `agent→other agent`, etc. Reading any file tells you what it is allowed to assume.
- **Physical role separation.** Each agent has `rw` on specific paths and `r`
  everywhere else, enforced by an explicit File & Folder Access table in every
  persona. Boundaries are directory-level, not rhetorical.
- **DDD baked into the requirements step.** A Glossary holds the Ubiquitous
  Language; the PM registers every domain term before an AC may reference it; code
  identifiers must match the Glossary `Code ID` column.
- **TDD enforced by ordering.** QA reviews AC testability *before* AR architecture
  (cheapest gate). Test scenarios are designed by QA before TE writes test code;
  DE only implements against passing test gates.
- **Two-phase story lifecycle.** Phase 1 (Requirements) rotates a single Story
  document by Owner. Phase 2 (Implementation) freezes the Story and uses
  per-work-unit `dev/` and `test/` files whose filenames carry status. Two
  different orchestration patterns, both file-driven.
- **Spec → Rules extraction.** Domain authorities (AR for architecture, QA for
  testing) own single-source-of-truth specs. They extract per-consumer rules
  files. Agents read only their own rules — never specs, never other agents'
  rules. Tasks auto-match rules via `Global Constraints` + `## {task id}` →
  `### {step name}`.
- **Per-story git worktrees.** PM creates `.worktrees/story/STORY-XXX/` on story
  creation, removes it on completion. Parallel stories proceed without branch
  switching. Every commit carries `[STORY-XXX][agent][workflow]`.
- **Self-evaluation built in.** Every session log ends with a
  Persona / Task / Process compliance block plus improvement notes — a feedback
  signal embedded in the protocol itself.
- **Two-channel inter-agent communication.** Phase 1: Handoff Notes inside the
  Story document. Phase 2: `Notes for {target}` sections inside dev/test files.
  No third channel.
- **Co-design as a hard rule.** Stories and UX prototypes are built iteratively
  with the user; finalization without confirmation is forbidden.

---

## Directory structure

```text
MyFirstHarness/
├── README.md                 (· README.ko.md)
├── AGENTS.md                 ← agent activation rules + session startup scan
├── CLAUDE.md                 ← Claude Code entry point
├── agents/
│   ├── git-convention.md
│   ├── personas/             ← one file per agent (WHO and WHEN)
│   ├── tasks/                ← reusable, tech-agnostic task procedures (WHAT)
│   ├── templates/            ← output shapes (story, ux-spec, session-log, ...)
│   └── docs/
│       ├── glossary.md       ← Ubiquitous Language (DDD)
│       └── stories/          ← per-story folders; archived/ on completion
├── docs/
│   ├── harness.md            ← THE design document
│   ├── context/              ← user-supplied external dependency docs
│   └── frontend-*.md         ← example stack adaptation
├── frontend/                 ← example application workspace
└── backend/                  ← reserved
```

---

## Agents

```text
        Orchestration (or)
   ┌────┬────┬────┬────┬────┬────┐
   PM   UX   AR   QA   DE   TE
```

| Agent | Role            | Owns                                   |
| ----- | --------------- | -------------------------------------- |
| `pm`  | Product Manager | Stories, project artifacts             |
| `ux`  | UX Designer     | Prototype, UX specs                    |
| `ar`  | Architect       | Architecture spec, dev design          |
| `qa`  | QA Lead         | Test spec, test design, verification   |
| `de`  | Developer       | Implementation                         |
| `te`  | Tester          | Test implementation                    |
| `or`  | Orchestration   | Personas, tasks, rules, this harness   |

See [`AGENTS.md`](AGENTS.md) for activation rules and the session startup scan.

---

## Roadmap

> Naming: project **Stages** (below) are not the same as the **Phases** of the
> Story lifecycle defined in [`docs/harness.md`](docs/harness.md). Stage 1 contains
> both Phases of the harness.

### Stage 1 — End-to-end development process

Six agents complete a full shipping cycle, end-to-end, with every artifact
attributable to a specific agent and workflow step.

- **Phase 1 (Requirements: PM / UX / QA)** ✅ — personas, requirements-side tasks,
  shared templates, Glossary, git convention, and the harness design are in place.
- **Phase 2 (Implementation: AR / DE / TE + QA verification)** 🛠 — outstanding:
  AR/DE/TE personas; the `design-test` / `review-test` / `revise-test-design` /
  `verify-implementation` / `manage-qa-artifact` / `manage-qa-spec` task family;
  PM's `refine-story` / `complete-story`; and concrete frontend specs produced
  from the templates under `agents/templates/frontend-web-react/` together with
  the consumer rules files they extract into.

### Stage 2 — Orchestration layer

A standing `or` runtime that turns the markdown protocol into an actively-managed
system. Five concerns, drawn from current production-LLM-agent practice:

1. **Self-improvement via feedback loops.** Every session log already ends with a
   Persona / Task / Process compliance block plus improvement notes. The runtime
   consumes these as orchestration traces and proposes typed deltas to personas,
   tasks, and rules — the pattern [Reflexion][reflexion], [Multi-Agent
   Reflexion][mar], and [Instruction-Level Weight Shaping][ilws] converge on
   (reflect → store → condition next run; persona-based critics give richer
   signal than single-agent self-critique).

2. **Deterministic / non-deterministic separation.** The protocol already draws
   the line — workflow is decided, tasks are judgment work — but relies on the
   model to honor it. Stage 2 moves the deterministic side into code: the
   session-startup scan, filename status transitions, Handoff Note + Owner
   renaming, commit templating, DoD aggregation, the branch + worktree
   lifecycle, and rules-file matching. This matches the production consensus
   that [state transitions must be deterministic; the LLM is the planner /
   executor inside a distributed system][agent-arch].

3. **Token minimization.** Three non-negotiable guardrails — per-session token
   budget, per-loop iteration cap, termination evaluator — plus context
   compression via [anchored iterative summarization and provider-native
   compaction APIs][compaction]. Token inflation is nonlinear; each added tool
   and memory source multiplies cost.

4. **System integrity.** Strict tool contracts; OpenTelemetry trace spans for
   every LLM call, tool call, and handoff; evals running in CI against historical
   traces converted to regression datasets; declared stop conditions on every
   loop. Reliability is architectural, not prompt-engineered.

5. **Performance.** Hybrid model routing (cheap / local model as workhorse,
   frontier model as fallback for hard reasoning); sub-agent parallelism where
   the Phase 2 `dev/` and `test/` queues permit; prompt caching across persona
   boundaries; failure-driven prompt optimization using the self-evaluation
   blocks as the failure corpus.

### Stage 3 — Software development kit for non-developers

A user-facing surface — chat plus UI — that lets a person who knows nothing
about AI, the problem domain, or programming use this system as a software
development kit. The user expresses intent; the harness ships and maintains
the software.

The category is real and crowded: Gartner named AI-native development platforms
a top 2026 strategic trend, the segment is ~$4.7B with 63% non-developer users,
and [Replit Agent][replit] and [Lovable][lovable] each crossed $100M ARR in
under a year.

What this approach changes: those tools deliver code the non-expert must trust
blindly. This harness produces, *as a side effect of the protocol itself,*
the artifacts a non-expert can actually audit — human-readable stories with
acceptance criteria, a working prototype reviewed before any code is written,
a Glossary that fixes the language of the system, test designs that name what
must hold, and a per-commit git history they can roll back. The UI surface
translates intent into Stories on one side and surfaces these artifacts on the
other; the harness does the engineering.

Stage 3 is only credible because Stages 1 and 2 produce that audit substrate.

[reflexion]: https://arxiv.org/pdf/2505.24726
[mar]: https://arxiv.org/pdf/2512.20845
[ilws]: https://arxiv.org/pdf/2509.00251
[agent-arch]: https://andriifurmanets.com/blogs/ai-agents-2026-practical-architecture-tools-memory-evals-guardrails
[compaction]: https://currentstack.io/stories/agent-context-compression-gateway-pattern-2026/
[replit]: https://replit.com/discover/replit-vs-bolt
[lovable]: https://lovable.dev/guides/best-ai-app-builders

---

## Where to start

1. **[`docs/harness.md`](docs/harness.md)** — the 15-section design. If you read one file, read this one.
2. **[`AGENTS.md`](AGENTS.md)** — agent table, activation rules, session startup scan.
3. **[`agents/personas/pm.md`](agents/personas/pm.md)** — a concrete persona showing how Workflow / Tasks / Data / File Access fit together.
4. **[`agents/git-convention.md`](agents/git-convention.md)** — branch model, commit format, per-story worktree lifecycle.