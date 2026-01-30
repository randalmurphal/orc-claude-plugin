---
name: orc-task-execution
description: Use when running orc tasks or monitoring task progress. Guides background execution and validation.
---

# Orc Task Execution

## Overview

Run orc tasks in the background and validate results when complete. Don't implement directly - delegate to orc and verify the output.

**Core principle:** Delegate implementation to `orc run`, then validate the results.

## Running Tasks

### Start a Task

```bash
orc run TASK-XXX
```

Run in background so the conversation can continue. The task executes phases autonomously.

### Monitor Progress

| Command | Purpose |
|---------|---------|
| `orc status` | Overview of all tasks |
| `orc show TASK-XXX` | Detailed task info, current phase |
| `orc log TASK-XXX --follow` | Stream live execution logs |

### Check Completion

When a task finishes:

1. **Check status:**
   ```bash
   orc show TASK-XXX
   ```
   Verify status is `complete`, not `failed` or `blocked`.

2. **Review changes:**
   ```bash
   orc diff TASK-XXX
   ```
   Shows what files changed and the diff.

3. **Read key files:**
   If diff looks significant, read the actual changed files to understand the implementation.

4. **Validate functionality:**
   Run tests, try the feature, verify it works as intended.

## Task States

| State | Meaning | Action |
|-------|---------|--------|
| `running` | Task executing | Wait, or `orc log --follow` |
| `complete` | All phases done | Validate results |
| `failed` | Phase failed | Check error, `orc resume` or create follow-up |
| `blocked` | Needs input | Check what's blocking, `orc approve` if gate |
| `paused` | Manually paused | `orc resume` when ready |

## When Tasks Fail

1. **Check the error:**
   ```bash
   orc show TASK-XXX
   ```
   Look at the `error` field.

2. **Review logs:**
   ```bash
   orc log TASK-XXX
   ```
   Understand what went wrong.

3. **Options:**
   - `orc resume TASK-XXX` - Retry from failed phase
   - Create follow-up task to fix the issue
   - `orc resolve TASK-XXX` - Mark as resolved (manual fix)

## Validation Checklist

Before reporting a task as complete to the user:

- [ ] `orc show TASK-XXX` shows status `complete`
- [ ] `orc diff TASK-XXX` reviewed - changes look correct
- [ ] Tests pass (if applicable)
- [ ] Feature works as intended (if testable)
- [ ] No obvious issues in the implementation

**Never claim a task is done without verification.**

## Running Multiple Tasks

For independent tasks, run them in parallel:

```bash
orc run TASK-001
orc run TASK-002
orc run TASK-003
```

Check all with `orc status`.

For dependent tasks (one builds on another), run sequentially and validate between each.

## Initiative Execution

For tasks linked to an initiative:

```bash
orc initiative run INIT-XXX
```

Runs all ready tasks in dependency order.

## Common Patterns

### "Run this task"

```bash
orc run TASK-XXX
```
Then monitor and validate.

### "What's the status?"

```bash
orc status
```
Shows all active tasks grouped by state.

### "Show me what changed"

```bash
orc diff TASK-XXX
```
After task completes, shows the changes made.

### "Something went wrong"

```bash
orc show TASK-XXX  # See error
orc log TASK-XXX   # See full transcript
```
Then decide: resume, fix manually, or create follow-up task.

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| Implement directly when task exists | `orc run TASK-XXX` |
| Claim done without checking | Verify with `orc show`, `orc diff` |
| Ignore failed tasks | Investigate, resume, or follow-up |
| Run dependent tasks in parallel | Run in order, validate between |
