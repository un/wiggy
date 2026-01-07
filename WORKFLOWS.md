# Workflows

This document describes the step-by-step workflows for common Wiggy operations.

## 1. Initial Setup

### First-time Setup

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         INITIAL SETUP                                    │
└─────────────────────────────────────────────────────────────────────────┘

1. Install Wiggy
   └─> npm install -g wiggy

2. Initialize control repo
   └─> mkdir ~/wiggy-control && cd ~/wiggy-control
   └─> wiggy init
       ├─> Creates directory structure
       ├─> Initializes git repo
       ├─> Creates config.json
       └─> Creates schema files

3. Connect to GitHub (optional but recommended)
   └─> git remote add origin git@github.com:you/wiggy-control.git
   └─> git push -u origin main
   └─> wiggy config set controlRepo.remote "git@github.com:you/wiggy-control.git"

4. Launch Wiggy
   └─> wiggy
```

### Directory Structure After Init

```
~/wiggy-control/
├── config.json
├── schemas/
│   ├── project.schema.json
│   ├── prd.schema.json
│   ├── task.schema.json
│   └── session.schema.json
├── prompts/
│   ├── planning/
│   │   ├── requirements.md
│   │   ├── user-stories.md
│   │   └── task-breakdown.md
│   ├── execution/
│   │   ├── implement.md
│   │   ├── test.md
│   │   └── pr.md
│   └── review/
│       └── self-review.md
└── projects/
    └── .gitkeep
```

## 2. Adding a New Project

### Via TUI

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ADD PROJECT (TUI)                                │
└─────────────────────────────────────────────────────────────────────────┘

1. Launch wiggy
   └─> wiggy

2. Press 'n' then 'p' (New Project)
   └─> Opens project creation form

3. Fill in details:
   ┌─ New Project ─────────────────────────────────────────┐
   │                                                        │
   │  Name: My Awesome App                                  │
   │  Path: /Users/me/code/my-awesome-app                   │
   │  Remote: git@github.com:me/my-awesome-app.git          │
   │  Default Branch: main                                  │
   │                                                        │
   │  [Create]  [Cancel]                                    │
   └────────────────────────────────────────────────────────┘

4. Wiggy validates:
   ├─> Path exists and is a git repo
   ├─> Remote is accessible (if provided)
   └─> Creates project.json

5. Project appears in list
   └─> Auto-synced to control repo
```

### Via CLI

```bash
wiggy new project

# Interactive prompts:
# ? Project name: My Awesome App
# ? Path to repository: /Users/me/code/my-awesome-app
# ? Git remote (optional): git@github.com:me/my-awesome-app.git
# ? Default branch: main
#
# ✓ Created project: my-awesome-app
```

## 3. Creating a PRD (Planning Phase)

This is the most important workflow - extracting requirements from your brain.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CREATE PRD                                       │
└─────────────────────────────────────────────────────────────────────────┘

1. Select project and press 'n' then 'r' (New PRD)
   OR
   └─> wiggy new prd my-awesome-app

2. Enter PRD title
   └─> "User Authentication System"

3. OpenCode launches in PLAN MODE
   ┌─────────────────────────────────────────────────────────────────────┐
   │  OpenCode (Plan Mode)                                                │
   │                                                                      │
   │  I'll help you define requirements for User Authentication System.  │
   │                                                                      │
   │  Let's start with the basics:                                        │
   │  1. What authentication methods do you need?                         │
   │     (email/password, OAuth, magic links, etc.)                       │
   │                                                                      │
   │  > We need email/password for now, but want to add OAuth later.      │
   │    Users should be able to reset their passwords.                    │
   │                                                                      │
   │  Great. A few follow-up questions:                                   │
   │  2. Do you need email verification for new accounts?                 │
   │  3. Any specific password requirements?                              │
   │  4. Session duration? Remember me functionality?                     │
   │                                                                      │
   │  > Yes to email verification. Password: min 8 chars, 1 number.       │
   │    Sessions expire in 24h, remember me extends to 30 days.           │
   │                                                                      │
   │  [... conversation continues ...]                                    │
   │                                                                      │
   │  Based on our discussion, here's my understanding:                   │
   │                                                                      │
   │  ## Requirements Summary                                             │
   │  - Email/password authentication                                     │
   │  - Email verification for new accounts                               │
   │  - Password reset via email                                          │
   │  - Password rules: min 8 chars, at least 1 number                    │
   │  - JWT-based sessions (24h default, 30 days with remember me)        │
   │  - Rate limiting on auth endpoints                                   │
   │                                                                      │
   │  Does this look correct? (yes/no/edit)                               │
   │                                                                      │
   │  > yes                                                               │
   │                                                                      │
   └─────────────────────────────────────────────────────────────────────┘

