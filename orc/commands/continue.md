---
description: "Orchestrate task execution with dependency awareness"
argument-hint: "[TASK-ID|--initiative INIT-ID|--all]"
---

# Orc Continue Command

Intelligent multi-task orchestrator that surveys project state, handles dependencies, runs tasks in background, and validates completion based on automation profile.

## Phase 1: State Survey

First, gather the current project state:

```bash
# Get task categorization (RUNNING, BLOCKED, READY, PAUSED, RECENT)
orc status --all --plain 2>/dev/null

# Get initiative list with progress
orc initiative list --plain 2>/dev/null

# Get current automation profile
orc config get profile 2>/dev/null
```

### Parse the Status Output

The `orc status` output is organized in priority sections:

| Section | Meaning |
|---------|---------|
| `RUNNING` | Tasks currently executing (shows current phase) |
| `BLOCKED` | Tasks with incomplete blockers (shows which tasks) |
| `READY` | Tasks that can be started immediately |
| `PAUSED` | Tasks paused mid-execution (can resume) |
| `RECENT` | Recently completed tasks (last 24h) |

Extract from each task line:
- Task ID (TASK-XXX format)
- Title
- Current phase (if running)
- Blocker info (if blocked, shows "by TASK-XXX")

## Phase 2: User Interaction

Based on the state survey, present options to the user:

### Scenario: Running Task Exists

```
TASK-XXX is currently running in [phase] phase.

Options:
1. Monitor until complete, then continue with ready tasks
2. Let it run and start additional tasks in parallel
3. Just show current status

What would you like to do?
```

### Scenario: Paused Task Exists

```
TASK-XXX is paused in [phase] phase.

Options:
1. Resume TASK-XXX
2. Start fresh with ready tasks (leave paused)
3. Resume AND start ready tasks in parallel

What would you like to do?
```

### Scenario: Only Ready Tasks

```
[N] tasks are ready to run.

Options:
1. Run all ready tasks (I'll decide parallelism)
2. Let me pick specific tasks
3. Run tasks for a specific initiative

What would you like to do?
```

If user picks specific tasks, present a numbered list:

```
## Ready Tasks

| # | ID | Priority | Weight | Title |
|---|-----|----------|--------|-------|
| 1 | TASK-189 | critical | medium | CRITICAL: Worktree cleanup... |
| 2 | TASK-159 | normal | large | Phase 0: Visual regression... |
...

Enter task numbers (e.g., "1,3,5" or "all" or "1-5"):
```

### Scenario: All Tasks Blocked

```
All [N] incomplete tasks are blocked.

Blocker chain:
- TASK-178 needs: TASK-176 (running)
- TASK-179 needs: TASK-178 (blocked)
...

[Root blocker TASK-176 is running. Monitor until complete?]
```

### Scenario: No Tasks

```
No tasks found in project.

Run `/orc:init` to create a new task or spec.
```

## Phase 3: Dependency Analysis

For each selected task, check for blockers:

```bash
# Read task to check blocked_by field
cat .orc/tasks/TASK-XXX/task.yaml
```

### Build Execution Order

1. **Group by dependency level**: Tasks with no blockers = level 0, tasks blocked by level 0 = level 1, etc.
2. **Run blockers first**: If user selected TASK-B which is blocked by TASK-A, run TASK-A first
3. **Parallel within level**: Tasks at the same dependency level can run parallel (if safe)

### Parallelism Decision

Decide parallelism based on:

| Factor | Serial | Parallel OK |
|--------|--------|-------------|
| Task weight | large/greenfield | trivial/small/medium |
| Same files | Yes | No overlap |
| Same initiative | Consider serial | Different initiatives |
| Count | N/A | Max 3 concurrent |

Present execution plan:

```
## Execution Plan

**Parallel Group 1** (start now):
- TASK-189 (medium, critical)
- TASK-188 (medium)

**Serial Queue** (after group 1):
- TASK-159 (large - needs focused execution)

Proceed? [Y/n]
```

## Phase 4: Execution

### Run Tasks in Background

Use Claude Code's `run_in_background` to start tasks without blocking:

```bash
orc run TASK-XXX
```

Set `run_in_background: true` on the Bash tool call.

### Wait for Completion

Use the `TaskOutput` tool to wait for the background task:

```
TaskOutput(task_id=<shell_id_from_bash>, block=true)
```

This returns when `orc run` completes, including exit code and output.

### Track Multiple Parallel Tasks

If running multiple tasks in parallel:
1. Start all tasks with `run_in_background: true`
2. Collect their task IDs
3. Use `TaskOutput` with `block=true` on each (they complete independently)
4. As each completes, check its result and update tracking

