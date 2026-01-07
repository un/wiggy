# CLI & TUI Design

Wiggy provides a terminal user interface (TUI) for managing AI coding agents. Run `wiggy` to launch.

## Installation

```bash
# Install globally
npm install -g wiggy

# Or run directly
npx wiggy
```

## Quick Start

```bash
# Launch the TUI
wiggy

# Or run specific commands
wiggy status              # Show current state
wiggy sync                # Sync with GitHub
wiggy run <prd-id> -n 3   # Run 3 agents on a PRD
```

## TUI Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│  WIGGY                                              ⟳ Synced │ 3 Active │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─ Projects ──────────────────────────────────────────────────────┐   │
│  │                                                                  │   │
│  │  > my-app           3 PRDs   12 tasks   ████████░░ 75%          │   │
│  │    backend-api      1 PRD    5 tasks    ██████████ 100%         │   │
│  │    shared-utils     2 PRDs   8 tasks    ███░░░░░░░ 25%          │   │
│  │                                                                  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─ Active Agents ─────────────────────────────────────────────────┐   │
│  │                                                                  │   │
│  │  agent-a1b2  my-app/task-003   implementing   ●●●○○  3/5 iter   │   │
│  │  agent-c3d4  my-app/task-004   testing        ●●●●○  4/5 iter   │   │
│  │  agent-e5f6  backend/task-001  pr_open        ●●●●●  done       │   │
│  │                                                                  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─ Notifications ─────────────────────────────────────────────────┐   │
│  │                                                                  │   │
│  │  ⚠ task-002 waiting_for_human: "Unclear requirements for..."    │   │
│  │                                                                  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│  n:New  r:Run  s:Sync  Enter:Select  ?:Help  q:Quit                     │
└─────────────────────────────────────────────────────────────────────────┘
```

## Views

### 1. Dashboard (Home)

The default view showing:
- Project list with progress
- Active agent sessions
- Notifications (waiting_for_human items)

### 2. Project View

Drill into a project to see:
- Project details and config
- PRD list with status
- Recent activity

```
┌─ my-app ────────────────────────────────────────────────────────────────┐
│                                                                         │
│  Path: /Users/me/code/my-app                                            │
│  Remote: git@github.com:me/my-app.git                                   │
│  Branch: main                                                           │
│                                                                         │
│  ┌─ PRDs ───────────────────────────────────────────────────────────┐  │
│  │                                                                   │  │
│  │  > prd-auth        User Authentication     active    ██████░░░░  │  │
│  │    prd-dashboard   Admin Dashboard         planning  ░░░░░░░░░░  │  │
│  │    prd-api-v2      API Redesign            draft     ░░░░░░░░░░  │  │
│  │                                                                   │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3. PRD View

Drill into a PRD to see:
- User stories
- Task list with statuses
- Linked PRs