4. OpenCode generates user stories
   ┌─────────────────────────────────────────────────────────────────────┐
   │                                                                      │
   │  ## User Stories                                                     │
   │                                                                      │
   │  ### US-001: User Registration                                       │
   │  As a new user, I want to register with email and password           │
   │  so that I can create an account.                                    │
   │                                                                      │
   │  Acceptance Criteria:                                                │
   │  - Email must be valid format                                        │
   │  - Password meets complexity requirements                            │
   │  - Verification email sent after registration                        │
   │  - Cannot login until email verified                                 │
   │                                                                      │
   │  ### US-002: User Login                                              │
   │  As a registered user, I want to login with my credentials           │
   │  so that I can access my account.                                    │
   │                                                                      │
   │  [... more user stories ...]                                         │
   │                                                                      │
   │  Review these user stories. Add, remove, or modify? (done/edit)      │
   │                                                                      │
   │  > done                                                              │
   │                                                                      │
   └─────────────────────────────────────────────────────────────────────┘

5. OpenCode breaks down into tasks
   ┌─────────────────────────────────────────────────────────────────────┐
   │                                                                      │
   │  ## Tasks                                                            │
   │                                                                      │
   │  1. task-001: Create User database model                             │
   │     - ~80 lines                                                      │
   │     - Acceptance: User model with email, password hash, verified     │
   │                                                                      │
   │  2. task-002: Implement password hashing utility                     │
   │     - ~50 lines                                                      │
   │     - Acceptance: bcrypt hash/verify functions                       │
   │                                                                      │
   │  3. task-003: Create registration endpoint                           │
   │     - ~120 lines                                                     │
   │     - Depends on: task-001, task-002                                 │
   │                                                                      │
   │  [... 12 more tasks ...]                                             │
   │                                                                      │
   │  15 tasks total, estimated ~1,200 lines.                             │
   │  Review and confirm? (confirm/edit)                                  │
   │                                                                      │
   │  > confirm                                                           │
   │                                                                      │
   └─────────────────────────────────────────────────────────────────────┘

6. PRD created and saved
   ├─> projects/my-awesome-app/prds/user-auth.json
   ├─> projects/my-awesome-app/tasks/task-001.json
   ├─> projects/my-awesome-app/tasks/task-002.json
   ├─> ... (all tasks)
   └─> Auto-synced to control repo

7. Return to TUI
   └─> PRD visible in project view, ready to run
