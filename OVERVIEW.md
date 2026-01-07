# Wiggy

**Multi-agent AI coding orchestrator built on OpenCode**

## Vision

Wiggy is a system that lets you run multiple AI coding agents in parallel, working on different tasks within the same project or across multiple projects. It extends the "Ralph Wiggum" approach with:

- **Parallel execution** - Run N agents simultaneously on different tasks
- **Deep planning** - Structured requirements extraction before coding begins
- **Rich state management** - Task lifecycle beyond just "pass/fail"
- **Cross-project support** - Manage multiple repos from one control center
- **Human-in-the-loop** - Agents escalate when stuck, humans merge PRs
- **Continuity** - Agents save progress; new agents can resume interrupted work

## Core Concepts

### Control Repo

A dedicated Git repository (this repo) that stores all Wiggy state:

- Project configurations
- PRDs (Product Requirements Documents)
- Tasks with full lifecycle status
- Agent session logs and progress
- Prompt templates

The control repo syncs to GitHub, acting as a distributed database that coordinates agents across machines.

### Projects

A project is a target codebase that Wiggy manages. Each project links to:

- A local path to the repository
- A Git remote URL
- Project-specific configuration

### PRDs (Product Requirements Documents)

A PRD defines a feature or body of work. It contains:

- User stories with acceptance criteria
- Success metrics
- Linked tasks
- Overall status (draft → active → completed)

### Tasks

Tasks are atomic units of work derived from PRDs. Each task is:

- Small enough to complete in <500 lines of code
- Independently testable
- Assignable to a single agent

Task lifecycle:

```
backlog → in_progress → implemented → tested → pr_open → pr_merged
              ↓              ↓
         waiting_for_human ←─┘
```

### Agents

Agents are OpenCode sessions running in loops. Each agent:

- Claims a task (with lock + timestamp)
- Works in a Git worktree to avoid conflicts
- Commits frequently with green CI
- Creates a PR when done
- Logs progress for continuity

## Key Principles

1. **Keep CI Green** - Every commit must pass lint, types, and tests
2. **Small PRs** - Tasks target <500 lines to stay within context window
3. **Progress Logging** - Agents append to progress logs; never overwrite
4. **Optimistic Concurrency** - File-based locks with timestamps; stale locks can be claimed
5. **Human Authority** - Humans review PRs, resolve conflicts, merge to main

## Technology Stack

- **Runtime**: Node.js / Bun
- **Agent Engine**: OpenCode SDK (`@opencode-ai/sdk`)
- **TUI Framework**: OpenTUI (`@opentui/react`)
- **Language**: TypeScript
- **State**: JSON files with JSON Schema validation
- **Notifications**: node-notifier (desktop) + extensible webhooks

## Directory Structure

```
wiggy/                              # Control repo
├── wiggy.ts                        # CLI entrypoint
├── package.json
├── tsconfig.json
├── config.json                     # Global configuration
│
├── schemas/                        # JSON Schema definitions
│   ├── project.schema.json
│   ├── prd.schema.json
│   ├── task.schema.json
│   └── session.schema.json
│
├── prompts/                        # Prompt templates
│   ├── planning/
│   │   ├── requirements.md         # Extract requirements from human
│   │   ├── user-stories.md         # Generate user stories
│   │   └── task-breakdown.md       # Break PRD into tasks
│   ├── execution/
│   │   ├── implement.md            # Main implementation loop
│   │   ├── test.md                 # Write/run tests
│   │   └── pr.md                   # Create PR with description
│   └── review/
│       └── self-review.md          # Agent self-review before PR
│
├── projects/                       # One folder per target project
│   └── {project-slug}/
│       ├── project.json            # Project configuration
│       ├── prds/
│       │   └── {prd-id}.json       # PRD definitions
│       ├── tasks/
│       │   └── {task-id}.json      # Task definitions
│       └── sessions/
│           └── {session-id}/
│               ├── session.json    # Session metadata
│               ├── progress.md     # Append-only progress log
│               └── context.json    # Structured checkpoints
│
└── src/                            # Wiggy source code
    ├── cli/                        # TUI components
    ├── core/                       # Business logic
    ├── agents/                     # Agent orchestration
    └── sync/                       # Git sync utilities
```

## Quick Start

```bash
# Install wiggy globally
npm install -g wiggy

# Run the TUI
wiggy

# Or run specific commands
wiggy new project     # Add a new project
wiggy new prd         # Create a new PRD (interactive planning)
wiggy run <prd-id>    # Run agents on a PRD
wiggy status          # View current state
wiggy sync            # Sync control repo with GitHub
```

## Related Documents

- [ARCHITECTURE.md](./ARCHITECTURE.md) - Technical design and component details
- [SCHEMAS.md](./SCHEMAS.md) - JSON Schema definitions for all data types
- [PROMPTS.md](./PROMPTS.md) - Prompt templates for agents
- [CLI.md](./CLI.md) - TUI design, commands, and hotkeys
- [WORKFLOWS.md](./WORKFLOWS.md) - Step-by-step user workflows