### Periodic Status Check

While waiting for long-running tasks:

```bash
orc status --plain 2>/dev/null
```

Report progress:
```
[10:32:15] TASK-189: test phase (3/4 phases complete)
[10:33:20] TASK-188: completed successfully
```

## Phase 5: Completion Validation

After `orc run` exits, validate the final state:

```bash
# Read final state
cat .orc/tasks/TASK-XXX/state.yaml
cat .orc/tasks/TASK-XXX/task.yaml
```

### Profile-Aware Validation

| Profile | Expected Success State | Gate Stop State |
|---------|----------------------|-----------------|
| `auto` | `completed` or `finished` | N/A - fully automated |
| `fast` | `completed` or `finished` | N/A - fully automated |
| `safe` | `completed` (PR created) | Human merge approval needed |
| `strict` | May stop at spec/merge gate | Human approval needed |

### Success Report (auto/fast)

```
## TASK-XXX Completed

Status: completed -> finished
PR: https://github.com/owner/repo/pull/123
  - Auto-approved
  - CI passed
  - Merged to main

Commit: abc1234
Duration: 15m 32s
```

### Gate Stop Report (safe/strict)

```
## TASK-XXX At Gate

Status: completed (awaiting merge approval)
PR: https://github.com/owner/repo/pull/123
  - Needs human approval

To approve and merge:
  orc approve TASK-XXX

Or review PR at the link above.
```

### Failure Report

```
## TASK-XXX Failed

Status: failed
Phase: test
Error: Tests failing after 5 retries

Options:
1. Retry: `orc resume TASK-XXX`
2. Skip and continue with other tasks
3. Investigate transcript: `.orc/tasks/TASK-XXX/transcripts/`

What would you like to do?
```

## Phase 6: Loop/Continue

After each task completes:

### Check for Newly Unblocked Tasks

```bash
# Re-check status to see if blocked tasks are now ready
orc status --plain 2>/dev/null
```

If TASK-A completed and TASK-B was blocked by TASK-A:
- TASK-B should now appear in READY section
- Add to execution queue

### Continue Decision

| Condition | Action |
|-----------|--------|
| More tasks in selection queue | Continue to next task |
| Blocked tasks now unblocked | Add to queue, continue |
| All selected tasks done | Report final summary |
| Task failed | Offer retry/skip/stop |
| Human gate reached (strict) | Report gate, offer to continue with others |

### Final Summary

When orchestration session ends:

```
## Session Complete

### Results
| Task | Final Status | Duration | Notes |
|------|--------------|----------|-------|
| TASK-188 | finished | 12m | Merged to main |
| TASK-189 | completed | 25m | PR #124 awaiting CI |
| TASK-159 | failed | 8m | Test failures |

### Remaining Work
- 2 tasks still blocked (waiting on TASK-159)
- 10 tasks ready to run

### Suggested Next Steps
1. Fix TASK-159: `orc resume TASK-159`
2. Once fixed, unblocked tasks become ready
3. Run `/orc:continue` to process remaining tasks
```

## Orchestration Completion

When the orchestration session ends, output:

```xml
<orchestration_complete>
  <tasks_completed>N</tasks_completed>
  <tasks_failed>N</tasks_failed>
  <tasks_pending>N</tasks_pending>
  <session_duration>Xm Ys</session_duration>
</orchestration_complete>
```

## Edge Cases

### Interrupted Execution

If the user interrupts (Ctrl+C) or session ends:
- Background `orc run` processes continue running
- Check status with `orc status` later
- Resume with `/orc:continue`

### Conflict Detection

If you suspect tasks might conflict (modifying same files):
1. Read task specs/descriptions to identify touched files
2. Check if tasks are in same initiative (higher conflict risk)
3. Recommend serial execution for conflicting tasks
4. User can override with explicit parallel request

### Mixed Weight Tasks

When running mixed weights:
- Start small/medium tasks in parallel
- Queue large/greenfield tasks for serial execution
- This optimizes for throughput while giving complex tasks focus

## Context Files Reference

| File | Purpose |
|------|---------|
| `.orc/tasks/TASK-XXX/task.yaml` | Task definition, blocked_by, status |
| `.orc/tasks/TASK-XXX/state.yaml` | Execution state, current phase, errors |
| `.orc/tasks/TASK-XXX/plan.yaml` | Phase sequence for this task |
| `.orc/tasks/TASK-XXX/spec.md` | Task specification (if exists) |
| `.orc/tasks/TASK-XXX/transcripts/` | Previous execution logs |
| `.orc/config.yaml` | Project configuration, profile setting |
| `.orc/initiatives/INIT-XXX.yaml` | Initiative definition and task list |