```

## 4. Running Agents

### Starting Agents on a PRD

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         RUN AGENTS                                       │
└─────────────────────────────────────────────────────────────────────────┘

1. Navigate to PRD in TUI
   └─> Select project → Select PRD

2. Press 'r' (Run agents)
   ┌─ Run Agents ──────────────────────────────────────────┐
   │                                                        │
   │  PRD: user-auth                                        │
   │  Available tasks: 12 (backlog)                         │
   │                                                        │
   │  Number of agents: [3]                                 │
   │  Iterations per agent: [10]                            │
   │                                                        │
   │  [Start]  [Cancel]                                     │
   └────────────────────────────────────────────────────────┘

3. Agents start
   └─> For each agent (up to N):
       ├─> Sync control repo (pull)
       ├─> Find available task (backlog, no lock)
       ├─> Claim task (set in_progress, add lock)
       ├─> Sync control repo (push)
       ├─> Create worktree in target repo
       └─> Start OpenCode session

4. Monitor in TUI
   ┌─ Active Agents ─────────────────────────────────────────────────┐
   │                                                                  │
   │  agent-a1b2  task-001  Create User model    implementing  1/10  │
   │  agent-c3d4  task-002  Password hashing     implementing  1/10  │
   │  agent-e5f6  task-003  Registration endpoint  waiting    0/10   │
   │              ↳ waiting for task-001, task-002                   │
   │                                                                  │
   └──────────────────────────────────────────────────────────────────┘
```

### Agent Execution Detail

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    AGENT EXECUTION (Internal)                            │
└─────────────────────────────────────────────────────────────────────────┘

For each iteration:

1. Load context
   ├─> Read task.json
   ├─> Read context.json (if resuming)
   └─> Read progress.md (last entries)

2. Run implementation prompt
   └─> OpenCode executes with full context

3. After OpenCode response:
   ├─> Parse any file changes
   ├─> Commit changes (if any)
   │   └─> "wiggy(task-001): <description>"
   ├─> Run CI checks:
   │   ├─> npm run lint
   │   ├─> npm run typecheck
   │   └─> npm test
   ├─> If checks fail:
   │   ├─> Attempt fix (up to 3 retries)
   │   └─> If still failing, escalate
   └─> Update context.json

4. Append to progress.md
   └─> "[14:32:15] Implemented user model with email, passwordHash fields.
        Committed: abc123. Tests passing."

5. Check for completion
   ├─> If response contains <promise>COMPLETE</promise>:
   │   ├─> Run self-review prompt
   │   ├─> Create PR
   │   ├─> Update task status to pr_open
   │   └─> Exit loop
   ├─> If response contains <promise>WAITING</promise>:
   │   ├─> Update task status to waiting_for_human
   │   ├─> Trigger notification
   │   └─> Exit loop
   └─> Otherwise, continue to next iteration

6. End of loop
   ├─> If max iterations reached without completion:
   │   ├─> Save context (for resumption)
   │   ├─> Release lock
   │   └─> Log "Max iterations reached"
   └─> Clean up
```

## 5. Handling Waiting for Human

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    WAITING FOR HUMAN                                     │
└─────────────────────────────────────────────────────────────────────────┘

1. Agent encounters blocker
   ├─> After 3 failed attempts to fix an issue
   ├─> OR unclear/contradictory requirements
   └─> Agent emits <promise>WAITING</promise>

2. Task updated
   ├─> Status: waiting_for_human
   ├─> waitingReason: "Cannot determine correct API response format.
   │                   Requirements say 'return user object' but existing
   │                   endpoints return {data: user}. Need clarification."
   └─> Synced to control repo

3. Notification sent
   └─> Desktop notification appears

4. In TUI
   ┌─ Notifications ─────────────────────────────────────────────────┐
   │                                                                  │
   │  ⚠ task-003 waiting_for_human                                   │
   │    "Cannot determine correct API response format..."            │
   │                                                                  │
   │  Press Enter to view, 'r' to resolve                            │
   └──────────────────────────────────────────────────────────────────┘

5. Human reviews and resolves
   ├─> Press 'r' on notification
   ├─> Options:
   │   ├─> Add clarification (updates task notes)
   │   ├─> Edit task requirements
   │   └─> Mark as resolved (returns to backlog)
   └─> Task can be picked up by agent again

6. Agent resumes (on next run)
   └─> Reads updated task + context
   └─> Continues from where it left off
```

## 6. Agent Continuity (Resuming Work)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    AGENT CONTINUITY                                      │
└─────────────────────────────────────────────────────────────────────────┘

Scenario: Agent crashed / session ended before completing task

