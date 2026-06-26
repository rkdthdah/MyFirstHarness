---
id: development-principles
level: 1
owner: or
desc: The methodologies and cross-cutting values this project develops by — the canonical statement the harness implements
---
# Development Philosophy

The beliefs this project develops by. Each principle is stated once here as
the canonical source; the harness and specifications **implement** it, and
point back rather than restate. Two groups: **adopted methodologies** (named
practices the project commits to) and **cross-cutting values** (principles
that run through the whole system).

Each entry gives the principle, why it holds, and where it is implemented.

---

## A. Adopted Methodologies

### Test-Driven

Test design precedes implementation, and the gate on implementation is the
test **executing green** — never a document review.

- **Why:** A test written after the code tends to assert what the code does,
  not what the requirement demands. Designing the test first fixes the
  observable target before the body exists. A red test before implementation
  is the expected state, not a failure.
- **Implemented in:** status flow `test designed → implement` ahead of `dev`
  (harness §2); DE Development Gate = matching test set green (qa.spec); TE
  hands off red-but-runnable tests as normal.

### Domain-Driven

The Glossary is the project's Ubiquitous Language. Domain-model integrity —
registered kinds, Code IDs, aggregate boundaries — is a first-class concern,
not a downstream detail.

- **Why:** When every artifact names the same concept the same way, defects
  surface as language mismatches early. Drift in terms is drift in the model.
- **Implemented in:** Glossary as shared source of truth; AC spelling enforced
  against it (review-testability); Domain Terms carry Kind; AR is model
  authority, PM is the entry point for new terms.

### Make-or-Buy

When an established library can satisfy a requirement, prefer introducing it
over hand-building.

- **Why:** Re-creation is the silent default unless the system makes
  introduction a first-class option. Countering that bias is a standing choice,
  not a case-by-case afterthought.
- **Routing:** introduction is **AC-bound** (only to satisfy a specific AC —
  the AC is the user-agreed value, so it is the approval; no separate technical
  gate, since domain-expert users cannot meaningfully approve library choices);
  owned by the **spec authority** (AR for runtime/architecture, QA for test;
  agents without spec ownership signal, never introduce); **introduce-first**
  (settle before producing dependent artifacts; artifacts assume the introduced
  world); **bounce only on UI-representation change** (an introduction that
  changes what the prototype renders or how the user interacts returns upstream
  for rework; one that does not — a date library, a validator — proceeds in
  place, since nothing the user already approved is invalidated);
  **project-wide reach surfaces to PM** (an introduction creating capability
  beyond its AC is performed but noted for PM to weigh as a project-wide NFR).
- **Implemented in:** harness §16; spec ownership encodes the authority (no
  per-spec sentence needed — see harness §0.3).

### Shift-Left

Catch defects at the cheapest stage. When in doubt, bounce upstream.

- **Why:** One upstream round-trip costs less than one downstream redesign.
  Testability review and UI test are cheap gates before architecture pays its
  cost; architecture is the last cheap gate before implementation.
- **Implemented in:** `Revision Required` bias in review-testability and
  test-ui; architecture as the final pre-implementation gate (architect-story).

---

## B. Cross-Cutting Values

### Separation of Doer and Evaluator

Whoever produces an artifact does not evaluate it. Evaluation is meaningful
only when the evaluator did not predetermine the artifact's internals, and
only when the evaluator is a distinct actor with the context to judge.

- **Why:** A producer reviewing their own work cannot find what they could not
  see while making it. An evaluator who fixed the internals in advance has
  nothing left to review.
- **Manifestations:**
  - **Contract before body** — the contract (boundaries, signatures,
    observable behavior) is settled before implementation; the body is the
    implementer's. An evaluator who wrote the body cannot review it.
  - **Outcome over implementation** — verification and contracts assert
    externally observable terminal state, never internal path, timing, or call
    count. The same line separates a contract from its mechanism.
  - **Signal versus judgment** — an actor without the whole context observes
    and signals a fact, but does not diagnose. Judgment belongs to the actor
    who holds the context, who always re-evaluates — so a mis-signal never
    stalls the flow permanently.
  - **Fulfilled-world authoring** — an artifact is written assuming its
    prerequisites exist; a missing prerequisite is signalled to the workflow,
    not recorded in the body as "does not exist yet." The doer states the need
    as a signal; the workflow (which holds the context) resolves it before
    handoff, so the artifact a downstream reader receives is always
    self-consistent.
- **Implemented in:** AR designs contract / DE implements / AR reviews; QA
  designs test / TE implements / QA reviews; DE implements / QA verifies; TE
  and DE flag a needed redesign as an observation (the owner unchecks the
  relevant DoD item and reassigns); the owner then triages; infrastructure
  needs signalled to the workflow, resolved inline before handoff (harness §14).

### Single Source of Truth

Every fact, rule, and decision lives in exactly one place. Everywhere else
points to it — never copies.

- **Why:** A duplicated rule drifts; the copies disagree and no one knows which
  is authoritative. One place means one truth to change.
- **Implemented in:** harness §12 (a rule in exactly one place); ar.spec and
  qa.spec as domain sources of truth from which rules are extracted; a spec
  authority owning each introduction.

### Ownership Rotates, Never Shares

At any moment an artifact has one owner. Concerns are split by file, not by
regions of one file. Shared concerns hand off sequentially, never
concurrently.

- **Why:** Concurrent writers to one artifact race and conflict. Sequential
  ownership keeps boundaries physical (a file, a directory) rather than
  contested.
- **Implemented in:** harness §10; Story Owner rotation PM→UX→QA→AR; and a
  dev/test pair rotating through camps as a unit (harness §2 *Pair Camp*) — each
  camp's senior holds both documents to coordinate a cross-document redesign, so
  ownership stays single (one owner per document) and is never shared.

### Co-Design Boundary

Value decisions are made with the user; technical decisions are made by the
agents. Each is asked only what it can meaningfully answer.

- **Why:** A domain-expert user can confirm what the product should do, but
  cannot choose between libraries or architectures. Asking them a technical
  question is offloading a decision, not seeking consent — it adds friction
  without adding safety. The value is captured once, in the AC.
- **Implemented in:** harness §10 (co-design); PM requirements discovery and
  AC as the value-approval surface; Make-or-Buy's "the AC is the approval."

### Document versus Code

If something can be expressed in code, it belongs in code. Documents describe
only what code cannot express. Never maintain both a code artifact and a
document describing the same thing.

- **Why:** A document mirroring code is a second source of truth that rots.
- **Implemented in:** harness §6; cross-dev contracts materialized as
  code/schema files, not markdown (architect-story).

---

*Adopted methodologies are commitments; cross-cutting values are how the
harness is built to honor them. When the two appear to conflict in a specific
case, the cross-cutting value names the structural reason and usually resolves
it.*