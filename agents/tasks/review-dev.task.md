---
id: review-dev.task
level: 3
owner: or
rules:
  - @~/{{ProjectRoot}}/frontend/ar.ar.rules.md
  - @~/{{ProjectRoot}}/backend/ar.ar.rules.md
---

## Purpose

Check a DE-implemented unit against its contract — that every contract element is realized, no decision the contract reserved was resolved differently in the body, no public surface or boundary was overstepped — then classify whether a finding originates in the code or the contract.

## Workflow

### 1. Intake

1. Read the dev document — Files, Public Signatures, Schemas, Behavioral Contract, Position, and the Baseline field.
2. Read the implementation code DE wrote.
3. Work from these two. The contract carries its own AC restatements; review compares code to contract, not to the AC. Note whether Baseline is a commit (a redesign — prior code may need reverting) or `—` (initial).

**Checklist:**

- [ ] Contract (Files / Signatures / Schemas / Behavioral Contract / Position) and Baseline read
- [ ] Implementation code read
- [ ] Baseline noted as a commit (redesign) or `—` (initial)

### 2. Conformance Review

Review on three axes. Each finding is a fact decided by comparing code to contract — cite the axis and location. Style, naming, and internal structure are the implementer's, never a finding.

- **Completeness** — every exported signature, schema, and Behavioral Contract clause is realized, including the contract's behavior for each `covers-acs`. A contract element with no implementation is a finding.
- **Contract fidelity** — the code exposes the contract's boundary and no more: a public surface the contract does not declare, a behavior diverging from a stated pre/post-condition or invariant, or a decision the contract reserved resolved differently in the body, is a finding. When Baseline is a commit, also diff it against the current tree: any change the prior implementation made that the current contract drops — an addition, edit, or deletion — left un-restored to its pre-change state is a finding, easy to miss as it predates the latest work.
- **Boundary validity** — the implementation wrote only within its ownership zone, obeys the dependency rules, and carries the required file headers, per rules. A body reaching outside its zone or importing across a forbidden boundary is a finding.

**Checklist:**

- [ ] Every signature / schema / contract clause realized, including each `covers-acs` behavior the contract carries
- [ ] No public surface or behavior beyond the contract
- [ ] No decision the contract reserved resolved in the body against the contract
- [ ] When Baseline is a commit, every prior change the current contract drops is restored to its pre-change state
- [ ] Ownership zone, dependency rules, and file headers valid per rules
- [ ] Each finding cites an axis and location

### 3. Origin Classification

No finding → `Pass`. For each finding, classify by one question — **does the code faithfully follow the contract?**

- **Code diverges from the contract** (a contract element unrealized, a surface or behavior the contract does not carry, a boundary overstepped, or a prior change left un-restored) → the code is at fault → `Rework`.

- **Code faithfully follows the contract, and the contract carries the defect** (the contract specified a signature that cannot realize the AC, an infeasible pre/post-condition, a boundary that forces a forbidden dependency) → the contract is at fault → `Redesign`.

Mixed findings: any `Redesign`-class finding makes the outcome `Redesign` — a contract defect must be resolved before re-implementation against it is meaningful. Record the code-class findings in the same note so they survive the redesign.

**Checklist:**

- [ ] Each finding classified `Rework` (code) or `Redesign` (contract) by the fidelity question
- [ ] Outcome is `Pass` (no finding), `Rework` (all findings code-class), or `Redesign` (any contract-class finding)
- [ ] For `Redesign`, code-class findings recorded alongside

> No finding → `Pass`
> Findings, all originating in the code → `Rework`
> Any finding originating in the contract → `Redesign`

## Instructions

- Review compares code to contract, never code to the AC — the contract is the standard, settled at architecture. Re-reading the AC here re-opens design judgment that belongs to architect-story and redesign.
- AR validates contract realization, not implementation craft. A body's structure, naming, and private helpers are DE's; a finding must name a conformance fact, not a preference.
- The fidelity question is the whole of origin classification: a defect the code introduced is DE's to fix; a defect the code faithfully inherited from the contract is AR's. Sending an inherited defect back to DE only loops — the code has nothing to change.
- Outcome only — DoD check-off, owner rename, and the Handoff Note are the persona's; this task ends at the classified outcome.
- Task outcomes: `Pass` | `Rework` | `Redesign`
