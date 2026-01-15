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

## Step 2: Understand Task Context

For READY tasks you're considering, check their scope:
```bash
cat .orc/tasks/TASK-XXX/task.yaml 2>/dev/null
cat .orc/tasks/TASK-XXX/spec.md 2>/dev/null
```

If no spec exists and task belongs to an initiative, the initiative often has context:
```bash
cat .orc/initiatives/INIT-XXX.yaml 2>/dev/null
```

## Step 3: Plan Execution (up to 3 parallel)

Before running multiple tasks, check for conflicts:

**Run serial if:**
- Tasks modify the same files
- Tasks touch the same component/module
- One task depends on another

**Safe to parallelize:**
- Tasks in different areas of the codebase
- Independent bug fixes
- Docs alongside code tasks

Present your plan before executing.

## Step 4: Run Tasks

Delegate implementation - do not code yourself:

```bash
orc run TASK-XXX
```

For parallel execution (max 3):
```bash
orc run TASK-001
orc run TASK-002
orc run TASK-003
```

Set `run_in_background: true` on each Bash call, then **stop and wait**. You will be notified when tasks complete.

## Step 5: Validate After Completion

When notified of completion:
```bash
orc status --plain 2>/dev/null
orc diff TASK-XXX --stat
orc show TASK-XXX --plain
```

If the diff is significant or touches critical files, read and review them.

## Step 6: Handle Issues

If you find problems during validation:
```bash
orc new "Fix: [description]" --priority high
```

If the new task blocks other work, run it immediately before continuing.

## Step 7: Continue or Stop

```bash
orc status --plain
```

- More READY tasks? Plan next batch and continue
- Blocked on architecture/vision decisions? Ask the user
- All work done? Report summary and stop

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
| List by initiative | `orc list --initiative INIT-XXX` |
