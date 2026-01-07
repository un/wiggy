# Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         WIGGY TUI                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │ Projects │  │   PRDs   │  │  Tasks   │  │  Active Agents   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      WIGGY CORE                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ State Mgmt  │  │ Agent Pool  │  │   Git Sync Engine       │  │
│  │ (JSON files)│  │ (OpenCode)  │  │   (Control Repo)        │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
   ┌─────────────┐    ┌─────────────┐     ┌─────────────┐
   │  Project A  │    │  Project B  │     │  Project C  │
   │  (worktree) │    │  (worktree) │     │  (worktree) │
   └─────────────┘    └─────────────┘     └─────────────┘
```

## Components

### 1. TUI (Terminal User Interface)

Built with `@opentui/react`. Provides:

#### Main Dashboard

- Project list with PRD/task counts
- Active agent sessions with live status
- Notifications queue (waiting_for_human items)

#### Project View

- PRD list with progress bars
- Quick actions: New PRD, Run Agents

#### PRD View

- User stories with status
- Task list with full lifecycle status
- Linked PRs

#### Task View

- Task details and acceptance criteria
- Assigned agent (if any)
- Progress log viewer
- Context checkpoints

#### Hotkeys

| Key     | Action                              |
| ------- | ----------------------------------- |
| `n`     | New (context-sensitive)             |
| `r`     | Run agents on selected PRD          |
| `s`     | Sync control repo                   |
| `Enter` | Drill into selected item            |
| `Esc`   | Back / Cancel                       |
| `q`     | Quit                                |
| `?`     | Help                                |
| `1-5`   | Switch views (Dashboard/Projects/PRDs/Tasks/Agents) |

### 2. Core Engine

#### State Manager

- CRUD operations on JSON files
- JSON Schema validation on read/write
- Optimistic locking with timestamps
- Atomic file operations (write to temp, rename)

```typescript
interface StateManager {
  // Projects
  listProjects(): Promise<Project[]>
  getProject(id: string): Promise<Project>
  createProject(data: CreateProjectInput): Promise<Project>
  updateProject(id: string, data: Partial<Project>): Promise<Project>
  
  // PRDs
  listPRDs(projectId: string): Promise<PRD[]>
  getPRD(projectId: string, prdId: string): Promise<PRD>
  createPRD(projectId: string, data: CreatePRDInput): Promise<PRD>
  updatePRD(projectId: string, prdId: string, data: Partial<PRD>): Promise<PRD>
  
  // Tasks
  listTasks(projectId: string, prdId?: string): Promise<Task[]>
  getTask(projectId: string, taskId: string): Promise<Task>
  claimTask(projectId: string, taskId: string, agentId: string): Promise<Task>
  updateTaskStatus(projectId: string, taskId: string, status: TaskStatus): Promise<Task>
  
  // Sessions
  listSessions(projectId: string): Promise<Session[]>
  getSession(projectId: string, sessionId: string): Promise<Session>
  appendProgress(projectId: string, sessionId: string, entry: string): Promise<void>
  updateContext(projectId: string, sessionId: string, context: Context): Promise<void>
}
```

#### Agent Pool

- Spawns OpenCode sessions via SDK
- Manages concurrent agent limit
- Routes tasks to available agents
- Monitors agent health

```typescript
interface AgentPool {
  // Pool management
  start(maxAgents: number): Promise<void>
  stop(): Promise<void>
  
  // Agent lifecycle
  spawnAgent(task: Task): Promise<Agent>
  getAgent(agentId: string): Agent | undefined
  listActiveAgents(): Agent[]
  
  // Events
  on(event: 'agent:started', handler: (agent: Agent) => void): void
  on(event: 'agent:completed', handler: (agent: Agent, result: AgentResult) => void): void
  on(event: 'agent:failed', handler: (agent: Agent, error: Error) => void): void
  on(event: 'agent:waiting', handler: (agent: Agent, reason: string) => void): void
}
```

#### Task Scheduler

- Picks highest-priority unassigned tasks
- Respects task dependencies (if defined)
- Handles stale lock recovery (default: 30 min timeout)

```typescript
interface TaskScheduler {
  // Scheduling
  getNextTask(projectId: string, prdId: string): Promise<Task | null>
  getAvailableTasks(projectId: string, prdId: string): Promise<Task[]>
  