```
┌─ prd-auth: User Authentication ─────────────────────────────────────────┐
│                                                                         │
│  Status: active                     Progress: ██████░░░░ 6/10 tasks     │
│                                                                         │
│  ┌─ Tasks ──────────────────────────────────────────────────────────┐  │
│  │                                                                   │  │
│  │  ✓ task-001  Create User model              pr_merged            │  │
│  │  ✓ task-002  Add password hashing           pr_merged            │  │
│  │  ✓ task-003  Implement login endpoint       pr_merged            │  │
│  │  ✓ task-004  Implement register endpoint    pr_merged            │  │
│  │  ● task-005  Add JWT token generation       in_progress          │  │
│  │  ● task-006  Create auth middleware         in_progress          │  │
│  │  ○ task-007  Add refresh token flow         backlog              │  │
│  │  ○ task-008  Implement logout               backlog              │  │
│  │  ○ task-009  Add password reset             backlog              │  │
│  │  ○ task-010  Write integration tests        backlog              │  │
│  │                                                                   │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌─ Linked PRs ─────────────────────────────────────────────────────┐  │
│  │  #42 [task-001] Create User model                      merged    │  │
│  │  #43 [task-002] Add password hashing                   merged    │  │
│  │  #45 [task-005] Add JWT token generation               open      │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4. Task View

Drill into a task to see:
- Full task details
- Acceptance criteria
- Progress log
- Agent session info

```
┌─ task-005: Add JWT token generation ────────────────────────────────────┐
│                                                                         │
│  Status: in_progress                                                    │
│  Agent: agent-a1b2                                                      │
│  Branch: wiggy/task-005                                                 │
│  PR: #45 (open)                                                         │
│                                                                         │
│  ┌─ Acceptance Criteria ────────────────────────────────────────────┐  │
│  │  [x] Generate JWT on successful login                            │  │
│  │  [x] Include user ID and role in payload                         │  │
│  │  [ ] Set configurable expiration time                            │  │
│  │  [ ] Add token validation utility                                │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌─ Progress Log ───────────────────────────────────────────────────┐  │
│  │  [14:32] Started task implementation                             │  │
│  │  [14:33] Analyzed existing auth code in src/auth/                │  │
│  │  [14:35] Created src/auth/jwt.ts with sign/verify functions      │  │
│  │  [14:36] Committed: "Add JWT utility functions"                  │  │
│  │  [14:38] Updated login handler to return JWT                     │  │
│  │  [14:39] Tests passing (12/12)                                   │  │
│  │  [14:40] Working on token expiration config...                   │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5. Agent View

Show details of a specific agent session:
- Real-time output
- Metrics
- Controls (pause/abort)

## Hotkeys

### Global

| Key       | Action                          |
|-----------|---------------------------------|
| `q`       | Quit                            |
| `?`       | Show help                       |
| `s`       | Sync control repo               |
| `Esc`     | Back / Cancel                   |
| `1`       | Go to Dashboard                 |
| `2`       | Go to Projects                  |
| `3`       | Go to Agents                    |

### Navigation

| Key       | Action                          |
|-----------|---------------------------------|
| `↑` / `k` | Move up                         |
| `↓` / `j` | Move down                       |
| `Enter`   | Select / Drill in               |
| `Esc`     | Go back                         |
| `/`       | Search / Filter                 |

### Actions

| Key       | Action                          |
|-----------|---------------------------------|
| `n`       | New (context-sensitive)         |
| `r`       | Run agents (on PRD)             |
| `e`       | Edit selected item              |
| `d`       | Delete (with confirmation)      |
| `p`       | Pause agent                     |
| `x`       | Abort agent                     |

### Quick New

| Key       | Action                          |
|-----------|---------------------------------|
| `N p`     | New Project                     |
| `N r`     | New PRD                         |
| `N t`     | New Task                        |

## CLI Commands

When not in TUI mode, Wiggy supports direct commands:

### `wiggy`

Launch the TUI (default).

```bash
wiggy
```

### `wiggy status`

Show current state summary without TUI.

```bash
wiggy status

# Output:
# Projects: 3
# Active PRDs: 2
# Running Agents: 3
# Waiting for Human: 1
```

### `wiggy sync`

Manually sync the control repo.

```bash
wiggy sync

# Output:
# Pulling from origin/main...
# Pushing to origin/main...
# Synced successfully.
```

### `wiggy new project`

Add a new project interactively.

```bash
wiggy new project

# Prompts:
# Project name: My App
# Path to repository: /Users/me/code/my-app
# Git remote (optional): git@github.com:me/my-app.git
#
# Created project: my-app
```

### `wiggy new prd <project-id>`

Create a new PRD with interactive planning.

```bash
wiggy new prd my-app

# Opens OpenCode in Plan Mode for requirements extraction
# Then generates user stories and tasks
```

### `wiggy run <prd-id>`

Run agents on a PRD.

```bash
# Run with defaults (3 agents, 10 iterations)
wiggy run prd-auth

# Specify number of agents
wiggy run prd-auth -n 5

# Specify iterations per agent
wiggy run prd-auth -n 3 -i 15

# Run in background
wiggy run prd-auth --background
```

