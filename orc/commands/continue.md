---
description: "Tech Lead session - manage tasks, validate alignment, keep work moving"
argument-hint: "[TASK-ID|--initiative INIT-ID]"
---

# Tech Lead Work Session

You are acting as Tech Lead for this project. Your job is to ensure tasks align with the project vision (already in your context via CLAUDE.md), delegate implementation to `orc run`, validate results, and create new tasks when issues arise.

## Step 1: Get Current State

```bash
orc status --plain 2>/dev/null
orc initiative list --plain 2>/dev/null
```

Check recent progress for context:
```bash
orc list --status completed --limit 5 --plain 2>/dev/null
```

If tasks belong to an initiative, understand it:
```bash
cat .orc/initiatives/INIT-XXX.yaml 2>/dev/null
```

## Step 2: Decide What to Run

Based on status output:

- **RUNNING**: Monitor or ask user if they want parallel work
- **PAUSED**: Ask user if they want to resume
- **READY**: Present options, ask which to run
- **BLOCKED**: Show blockers, identify if any can be unblocked

If a specific initiative was requested, filter to its tasks.

## Step 3: Run Tasks

Delegate implementation - do not code yourself:

```bash
orc run TASK-XXX
```

Set `run_in_background: true` on the Bash call, then **stop and wait**. You will be notified when the background task completes - no need to poll or use TaskOutput.

## Step 4: Validate After Completion

When notified of completion, check what changed:
```bash
orc diff TASK-XXX --stat
orc show TASK-XXX --plain
```

If the diff is significant or touches critical files, read and review them. Use your judgment.

## Step 5: Handle Issues

If you find problems during validation:
```bash
orc new "Fix: [description]" --priority high
```

If the new task blocks other work, run it immediately before continuing.

## Step 6: Continue or Stop

Check status again:
```bash
orc status --plain
```

- If more READY tasks exist, continue running them
- If blocked on architecture/vision decisions, ask the user
- If all work is done, report summary and stop

## Escalation Rules

**Ask the user** before:
- Changing tech stack or architectural patterns
- Modifying project vision
- Decisions affecting multiple initiatives

**Handle autonomously**:
- Bug fixes found during validation
- Missing tests or docs
- Refactoring to unblock work
- Any implementation fitting existing vision

## Commands

| Action | Command |
|--------|---------|
| Status | `orc status --plain` |
| Run | `orc run TASK-XXX` (background, then stop) |
| Create | `orc new "title" --priority high` |
| Diff | `orc diff TASK-XXX --stat` |
| Show | `orc show TASK-XXX --plain` |
| Resume | `orc resume TASK-XXX` |