  // Lock management
  isTaskLocked(task: Task): boolean
  isLockStale(task: Task, maxAgeMs?: number): boolean
  recoverStaleLocks(projectId: string): Promise<Task[]>
}
```

### 3. Git Sync Engine

#### Control Repo Sync

- Auto-commit on state changes
- Push to `main` branch
- Pull before agent starts work
- Conflict resolution: last-write-wins for most fields; merge for append-only logs

```typescript
interface ControlRepoSync {
  // Sync operations
  pull(): Promise<SyncResult>
  push(): Promise<SyncResult>
  sync(): Promise<SyncResult>  // pull + push
  
  // Auto-sync
  enableAutoSync(intervalMs: number): void
  disableAutoSync(): void
  
  // Conflict handling
  resolveConflicts(): Promise<ConflictResolution[]>
}
```

#### Target Repo Worktrees

- Creates worktree per task: `{repo}/.worktrees/wiggy-{task-id}`
- Agent works in worktree
- Pushes to feature branch: `wiggy/{task-id}`
- Creates PR via `gh` CLI

```typescript
interface WorktreeManager {
  // Worktree lifecycle
  create(projectPath: string, taskId: string): Promise<Worktree>
  get(projectPath: string, taskId: string): Promise<Worktree | null>
  delete(projectPath: string, taskId: string): Promise<void>
  
  // Git operations (within worktree)
  checkout(worktree: Worktree, branch: string): Promise<void>
  commit(worktree: Worktree, message: string): Promise<string>
  push(worktree: Worktree): Promise<void>
  createPR(worktree: Worktree, title: string, body: string): Promise<string>
}
```

### 4. Agent Runtime

Each agent is an OpenCode session running with a specific prompt and task context.

#### Initialization

```
1. Pull latest control repo
2. Claim task (set status=in_progress, lock with timestamp)
3. Push control repo
4. Create/enter worktree in target repo
5. Pull latest from main
6. Load task context (if resuming)
```

#### Execution Loop

```
while not complete and iterations < max:
    1. Read current task context
    2. Execute implementation prompt
    3. Commit changes (if any)
    4. Run lint + types + tests
    5. If failed: attempt fix (up to 3 retries)
    6. Append to progress.md
    7. Update context.json
    8. Check if task complete → emit <promise>COMPLETE</promise>
```

#### Completion

```
1. Run self-review prompt
2. Create PR with description
3. Update task status to pr_open
4. Push control repo
5. Clean up worktree (optional, configurable)
```

#### Escalation

If agent gets stuck (3+ failed attempts, unclear requirements):

```
1. Set task status to waiting_for_human
2. Add note explaining the blocker
3. Trigger desktop notification
4. Exit gracefully
```

## Data Flow

### Creating a New PRD

```
┌──────┐     ┌─────┐     ┌──────┐     ┌──────────┐
│ User │     │ TUI │     │ Core │     │ OpenCode │
└──┬───┘     └──┬──┘     └──┬───┘     └────┬─────┘
   │            │           │              │
   │ New PRD    │           │              │
   ├───────────►│           │              │
   │            │ Create    │              │
   │            │ draft PRD │              │
   │            ├──────────►│              │
   │            │           │ Start plan   │
   │            │           │ session      │
   │            │           ├─────────────►│
   │            │           │              │
   │◄───────────┼───────────┼──────────────┤
   │     Interactive planning (Plan Mode)  │
   ├───────────►│           │              │
   │            │           ├─────────────►│
   │◄───────────┼───────────┼──────────────┤
   │            │           │              │
   │ Approve    │           │              │
   ├───────────►│           │              │
   │            │ Finalize  │              │
   │            │ PRD       │              │
   │            ├──────────►│              │
   │            │           │ Break into   │
   │            │           │ tasks        │
   │            │           ├─────────────►│
   │            │           │◄─────────────┤
   │            │           │              │
   │            │ Show      │              │
   │            │ tasks     │              │
   │◄───────────┤           │              │
   │            │           │              │
