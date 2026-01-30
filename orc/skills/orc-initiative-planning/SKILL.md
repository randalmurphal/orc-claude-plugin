---
name: orc-initiative-planning
description: Use when work spans multiple tasks or requires shared context across implementations
---

# Orc Initiative Planning

## Overview

Use initiatives when work spans multiple tasks that share context. The initiative's vision and decisions flow into every linked task, keeping implementations aligned.

## When to Use Initiatives

| Situation | Use Initiative? |
|-----------|-----------------|
| Single isolated feature | No - just a task |
| Multiple related features | Yes |
| Architectural change | Yes |
| "I want to add A, B, and C" | Yes |
| Shared context needed across tasks | Yes |
| Sequential work building on itself | Yes |

## Creating an Initiative

```bash
orc initiative new "Initiative Title" --vision "Shared context that applies to all sub-tasks"
```

**Vision** is the key. It flows into every linked task's prompts. Include:
- Overall goal
- Architectural decisions
- Constraints
- Patterns to follow

### Example

```bash
orc initiative new "User Authentication System" --vision "JWT-based auth with refresh tokens. Use bcrypt for password hashing. Store sessions in Redis. All endpoints under /api/auth/*."
```

## Recording Decisions

As you make decisions that affect multiple tasks, record them:

```bash
orc initiative decide INIT-XXX "Use bcrypt for password hashing" --rationale "Industry standard, well-tested, appropriate cost factor for our scale"
```

Decisions also flow into linked task prompts. This prevents Claude from re-deciding the same things differently in each task.

### Superseding Decisions

If a decision changes:

```bash
orc initiative decide INIT-XXX "Use Argon2 instead of bcrypt" --rationale "Better resistance to GPU attacks" --supersedes 1
```

The old decision is marked superseded, new one takes precedence.

## Creating Linked Tasks

```bash
orc new "Implement login endpoint" -w medium -i INIT-XXX --spec-content "..."
orc new "Implement logout endpoint" -w medium -i INIT-XXX --spec-content "..."
orc new "Add JWT refresh logic" -w medium -i INIT-XXX --spec-content "..."
```

Each task inherits:
- Initiative vision
- All recorded decisions
- Context from the initiative

## Task Dependencies

For sequential work:

```bash
orc new "Part 2: Add validation" -w medium -i INIT-XXX --blocked-by TASK-001
```

Blocked tasks wait for dependencies to complete.

## Running Initiative Tasks

Run all ready tasks in order:

```bash
orc initiative run INIT-XXX
```

Or run specific tasks:

```bash
orc run TASK-001
# validate
orc run TASK-002
# validate
```

## Viewing Initiative Status

```bash
orc initiative show INIT-XXX
```

Shows:
- Vision
- Decisions (with rationale)
- Linked tasks and their status

## Initiative Workflow

1. **Identify multi-task work** during brainstorming
2. **Create initiative** with clear vision
3. **Break down into tasks** with appropriate weights
4. **Record decisions** as they're made
5. **Run tasks** in dependency order
6. **Validate each** before proceeding

## Example: Full Flow

**User says:** "I want to add a complete user authentication system with login, logout, and password reset"

**Brainstorming identifies:** This is multi-task work needing shared context.

**Create initiative:**
```bash
orc initiative new "User Authentication" --vision "Complete auth system: JWT tokens, bcrypt passwords, email-based password reset. Session duration 7 days. Rate limit login attempts."
```

**Break down:**
```bash
orc new "Implement login endpoint" -w medium -i INIT-001 --spec-content "..."
orc new "Implement logout endpoint" -w medium -i INIT-001 --spec-content "..."
orc new "Add password reset flow" -w medium -i INIT-001 --spec-content "..."
```

**Record decisions as they emerge:**
```bash
orc initiative decide INIT-001 "Store refresh tokens in Redis" --rationale "Fast lookup, automatic expiry support"
```

**Execute:**
```bash
orc initiative run INIT-001
```

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| Create initiative for single task | Just use a task |
| Forget to link tasks | Use `-i INIT-XXX` flag |
| Make decisions without recording | `orc initiative decide` |
| Run dependent tasks in parallel | Let orc handle ordering |
| Create vague vision | Be specific about patterns, constraints |
