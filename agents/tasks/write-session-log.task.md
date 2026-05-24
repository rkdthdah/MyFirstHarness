---
id: write-session-log.task
level: 3
owner: or
templates:
- @~/{{ProjectRoot}}/agents/templates/agent/session-log.tmp.md
- @~/{{ProjectRoot}}/agents/templates/agent/latest-state.tmp.md
---

## Purpose

Record session activity and persist current state for cross-session continuity.

## Workflow

### 1. Append Session Log Entry

File: `~/{{ProjectRoot}}/agents/sessionlogs/{{agent-id}}/session-log_{{YYYYMMDD-HHMM}}.md`

If file does not exist → create per `session-log.tmp.md`

On every execution, append (HH:MM is current time):

```
### {{HH:MM}} — [{{tag}}]
{{1-2 sentence summary}}
```

**Tags:** decision | document | handoff | conversation | review | implementation | test

### 2. Overwrite Latest State

File: `~/{{ProjectRoot}}/agents/sessionlogs/{{agent-id}}/latest-state.md`

Overwrite entirely per `latest-state.tmp.md`
Max 30 lines. Only what the NEXT session needs

### 3. Self-Evaluation (Session End only)

Append evaluation block at the end of session log.
Review this session's activities against:

- Core Principles from persona
- Checklist items from executed tasks
- General Rules from AGENTS.md

**Checklist:**
- [ ] Reviewed against Core Principles
- [ ] Reviewed against executed task checklists
- [ ] Reviewed against AGENTS.md general rules
- [ ] Evaluation block appended to session log

## Instructions

- Use your own agent ID for file paths
- Session log: append-only, never edit past entries
- latest-state.md: always full overwrite, never append
- Keep entries concise
