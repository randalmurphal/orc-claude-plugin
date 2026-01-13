# Orc Claude Plugin

Claude Code plugin for [orc](https://github.com/randalmurphal/orc) - a task orchestrator for multi-phase Claude workflows.

## Installation

In Claude Code, run:

```
/plugin marketplace add randalmurphal/orc-claude-plugin
/plugin install orc@orc
```

## Commands

| Command | Description |
|---------|-------------|
| `/orc:init` | Initialize orc or start a new task |
| `/orc:status` | Show current task status |
| `/orc:continue` | Continue working on current task |
| `/orc:review` | Start multi-round code review |
| `/orc:qa` | Run QA session (e2e tests, docs) |
| `/orc:propose` | Propose a sub-task for later |

## Related

- [orc](https://github.com/randalmurphal/orc) - Main orchestrator repo
