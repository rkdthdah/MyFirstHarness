---
id: complete-story.task
level: 3
owner: or
---

## Purpose

Archive a delivered story — read it to find what travels with it, then move the story folder and its referenced context documents out of the active trees into their archives, so the workspace holds only live work. The completion verdict (the `Delivery` check-off, the owner rename to `Complete`), the completion commit, and the branch merge and teardown are the persona's, around this step. The mirror of create-story: that opens a story's workspace, this files it away.

## Workflow

### 1. Intake

1. Read the full Story — confirm its `Development` DoD is checked (the precondition for completion), and read its External Dependencies for any context document that archives with the story.
2. Locate what the story occupies in the active trees: its story folder, and each referenced context document.

**Checklist:**

- [ ] Story read; `Development` DoD confirmed checked
- [ ] Referenced context documents identified from External Dependencies (or none)

### 2. Archive

Move each out of the active trees, preserving history (a move, never a delete):

1. The story folder: `agents/docs/stories/STORY-XXX/` → `agents/docs/stories/archived/STORY-XXX/`.
2. Each referenced context document: `docs/context/{file}` → `docs/context/archived/{file}` (per the External Dependencies rows that set a context document).

**Checklist:**

- [ ] Story folder moved to the stories archive
- [ ] Every referenced context document moved to the context archive
- [ ] The active stories and context trees hold nothing of this story

> Story and its context documents archived → outcome `Completed`

## Instructions

- Filing only — the `Delivery` check-off, the owner rename to `Complete`, the completion commit, and the merge-and-teardown of the story branch are the persona's, settled around this step.
- Archive is a move, never a delete — the story and its context survive in the archive; the Story remains the lifecycle record.
- The mirror of create-story: that opens a story's folder and document, this closes them; the branch and worktree the persona opened, the persona removes at the close.
- Task outcome: `Completed`
