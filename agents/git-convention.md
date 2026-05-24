---
id: git-convention
level: 3
owner: or
---

# Git Convention

## Repository

Single monorepo at project root.

## Branch Strategy

```text
main                            ← Release-ready code only
├── agents                      ← Agent configuration (personas, tasks, templates)
└── develop                     ← Story integration and verification
    ├── story/STORY-XXX
    └── story/STORY-YYY
```

- `main` — always stable, merge from develop only at release
- `agents` — orchestration agent edits personas, tasks, templates. Merge to develop only when no active story branches exist
- `develop` — integration branch, merge from story branches
- `story/STORY-XXX` — created from develop by PM, deleted after merge to develop

## Story Branch Lifecycle

1. PM creates `story/STORY-XXX` from `develop` when creating a story
2. PM creates worktree: `git worktree add .worktrees/story/STORY-XXX story/STORY-XXX`
3. All agents work in `.worktrees/story/STORY-XXX/` and commit to the story branch
4. AR passes merge readiness checks
5. PM completes the story, merges to `develop`, removes worktree and deletes the story branch:
6. At release → merge `develop` to `main`

### Agents Branch

1. Orchestration agent or user edits configuration files on `agents` branch
2. Worktree: `worktrees/agents`
2. When all story branches are merged (no active stories) → merge `agents` to `develop`

## Commit Convention

```text
[agent] summary
- detail (optional, as many as needed)
```

Story work includes story ID:
```text
[STORY-XXX][agent] summary
- detail (optional, as many as needed)
```

## Commit Timing

- Commit at each workflow step completion
- Commit before every Handoff (next agent must receive latest code)

## Protected Branches

- `main` — no direct commits, merge from develop only
- `agents` — orchestration agent or user only
- `develop` — no direct commits, merge from story branches only

## Ownership Enforcement

Each agent may only commit files within their rw directories.
Commit message must include agent ID for traceability.