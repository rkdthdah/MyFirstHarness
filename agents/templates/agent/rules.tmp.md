---
id: rules.tmp
level: 3
owner: or
type: frontend-web-react
---

<!--
  Frontmatter:
  ---
  id: {{consumer}}.{{rule-owner}}.rules
  level: 6
  type: frontend-web-react
  owner: {{rule-owner agent}}
  desc: {{one-line summary, e.g. "UI test rules for QA agent" or "Frontend rules for UX agent"}}
  ---

  Filename convention: {consumer}.{rule-owner}.rules.md
  - consumer == rule-owner: agent writes rules for itself (e.g. qa.qa, de.de)
  - consumer != rule-owner: one agent writes rules for another (e.g. ux.ar, de.qa, te.qa)
-->

# {{Consumer agent}} {{Domain}} Rules

Owner: {{rule-owner}}. Consumed by {{consumer}} per AGENTS.md rules-matching protocol.

## Global Constraints

### File Access

| Permission | Path | Description |
| ---------- | ---- | ----------- |
| {{r|rw}}   | `{{glob path}}` | {{one-line purpose}} |

{{Closing statement on default for unlisted paths, e.g. "All other frontend paths are read-only." Optionally point to the parent spec's *Ownership Boundaries* section for the canonical read-path map.}}

### Dependency Rules

{{One of two patterns:
  (a) one-line cross-ref to the parent spec's *Forbidden Dependencies* section, when this rule file adds no new constraints;
  (b) explicit Allowed / Forbidden code blocks below, when this rule file narrows the spec's rules for the consumer's scope.}}

```text
{{layer → allowed/forbidden dependencies, if applicable}}
```

### {{Add more sections as needed}}

### {{File header section name — "File Header" or "Required File Header"}}

```tsx
// @{{tag1}} {{value-pattern}}
// @{{tag2}} {{value-pattern}}
```

{{Note on empty-value placeholder (e.g. `—`, not `none`/`N/A`/blank), which tags may be empty for which cases, and any per-tag ID conventions (e.g. AC numbering, story IDs).}}

### {{Optional domain-specific subsection — e.g. "Test File Convention", "Component Rules", "Import Convention", "i18n Dictionary Slices", "Storybook Manager Boundary", "Test Infrastructure"}}

{{One-line operational rule, ideally referencing the parent spec for the rationale (e.g. "Per {{spec}} *{{section}}*, ...").}}

- {{rule}}

{{Add more domain-specific subsections as needed. Each should be a short, operational rule — not a re-explanation of spec material. If the spec already covers the rationale, reference it rather than repeat.}}

## {{task-id, e.g. test-ui.task, create-ux-spec}}

{{One-line task purpose, if not obvious from the task-id.}}

### {{Task step 1, e.g. "Coverage Mapping", "Story Analysis"}}

{{Step description — typically a decision table mapping inputs to outputs (test level, asset type, target location), or an analysis checklist.}}

| {{column}} | {{column}} |
| ---------- | ---------- |
| {{cell}}   | {{cell}}   |

**Checklist:**

- [ ] {{verifiable item}}

### {{Task step 2, e.g. "Test Implementation", "Prototype Build & Co-Design"}}

{{Step description with tooling tables, location tables, or implementation rules.}}

| {{column}} | {{column}} |
| ---------- | ---------- |
| {{cell}}   | {{cell}}   |

- {{implementation rule}}

### {{Task step 3, e.g. "Run & Verify", "UX Spec & Validation"}}

```bash
{{command}}
{{command}}
```

**Checklist:**

- [ ] {{verifiable item}}

### {{Optional task step — e.g. "Post-Creation"}}

{{Steps to update catalogs / indexes (typically `*.idx.md` files) after task completion. Specify per-index update format inline.}}

**Checklist:**

- [ ] {{verifiable item}}