1. Task state on crash:
   ├─> Status: in_progress
   ├─> Lock: { agentId: "agent-a1b2", lockedAt: "2024-01-15T14:30:00Z" }
   ├─> context.json: { phase: "implementing", ... }
   └─> progress.md: [...entries up to crash point]

2. Another agent starts (or same agent restarts)
   ├─> Finds task with stale lock (>30 min old)
   ├─> Claims task (overwrites lock)
   └─> Checks for existing context

3. Context loading
   ├─> Read context.json
   │   {
   │     "phase": "implementing",
   │     "plan": { "steps": [...] },
   │     "implementation": {
   │       "filesModified": ["src/auth/user.ts"],
   │       "commits": [{ "sha": "abc123", "message": "..." }]
   │     }
   │   }
   └─> Read progress.md for narrative context

4. Resume prompt includes:
   ├─> Original task description
   ├─> Context summary (what was done)
   ├─> Last progress entries
   └─> Instruction: "Continue from where previous agent stopped"

5. Agent continues
   └─> Picks up from last known state
```

## 7. PR Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PR WORKFLOW                                           │
└─────────────────────────────────────────────────────────────────────────┘

1. Agent completes implementation
   ├─> All acceptance criteria met
   ├─> Tests written and passing
   └─> Self-review passed

2. Agent creates PR
   ├─> Pushes branch: wiggy/task-001
   ├─> Creates PR via gh CLI
   │   gh pr create \
   │     --title "[task-001] Create User database model" \
   │     --body "## Summary\n..."
   └─> Gets PR URL

3. Task updated
   ├─> Status: pr_open
   ├─> pr: { url: "https://github.com/...", number: 42, status: "open" }
   └─> Synced to control repo

4. In TUI
   ┌─ PRD: user-auth ────────────────────────────────────────────────┐
   │                                                                  │
   │  ✓ task-001  Create User model              pr_open   PR #42    │
   │                                                                  │
   │  Press Enter to view PR                                          │
   └──────────────────────────────────────────────────────────────────┘

5. Human reviews PR
   ├─> Reviews on GitHub
   ├─> May request changes (agent can be re-run)
   └─> Merges when satisfied

6. After merge
   ├─> Wiggy detects PR merged (via periodic check or webhook)
   ├─> Updates task status: pr_merged
   ├─> Cleans up worktree (optional)
   └─> Updates PRD progress
```

## 8. Control Repo Sync

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SYNC WORKFLOW                                         │
└─────────────────────────────────────────────────────────────────────────┘

Automatic sync (every 30s by default):

1. Pull
   ├─> git fetch origin main
   ├─> git merge origin/main
   └─> Handle conflicts (see below)

2. Push
   ├─> git add -A
   ├─> git commit -m "wiggy: sync $(date)"
   └─> git push origin main

Conflict resolution:

1. JSON state files (project.json, task.json, etc.)
   └─> Last-write-wins based on updatedAt timestamp

2. progress.md (append-only)
   └─> Merge both versions (concat unique entries)

3. context.json
   └─> Prefer version with later savedAt

Manual sync:
   └─> Press 's' in TUI or run `wiggy sync`
```

## 9. Multi-Machine Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MULTI-MACHINE SETUP                                   │
└─────────────────────────────────────────────────────────────────────────┘

Machine A (your laptop):
1. Clone control repo
   └─> git clone git@github.com:you/wiggy-control.git

2. Clone target project
   └─> git clone git@github.com:you/my-app.git

3. Run wiggy
   └─> cd wiggy-control && wiggy

Machine B (remote server):
1. Clone same repos
2. Run wiggy agents
   └─> wiggy run prd-auth -n 5 --background

Both machines:
├─> Auto-sync to GitHub
├─> See each other's agents
├─> Task locks prevent conflicts
└─> Progress visible everywhere
```

## 10. Complete Example Session

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FULL SESSION EXAMPLE                                  │
└─────────────────────────────────────────────────────────────────────────┘

