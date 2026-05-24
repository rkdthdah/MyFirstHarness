---
id: qa.spec.tmp
level: 3
owner: or
type: frontend-web-react
---

<!--
  Frontmatter:
  ---
  id: qa.spec
  level: 5
  owner: qa
  type: frontend-web-react
  desc: {{one-line summary of test pyramid, tooling, conventions, coverage, and gate}}
  ---
-->

# Frontend Test Specification

Single source of truth for frontend testing.

{{Pointer to where test code ownership and directory boundaries live — typically `ar.spec.md` or equivalent architecture doc.}}

## Test Pyramid

| Level | Scope | Location | Owner |
| ----- | ----- | -------- | ----- |
| {{level name}} | {{what it verifies}} | {{glob path}} | {{agent}} |

**Authoring rule:**
- {{which agent authors which test levels directly}}
- {{which agent implements from another agent's test design documents, if any}}

## Tooling

| Concern | Tool |
| ------- | ---- |
| {{concern}} | {{tool name}} |

## Test File Convention

- {{co-location rule, e.g. `Foo.tsx` → `Foo.test.tsx`}}
- {{extension rule, e.g. `.tsx` when rendering React, `.ts` otherwise}}
- {{fixtures convention, e.g. sibling file vs `__fixtures__/` folder}}
- {{statement on how test files inherit dependency rules from the architecture spec}}

### Test infrastructure — `{{test-utils path}}`

```text
{{test infrastructure directory tree}}
```

{{Rule on mandatory use of the custom render helper and what direct framework imports are forbidden.}}

### Required file header

```tsx
// @story {{STORY-XXX}}
// @ac {{AC-1, AC-3}}
// @subject {{subject path relative to project root}}
// @level {{component | integration | unit | feature}}
```

{{Note on empty-value placeholder (e.g. `—`) and which fields may be empty for which test types.}}

## Test Scope per Type

### {{Test type 1, e.g. Component}}

Verifies: {{what this test type covers}}.
{{Forbidden: ... — only if there are hard exclusions.}}

### {{Test type 2}}

Verifies: {{what this test type covers}}.

{{Note on transitive coverage of any layer not directly tested — e.g. production pages covered via mirror-check, with pointer to the enforcing spec.}}

### {{Test type 3}}

Verifies: {{what this test type covers}}.
Forbidden: {{hard exclusions, e.g. no full component-tree rendering — use `renderHook` for hooks}}.

### {{Test type 4}}

Verifies: {{what this test type covers}}.
Forbidden: {{hard exclusions, e.g. no external dependencies — no network, DOM, or real timers}}.

## Coverage Policy

Mode: **{{report-only | gated | hybrid}}**. Metrics: {{which metrics — line, branch, function, statement}}. {{Note on when numeric gates are added.}}

Excluded paths:
- {{glob}}
- {{glob}}

## Test Design Document Lifecycle

{{Which test levels require a design document and which don't.}}
{{filename pattern, e.g. test-{STORY}-{NN}_{status}.md}}

- {{Pointer to fields spec, e.g. `harness.md` §2 *Implementation Documents*.}}
- {{Pairing rule between test documents and dev documents, including placeholder/skip behavior for dev work with no runtime logic to verify.}}
- {{Filename transition rule on status change.}}

## {{Cross-agent Gate Name, e.g. DE Development Gate}}

{{Owning agent}}'s gate on {{target agent}}'s work for a `{{dev document id pattern}}`:

{{Gate condition — e.g. all tests in the matching test set pass against the implementation.}}

## Runtime Commands

| Command | Purpose |
| ------- | ------- |
| `{{command}}` | {{purpose}} |