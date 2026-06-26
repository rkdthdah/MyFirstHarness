---
id: implement-dev.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/de.ar.rules.md
  - @~/{{ProjectRoot}}/backend/de.ar.rules.md
  - @~/{{ProjectRoot}}/frontend/de.qa.rules.md
  - @~/{{ProjectRoot}}/backend/de.qa.rules.md
---

## Purpose

Implement an AR-designed dev contract as working code, faithfully and within its ownership zone, until the paired test set runs green — the development gate. The contract is the standard; the body, its private helpers and internal structure, is DE's. Publish the unit's boundary files first so its paired test resolves the subject, then fill the body against the contract.

## Workflow

### 1. Contract Intake

1. Read the dev document — Files and their Action, Public Signatures, Schemas, Behavioral Contract, Position (ownership zone, allowed imports), and the Baseline field.
2. Read the paired test set — the green gate this implementation must reach — and the cross-dev contract artifacts this unit imports or generates from.
3. Work from the contract, this task, the consumed rules, and the source the contract names. The contract is authored to stand alone — do not read the Story or UX spec to fill a gap; a gap that blocks implementation is a contract flaw, surfaced in Step 3, never patched from outside.

**Checklist:**

- [ ] Files / Signatures / Schemas / Behavioral Contract / Position / Baseline read
- [ ] Paired test set (the gate) and imported contract artifacts read
- [ ] No unrestated Story / UX-spec reference required to implement

### 2. Implementation

1. If Baseline is a commit (a redesign), first restore: diff it against the current contract and undo every change the prior implementation made that the current contract no longer carries, returning each affected file to its pre-change state — a prior addition removed, an edit reverted, a deletion restored. A boundary file the contract now marks `remove` is deleted. Baseline `—` (initial design) skips this. Work from the referenced commit and git history, not memory.
2. Publish the unit's boundary files — those its design lists `create` / `modify`, with their exported signatures and types — so the paired test resolves its subject and stops being *unexecuted*; then implement the body against the contract. A unit handed back already implemented (a test-only revalidation) has its files present already — confirm them rather than re-create.
3. Implement only within this unit's ownership zone, per rules. The contract's signatures, schemas, and Behavioral Contract are honored exactly; private helpers, internal structure, and control flow are DE's to choose. Never widen the public surface beyond the contract.
4. Author against the fulfilled world — assume designed fixtures, introduced libraries, and depended-on units exist. Never write "not yet implemented" into the code; an unmet prerequisite is a Step 3 signal, not body content.

**Checklist:**

- [ ] When Baseline is a commit, every prior change the current contract drops is undone; files marked `remove` deleted
- [ ] Boundary files published before the body (or confirmed present on a revalidation)
- [ ] Body within the ownership zone; public surface exactly the contract's
- [ ] Signatures, schemas, and Behavioral Contract honored; private structure DE's own
- [ ] No unmet-prerequisite note written into the code

### 3. Run & Classify

1. Run the paired test set per rules — the development gate.
2. Classify by what is observable, never by inferred cause:
   - **Paired test set green** → the contract is realized and verified at the unit boundary → `Implemented`. Own body errors are DE's to fix and are not a handoff state — iterate until the gate is green or the block is the contract's.
   - **The gate cannot be reached within the contract** — the body fulfills the contract but the paired test still fails, or passing it would require a public surface, behavior, or dependency the contract does not grant → `Contract Blocked`, stating the observed fact only.

**Checklist:**

- [ ] Paired test set executed per rules
- [ ] Own body defects self-corrected (no unresolved author error)
- [ ] Result classified to exactly one outcome
- [ ] `Contract Blocked` carries an observed fact, not a diagnosed cause

> Paired test set runs green → outcome `Implemented`
> The contract cannot be met, or cannot be met without exceeding it → outcome `Contract Blocked`

## Instructions

- Faithful to the contract — contract quality is settled at AR Review and redesign, not here. Realizing it as written, with a body of DE's own design, is the contribution.
- The body is DE's, the boundary is AR's. Private helpers, internal structure, and control flow are chosen freely; the public surface, signatures, and observable behavior are the contract's and are never widened or narrowed.
- The gate is green, not a document. DE implements until the paired test set passes against the subject; a red-but-pending subject is where the unit started, not where it is handed off. A unit returned for a test-only revalidation is re-run against its revised gate and patched within the contract to reach green.
- Author the fulfilled world. A missing prerequisite is signalled as the outcome, never recorded in the code as "does not exist yet."
- A stable stop is a green gate or a stated block, never a half-applied edit.
- Observe, never diagnose. DE reports the gate is unreachable within the contract; whether the contract is at fault is AR's judgment at redesign.
- Outcome only — DoD check-off, owner rename, and the Handoff Note are the persona's; this task ends at the classified outcome.
- Task outcomes: `Implemented` | `Contract Blocked`