### `wiggy agents`

List active agents.

```bash
wiggy agents

# Output:
# ID         Task          Status        Iterations
# agent-a1   task-005      implementing  3/10
# agent-b2   task-006      testing       8/10
# agent-c3   task-007      pr_open       done
```

### `wiggy agent <id>`

Show agent details / attach to output.

```bash
wiggy agent agent-a1

# Shows live agent output
# Ctrl+C to detach (agent continues running)
```

### `wiggy abort <agent-id>`

Abort a running agent.

```bash
wiggy abort agent-a1

# Output:
# Aborting agent-a1...
# Agent aborted. Task task-005 returned to backlog.
```

### `wiggy config`

Edit configuration.

```bash
# Open config in editor
wiggy config

# Show current config
wiggy config --show

# Set a value
wiggy config set agents.maxConcurrent 5
```

## Configuration

### Config File Location

- Global: `~/.config/wiggy/config.json`
- Project: `./wiggy.json` (in control repo root)

### Config Options

```json
{
  "controlRepo": {
    "remote": "git@github.com:me/wiggy-control.git",
    "autoSync": true,
    "syncIntervalMs": 30000
  },
  "agents": {
    "maxConcurrent": 5,
    "defaultIterations": 10,
    "lockTimeoutMs": 1800000
  },
  "notifications": {
    "desktop": true,
    "webhooks": []
  },
  "tui": {
    "theme": "default",
    "refreshIntervalMs": 1000
  }
}
```

## Desktop Notifications

Wiggy sends desktop notifications for:

- **Task waiting for human** - When an agent gets stuck
- **PRD completed** - When all tasks in a PRD are done
- **Agent error** - When an agent fails unexpectedly

Notifications use the system notification center (macOS/Linux/Windows).

## Implementation Notes

### Technology Stack

- **Framework**: OpenTUI (`@opentui/react`)
- **Rendering**: React-based component model
- **State**: Zustand for TUI state management
- **CLI Parsing**: Commander.js
- **Notifications**: node-notifier

### Component Structure

```
src/cli/
├── index.tsx           # Entry point
├── App.tsx             # Main app component
├── hooks/
│   ├── useProjects.ts
│   ├── usePRDs.ts
│   ├── useTasks.ts
│   ├── useAgents.ts
│   └── useSync.ts
├── components/
│   ├── Dashboard.tsx
│   ├── ProjectList.tsx
│   ├── ProjectView.tsx
│   ├── PRDList.tsx
│   ├── PRDView.tsx
│   ├── TaskList.tsx
│   ├── TaskView.tsx
│   ├── AgentList.tsx
│   ├── AgentView.tsx
│   ├── Notifications.tsx
│   ├── StatusBar.tsx
│   └── Help.tsx
├── screens/
│   ├── NewProject.tsx
│   ├── NewPRD.tsx
│   └── RunAgents.tsx
└── utils/
    ├── keybindings.ts
    └── notifications.ts
```

### Example Component

```tsx
import { Box, Text, useInput } from '@opentui/react'
import { useProjects } from '../hooks/useProjects'

export function ProjectList() {
  const { projects, selectedIndex, setSelectedIndex } = useProjects()
  
  useInput((input, key) => {
    if (key.upArrow || input === 'k') {
      setSelectedIndex(Math.max(0, selectedIndex - 1))
    }
    if (key.downArrow || input === 'j') {
      setSelectedIndex(Math.min(projects.length - 1, selectedIndex + 1))
    }
  })
  
  return (
    <Box flexDirection="column" borderStyle="round" title="Projects">
      {projects.map((project, i) => (
        <Box key={project.id}>
          <Text inverse={i === selectedIndex}>
            {i === selectedIndex ? '>' : ' '} {project.name}
          </Text>
          <Text> {project.stats.totalPRDs} PRDs</Text>
        </Box>
      ))}
    </Box>
  )
}
```
