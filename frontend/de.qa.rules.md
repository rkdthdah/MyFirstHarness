---
id: de.qa.rules
level: 6
type: frontend-web-react
owner: qa
desc: Development-gate rules for DE agent
---

# DE Frontend Rules (Test Gate)

Owner: QA. Consumed by DE per AGENTS.md rules-matching protocol.

## Global Constraints

### Development Gate

The gate on a `dev-XXX-NN` is its paired `test-XXX-NN` running green against DE's implementation (per `qa.spec.md` *Development Gate Binding*). DE runs the gate; DE never edits it — every `*.test.ts(x)` file is read-only to DE.

Locate the paired test from the subject by co-location: `useFoo.ts` → `useFoo.test.ts`, `Foo.tsx` → `Foo.test.tsx`. A subject whose implementation file does not yet exist surfaces as an import-resolution failure before any test body runs — the *unexecuted* pre-implementation state, not a failure to fix.

## implement-dev.task

### Run & Classify

```bash
npm run test
```

Runs once (`--passWithNoTests`). The gate is reached only once the subject exists and the paired test set runs green. A test that cannot pass without a public surface, behavior, or dependency the contract does not grant is a contract block — surface it as the task outcome; never widen the body to force green.

**Checklist:**

- [ ] Paired test located from the subject by co-location
- [ ] `npm run test` run; gate green, or a contract block surfaced as the outcome
- [ ] No `*.test.ts(x)` file edited