$ wiggy

┌─ WIGGY ─────────────────────────────────────────────── Synced │ 0 Active ┐
│                                                                          │
│  Welcome to Wiggy! No projects yet.                                      │
│  Press 'n' to add your first project.                                    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

> Press 'n' then 'p'

┌─ New Project ─────────────────────────────────────────────────────────────┐
│  Name: shopping-cart-api                                                  │
│  Path: /Users/me/code/shopping-cart-api                                   │
│  Remote: git@github.com:me/shopping-cart-api.git                          │
│  [Create]                                                                 │
└───────────────────────────────────────────────────────────────────────────┘

✓ Created project: shopping-cart-api

> Press Enter to select project, then 'n' then 'r'

Opening OpenCode for PRD planning...

═══════════════════════════════════════════════════════════════════════════

OpenCode (Plan Mode)

Let's define your new feature. What would you like to build?

> I want to add a wishlist feature. Users should be able to save products
  they're interested in and move them to cart later.

Great! Let me understand the requirements better...
[... interactive planning session ...]

PRD created: wishlist-feature
Tasks created: 8 tasks

═══════════════════════════════════════════════════════════════════════════

> Press 'r' to run agents

┌─ Run Agents ───────────────────────────────────────────────────────────────┐
│  PRD: wishlist-feature                                                     │
│  Tasks: 8 (backlog)                                                        │
│  Agents: [3]    Iterations: [10]                                           │
│  [Start]                                                                   │
└────────────────────────────────────────────────────────────────────────────┘

Starting 3 agents...

┌─ WIGGY ─────────────────────────────────────────────── Synced │ 3 Active ┐
│                                                                          │
│  ┌─ Active Agents ────────────────────────────────────────────────────┐ │
│  │  agent-x1  task-001  Create Wishlist model      implementing  2/10 │ │
│  │  agent-y2  task-002  Add wishlist endpoints     implementing  1/10 │ │
│  │  agent-z3  task-003  Wishlist UI component      implementing  1/10 │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─ PRD Progress ─────────────────────────────────────────────────────┐ │
│  │  wishlist-feature  ██░░░░░░░░  3/8 in progress                     │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

[... time passes ...]

┌─ WIGGY ─────────────────────────────────────────────── Synced │ 2 Active ┐
│                                                                          │
│  ┌─ Active Agents ────────────────────────────────────────────────────┐ │
│  │  agent-y2  task-005  Wishlist to cart           testing       8/10 │ │
│  │  agent-z3  task-006  Wishlist persistence       implementing  5/10 │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─ PRD Progress ─────────────────────────────────────────────────────┐ │
│  │  wishlist-feature  ██████░░░░  5/8 complete                        │ │
│  │    ✓ task-001 (PR #51 merged)                                      │ │
│  │    ✓ task-002 (PR #52 merged)                                      │ │
│  │    ✓ task-003 (PR #53 merged)                                      │ │
│  │    ✓ task-004 (PR #54 merged)                                      │ │
│  │    ○ task-005 (PR #55 open - ready for review)                     │ │
│  │    ● task-006 (in progress)                                        │ │
│  │    ○ task-007 (backlog)                                            │ │
│  │    ○ task-008 (backlog)                                            │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─ Notifications ────────────────────────────────────────────────────┐ │
│  │  ✓ PR #51-54 merged                                                │ │
│  │  → PR #55 ready for review                                         │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

[... all tasks complete ...]

┌─ WIGGY ─────────────────────────────────────────────── Synced │ 0 Active ┐
│                                                                          │
│  ┌─ PRD Progress ─────────────────────────────────────────────────────┐ │
│  │  wishlist-feature  ██████████  8/8 complete                        │ │
│  │    All PRs merged!                                                 │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌─ Notifications ────────────────────────────────────────────────────┐ │
│  │  🎉 PRD wishlist-feature completed!                                │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

> Press 'q' to quit

Thanks for using Wiggy!
```
