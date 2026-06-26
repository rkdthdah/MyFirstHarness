---
id: test.tmp
level: 3
owner: or
type: agent
---

<!--
  Frontmatter (paired test, NN ≥ 01):
  ---
  id: test-XXX-NN
  level: 7
  owner: {{QA | TE | AR | Complete}}
  parent: STORY-XXX
  covers-acs: [{{AC ID list — inherited from paired dev, full set retained}}]
  test-kind: {{business-logic | placeholder | ar-integration — matches paired dev}}
  desc: {{one-line summary of what this design verifies}}
  ---

  Story suite (NN = 00, one per story): covers-acs = the story's deferred ACs;
  omit test-kind. Verifies integration / contract / E2E across units — see the task.
-->

# [test-XXX-NN]

{{One-line description of what this design verifies}}

## Definition of Done

<!-- business-logic runs all three. placeholder / ar-integration, and a closing or
     E2E-only suite: Designed only, Implemented / Reviewed [N/A]. -->

- [ ] Designed — QA
- [ ] Implemented — TE
- [ ] Reviewed — QA

## Baseline

<!-- Redesign only: the commit holding this unit's prior test implementation. TE diffs
     the current design against it to delete test files the design no longer needs; QA
     review checks the deletion is complete. `—` when there is no prior implementation. -->

- {{commit hash / —}}

## Subject

<!-- Paired: the file / export under test, from the paired dev document's Files and
     Public Signatures.
     Story suite (00): not a single subject — the units composed and the boundaries
     where they meet, the contracts both sides honor, and the journeys verified end
     to end. -->

- {{file path :: export under test  /  units · boundaries · contracts · journeys}}

## Scenarios

<!-- One entry per scenario. given/when/then with an observable terminal state.
     Assertion code, exact mocks, timing, internal paths, and unmandated call
     counts are forbidden — assert what the unit produces, not how.
     Story suite (00): append a level tag to each scenario — (integration | contract | e2e). -->

### S1 — {{short scenario name}}

- **Given** {{precondition / entry state}}
- **When** {{trigger}}
- **Then** {{observable terminal state}}

## AC Coverage

<!-- Every covers-acs ID gets a row and a disposition. scenario IDs link to the
     Scenarios section; many-to-many is expected (one scenario, several ACs).
     defer-to-verify rows are the unit-level input QA collects at verification.
     Story suite (00): every AC is scenario-covered — these ARE the deferred ACs
     the paired tests punted here. -->

| AC   | Disposition                                                 | Scenarios | Reason                          |
| ---- | ----------------------------------------------------------- | --------- | ------------------------------- |
| {{ID}} | {{scenario / covered-by-ui-test / defer-to-verify / N/A}} | {{S1, … / —}} | {{required for non-scenario rows}} |

## Fixtures & Mock Strategy

<!-- Paired: what the scenarios require, at outcome level — what must be present or
     stubbed for the terminal state to be observable.
     Story suite (00): real or stubbed boundaries, seeded state, environment —
     integration / contract exercise real boundaries with fixtures; e2e runs the full
     stack. Concrete tooling and locations are per rules, not named here. -->

- {{fixture / mock need}}

## Notes

<!-- Cross-unit sequencing, foreseen edge cases not captured as scenarios,
     dependencies on other units. Restate any referenced content — the implementer
     does not read the Story, UX spec, or other dev units. -->

- {{free-form note, list, or prose}}

## Handoff Notes

<!-- Each owner appends a note when passing the document to the next agent.
     Format: [DateTime] FROM → TO: note -->


<!-- Empty section: replace bullet content with `—` (em-dash). -->