```

### Running Agents

```
┌──────┐     ┌─────┐     ┌──────────┐     ┌───────────┐     ┌────────────┐
│ User │     │ TUI │     │ AgentPool│     │ OpenCode  │     │ Target Repo│
└──┬───┘     └──┬──┘     └────┬─────┘     └─────┬─────┘     └──────┬─────┘
   │            │             │                 │                  │
   │ Run PRD    │             │                 │                  │
   │ (n=3)      │             │                 │                  │
   ├───────────►│             │                 │                  │
   │            │ Start pool  │                 │                  │
   │            ├────────────►│                 │                  │
   │            │             │                 │                  │
   │            │             │ For each task (up to n):          │
   │            │             ├─────────────────┼──────────────────┤
   │            │             │                 │                  │
   │            │             │ Spawn session   │                  │
   │            │             ├────────────────►│                  │
   │            │             │                 │                  │
   │            │             │                 │ Create worktree  │
   │            │             │                 ├─────────────────►│
   │            │             │                 │                  │
   │            │             │                 │ [Execution Loop] │
   │            │             │                 │◄────────────────►│
   │            │             │                 │                  │
   │            │             │ Progress events │                  │
   │            │◄────────────┤                 │                  │
   │            │             │                 │                  │
   │ Live       │             │                 │ Create PR        │
   │ updates    │             │                 ├─────────────────►│
   │◄───────────┤             │                 │                  │
   │            │             │                 │                  │
```

## OpenCode Integration

Wiggy uses the OpenCode SDK for agent execution:

```typescript
import { createOpencode, createOpencodeClient } from '@opencode-ai/sdk'

// Option 1: Spawn a new server per agent
const { client, server } = await createOpencode({
  config: {
    // Uses user's configured model
  }
})

// Option 2: Connect to shared server (more efficient for multiple agents)
const client = createOpencodeClient({
  baseUrl: 'http://localhost:4096'
})

// Create session for task
const session = await client.session.create({
  body: { title: `Task: ${task.id}` }
})

// Send implementation prompt
const result = await client.session.prompt({
  path: { id: session.id },
  body: {
    parts: [{ type: 'text', text: implementationPrompt }]
  }
})

// Listen for events
const events = await client.event.subscribe()
for await (const event of events.stream) {
  if (event.type === 'message.completed') {
    // Check for COMPLETE marker
  }
}
```

## Notification System

```typescript
import notifier from 'node-notifier'

interface NotificationService {
  // Desktop notifications
  notify(title: string, message: string): void
  
  // Webhooks (optional, configured in config.json)
  sendWebhook(event: WiggyEvent): Promise<void>
}

// Usage
notificationService.notify(
  'Wiggy: Human Input Needed',
  `Task ${task.id} is waiting for your input`
)
```

## File Locking Strategy

To prevent race conditions when multiple agents access the same files:

```typescript
interface FileLock {
  taskId: string
  agentId: string
  lockedAt: string  // ISO timestamp
  expiresAt: string // ISO timestamp (lockedAt + 30min)
}

// Lock is embedded in task.json
interface Task {
  // ... other fields
  lock?: FileLock
}

// Claiming a task
async function claimTask(task: Task, agentId: string): Promise<boolean> {
  // 1. Pull latest from control repo
  await controlRepo.pull()
  
  // 2. Re-read task (might have changed)
  const freshTask = await stateManager.getTask(task.projectId, task.id)
  
  // 3. Check if locked
  if (freshTask.lock && !isLockStale(freshTask.lock)) {
    return false // Already claimed
  }
  
  // 4. Acquire lock
  freshTask.lock = {
    taskId: task.id,
    agentId,
    lockedAt: new Date().toISOString(),
    expiresAt: new Date(Date.now() + 30 * 60 * 1000).toISOString()
  }
  freshTask.status = 'in_progress'
  
  // 5. Save and push
  await stateManager.updateTask(task.projectId, task.id, freshTask)
  await controlRepo.push()
  
  return true
}
```

## Error Recovery

### Agent Crash Recovery

1. Scheduler periodically checks for stale locks
2. If lock is stale (>30min with no progress):
   - Read last context.json checkpoint
   - Release lock
   - Task becomes available for another agent
   - New agent resumes from checkpoint

### Git Conflict Recovery

1. Control repo conflicts:
   - For JSON state files: last-write-wins (most recent timestamp)
   - For progress.md: merge both versions (append-only)
   
2. Target repo conflicts:
   - Agent rebases worktree on latest main
   - If rebase fails, escalate to waiting_for_human

### Network Failure Recovery

1. All state changes are local-first
2. Sync failures are queued and retried
3. Agents can work offline; sync when network returns
