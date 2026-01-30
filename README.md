# Orc Claude Plugin

Claude Code plugin providing discipline skills and [orc](https://github.com/randalmurphal/orc) guidance.

## Installation

In Claude Code, run:

```
/plugin marketplace add randalmurphal/orc-claude-plugin
/plugin install orc@orc
```

## Philosophy

This plugin provides two things:

1. **Discipline skills** - Change HOW Claude works (TDD, debugging, verification)
2. **Orc guidance** - Help using orc WHEN you want orchestration

These are orthogonal. The discipline skills apply whether you're using orc or not. Orc is a tool you use when work needs isolation, phased execution, or is too big for one session.

## Skills

### Discipline Skills

These change Claude's behavior to follow best practices automatically.

| Skill | Purpose |
|-------|---------|
| **brainstorming** | Socratic dialogue to understand intent before implementation |
| **test-driven-development** | Write failing test first, then implement |
| **systematic-debugging** | Find root cause before attempting fixes |
| **verification-before-completion** | Evidence before claims, always |

### Orc Guidance Skills

These help you use orc effectively when you choose to.

| Skill | Purpose |
|-------|---------|
| **orc-task-execution** | How to run and monitor orc tasks |
| **orc-initiative-planning** | How to break work into initiatives with linked tasks |

## When to Use Orc

Orc adds value when:
- Work is too big for one Claude session (context pollution)
- Need fresh context per concern (prevents drift)
- Want async background execution
- Want multi-agent code review

**You decide when to use orc.** The skills don't force it.

Examples:
- "Add email validation" → Claude does it directly with TDD
- "Add user auth, break it into an initiative" → Claude uses orc
- "This is getting complex, let's use orc" → Claude creates tasks

## Example Usage

### Direct work (most common)

```
User: "Add validation to the email field"
Claude: (brainstorming) asks clarifying questions
Claude: (TDD) writes failing test, implements, verifies
Done.
```

### Using orc for larger work

```
User: "I want to add user authentication. Let's break this into an orc initiative."
Claude: Creates initiative with vision
Claude: Creates linked tasks for each concern
Claude: Runs tasks, monitors progress
Claude: Reports results
```

## CLI Commands

When using orc:

| Command | Purpose |
|---------|---------|
| `orc status` | View active tasks |
| `orc new "title"` | Create task |
| `orc run TASK-XXX` | Execute task |
| `orc show TASK-XXX` | Task details |
| `orc diff TASK-XXX` | What changed |
| `orc initiative new "title"` | Create initiative |
| `orc initiative decide INIT-XXX "decision"` | Record decision |

## Related

- [orc](https://github.com/randalmurphal/orc) - Task orchestrator
- [Superpowers](https://github.com/obra/superpowers-marketplace) - Inspiration for discipline skills
