---
description: "Orchestrate task execution with dependency awareness"
argument-hint: "[TASK-ID|--all]"
---

# Orc Continue Command

**CRITICAL: You are an ORCHESTRATOR, not an implementer. You run `orc run` commands and monitor their completion. You do NOT implement tasks yourself.**

## Step 1: Get Current State

Run this command NOW:

```bash
orc status --plain 2>/dev/null
```

## Step 2: Identify What to Run

From the status output, identify tasks in these sections:

- **RUNNING**: A task is already executing. Ask user: "TASK-XXX is running. Monitor it or start another in parallel?"
- **PAUSED**: A task was paused. Ask user: "Resume TASK-XXX?"
- **READY**: Tasks available to start. List them and ask which to run.
- **BLOCKED**: Tasks waiting on dependencies. Show what they're blocked by.

If no RUNNING, PAUSED, or READY tasks exist, tell the user and suggest `/orc:init`.

## Step 3: Run the Task

**DO NOT implement the task yourself. Run orc to do it:**

```bash
orc run TASK-XXX
```

Use `run_in_background: true` on the Bash tool call so you can monitor progress.

## Step 4: Wait for Completion

Use the TaskOutput tool to wait for the background task:

```
TaskOutput(task_id=<task_id_from_step_3>, block=true, timeout=600000)
```

This blocks until `orc run` completes (up to 10 minutes).

## Step 5: Check Result

After TaskOutput returns, check the task status:

```bash
orc show TASK-XXX --plain 2>/dev/null
```

Report the result to the user:
- **completed/finished**: Task succeeded. Check if more tasks are now READY.
- **failed**: Task failed. Offer `orc resume TASK-XXX` to retry.
- **blocked**: Task hit a gate. Show what's needed.

## Step 6: Continue or Stop

Ask the user if they want to continue with remaining tasks, or stop here.

If continuing, go back to Step 1 to get fresh status.

## Running Multiple Tasks in Parallel

If user wants parallel execution:

1. Start each task with `run_in_background: true`:
   ```bash
   orc run TASK-001
   orc run TASK-002
   ```

2. Wait for each with TaskOutput (they run concurrently)

3. Report results as they complete

**Max 3 parallel tasks** to avoid resource contention.

## What You Must NOT Do

- **DO NOT** read task specs and implement them yourself
- **DO NOT** write code, run tests, or make changes directly
- **DO NOT** investigate the codebase to understand tasks
- **DO NOT** use any tool except Bash (for orc commands) and TaskOutput (for waiting)

You are a dispatcher. `orc run` does the actual work.

## Quick Reference

| Action | Command |
|--------|---------|
| Check status | `orc status --plain` |
| Run a task | `orc run TASK-XXX` (with run_in_background: true) |
| Show task details | `orc show TASK-XXX --plain` |
| Resume paused/failed | `orc resume TASK-XXX` |
| List by initiative | `orc list --initiative INIT-XXX` |
