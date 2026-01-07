# Schemas

This document defines the JSON Schema for all Wiggy data types. These schemas are used for validation and provide a contract for the state files.

## Schema Files

All schemas are stored in `/schemas/` and referenced using `$ref`.

## Project Schema

**File**: `schemas/project.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://wiggy.dev/schemas/project.schema.json",
  "title": "Project",
  "description": "A target codebase managed by Wiggy",
  "type": "object",
  "required": ["id", "name", "path", "createdAt", "updatedAt"],
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^[a-z0-9-]+$",
      "description": "URL-safe slug identifier"
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100,
      "description": "Human-readable project name"
    },
    "description": {
      "type": "string",
      "maxLength": 500,
      "description": "Brief project description"
    },
    "path": {
      "type": "string",
      "description": "Absolute path to the project repository"
    },
    "remote": {
      "type": "string",
      "format": "uri",
      "description": "Git remote URL (e.g., git@github.com:user/repo.git)"
    },
    "defaultBranch": {
      "type": "string",
      "default": "main",
      "description": "Default branch to base worktrees on"
    },
    "config": {
      "$ref": "#/$defs/projectConfig"
    },
    "stats": {
      "$ref": "#/$defs/projectStats"
    },
    "createdAt": {
      "type": "string",
      "format": "date-time"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time"
    }
  },
  "$defs": {
    "projectConfig": {
      "type": "object",
      "description": "Project-specific configuration overrides",
      "properties": {
        "testCommand": {
          "type": "string",
          "default": "npm test",
          "description": "Command to run tests"
        },
        "lintCommand": {
          "type": "string",
          "default": "npm run lint",
          "description": "Command to run linter"
        },
        "typeCheckCommand": {
          "type": "string",
          "default": "npm run typecheck",
          "description": "Command to run type checker"
        },
        "buildCommand": {
          "type": "string",
          "default": "npm run build",
          "description": "Command to build the project"
        },
        "maxAgents": {
          "type": "integer",
          "minimum": 1,
          "maximum": 10,
          "default": 3,
          "description": "Maximum concurrent agents for this project"
        },
        "worktreeDir": {
          "type": "string",
          "default": ".worktrees",
          "description": "Directory for git worktrees (relative to project root)"
        }
      }
    },
    "projectStats": {
      "type": "object",
      "description": "Aggregate statistics",
      "properties": {
        "totalPRDs": {
          "type": "integer",
          "minimum": 0
        },
        "activePRDs": {
          "type": "integer",
          "minimum": 0
        },
        "completedPRDs": {
          "type": "integer",
          "minimum": 0
        },
        "totalTasks": {
          "type": "integer",
          "minimum": 0
        },
        "completedTasks": {
          "type": "integer",
          "minimum": 0
        }
      }
    }
  }
}
```

## PRD Schema

**File**: `schemas/prd.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://wiggy.dev/schemas/prd.schema.json",
  "title": "PRD",
  "description": "Product Requirements Document",
  "type": "object",
  "required": ["id", "projectId", "title", "status", "createdAt", "updatedAt"],
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^[a-z0-9-]+$",
      "description": "URL-safe slug identifier"
    },
    "projectId": {
      "type": "string",
      "description": "Reference to parent project"
    },
    "title": {
      "type": "string",
      "minLength": 1,
      "maxLength": 200,
      "description": "PRD title"
    },
    "description": {
      "type": "string",
      "description": "Detailed description of the feature/work"
    },
    "status": {
      "type": "string",
      "enum": ["draft", "planning", "active", "paused", "completed", "cancelled"],
      "description": "Current PRD status"
    },
    "priority": {
      "type": "string",
      "enum": ["critical", "high", "medium", "low"],
      "default": "medium"
    },
    "userStories": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/userStory"
      }
    },
    "successCriteria": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Measurable success criteria"
    },
    "technicalNotes": {
      "type": "string",
      "description": "Technical implementation notes"
    },
    "taskIds": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "References to child tasks"
    },
    "stats": {
      "$ref": "#/$defs/prdStats"
    },
    "createdAt": {
      "type": "string",
      "format": "date-time"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time"
    }
  },
  "$defs": {
    "userStory": {
      "type": "object",
      "required": ["id", "story", "status"],
      "properties": {
        "id": {
          "type": "string"
        },
        "story": {
          "type": "string",
          "description": "As a [user], I want [feature] so that [benefit]"
        },
        "acceptanceCriteria": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "status": {
          "type": "string",
          "enum": ["pending", "in_progress", "completed"],
          "default": "pending"
        }
      }
    },
    "prdStats": {
      "type": "object",
      "properties": {
        "totalTasks": {
          "type": "integer",
          "minimum": 0
        },
        "tasksByStatus": {
          "type": "object",
          "additionalProperties": {
            "type": "integer"
          }
        },
        "linkedPRs": {
          "type": "array",
          "items": {
            "$ref": "#/$defs/linkedPR"
          }
        }
      }
    },
    "linkedPR": {
      "type": "object",
      "properties": {
        "url": {
          "type": "string",
          "format": "uri"
        },
        "number": {
          "type": "integer"
        },
        "status": {
          "type": "string",
          "enum": ["open", "merged", "closed"]
        },
        "taskId": {
          "type": "string"
        }
      }
    }
  }
}
```

## Task Schema

**File**: `schemas/task.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://wiggy.dev/schemas/task.schema.json",
  "title": "Task",
  "description": "An atomic unit of work",
  "type": "object",
  "required": ["id", "projectId", "prdId", "title", "status", "createdAt", "updatedAt"],
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^[a-z0-9-]+$",
      "description": "URL-safe identifier"
    },
    "projectId": {
      "type": "string",
      "description": "Reference to parent project"
    },
    "prdId": {
      "type": "string",
      "description": "Reference to parent PRD"
    },
    "title": {
      "type": "string",
      "minLength": 1,
      "maxLength": 200,
      "description": "Brief task title"
    },
    "description": {
      "type": "string",
      "description": "Detailed task description"
    },
    "status": {
      "type": "string",
      "enum": [
        "backlog",
        "in_progress",
        "implemented",
        "tested",
        "pr_open",
        "pr_merged",
        "waiting_for_human",
        "cancelled"
      ],
      "default": "backlog"
    },
    "priority": {
      "type": "integer",
      "minimum": 0,
      "description": "Lower number = higher priority (0 is highest)"
    },
    "estimatedLines": {
      "type": "integer",
      "minimum": 1,
      "maximum": 500,
      "description": "Estimated lines of code"
    },
    "acceptanceCriteria": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "dependencies": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Task IDs this task depends on"
    },
    "lock": {
      "$ref": "#/$defs/taskLock"
    },
    "worktree": {
      "$ref": "#/$defs/worktreeInfo"
    },
    "pr": {
      "$ref": "#/$defs/prInfo"
    },
    "waitingReason": {
      "type": "string",
      "description": "Reason for waiting_for_human status"
    },
    "sessionId": {
      "type": "string",
      "description": "Current or last agent session working on this task"
    },
    "attempts": {
      "type": "integer",
      "minimum": 0,
      "default": 0,
      "description": "Number of implementation attempts"
    },
    "createdAt": {
      "type": "string",
      "format": "date-time"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time"
    },
    "statusHistory": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/statusChange"
      }
    }
  },
  "$defs": {
    "taskLock": {
      "type": "object",
      "required": ["agentId", "lockedAt"],
      "properties": {
        "agentId": {
          "type": "string"
        },
        "sessionId": {
          "type": "string"
        },
        "lockedAt": {
          "type": "string",
          "format": "date-time"
        },
        "expiresAt": {
          "type": "string",
          "format": "date-time"
        }
      }
    },
    "worktreeInfo": {
      "type": "object",
      "properties": {
        "path": {
          "type": "string",
          "description": "Absolute path to worktree"
        },
        "branch": {
          "type": "string",
          "description": "Git branch name"
        },
        "createdAt": {
          "type": "string",
          "format": "date-time"
        }
      }
    },
    "prInfo": {
      "type": "object",
      "properties": {
        "url": {
          "type": "string",
          "format": "uri"
        },
        "number": {
          "type": "integer"
        },
        "status": {
          "type": "string",
          "enum": ["open", "merged", "closed"]
        },
        "createdAt": {
          "type": "string",
          "format": "date-time"
        },
        "mergedAt": {
          "type": "string",
          "format": "date-time"
        }
      }
    },
    "statusChange": {
      "type": "object",
      "required": ["from", "to", "changedAt"],
      "properties": {
        "from": {
          "type": "string"
        },
        "to": {
          "type": "string"
        },
        "changedAt": {
          "type": "string",
          "format": "date-time"
        },
        "changedBy": {
          "type": "string",
          "description": "Agent ID or 'human'"
        },
        "reason": {
          "type": "string"
        }
      }
    }
  }
}
```

## Session Schema

**File**: `schemas/session.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://wiggy.dev/schemas/session.schema.json",
  "title": "Session",
  "description": "An agent execution session",
  "type": "object",
  "required": ["id", "projectId", "taskId", "status", "createdAt"],
  "properties": {
    "id": {
      "type": "string",
      "description": "Unique session identifier"
    },
    "projectId": {
      "type": "string"
    },
    "prdId": {
      "type": "string"
    },
    "taskId": {
      "type": "string"
    },
    "agentId": {
      "type": "string",
      "description": "Identifier for the agent instance"
    },
    "opencodeSessionId": {
      "type": "string",
      "description": "OpenCode session ID"
    },
    "status": {
      "type": "string",
      "enum": ["initializing", "running", "paused", "completed", "failed", "waiting"],
      "default": "initializing"
    },
    "config": {
      "$ref": "#/$defs/sessionConfig"
    },
    "metrics": {
      "$ref": "#/$defs/sessionMetrics"
    },
    "checkpoint": {
      "$ref": "#/$defs/checkpoint"
    },
    "error": {
      "type": "string",
      "description": "Error message if status is 'failed'"
    },
    "createdAt": {
      "type": "string",
      "format": "date-time"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time"
    },
    "completedAt": {
      "type": "string",
      "format": "date-time"
    }
  },
  "$defs": {
    "sessionConfig": {
      "type": "object",
      "properties": {
        "maxIterations": {
          "type": "integer",
          "minimum": 1,
          "default": 10
        },
        "maxRetries": {
          "type": "integer",
          "minimum": 0,
          "default": 3
        },
        "timeoutMs": {
          "type": "integer",
          "minimum": 60000,
          "default": 1800000,
          "description": "Session timeout in milliseconds (default 30 min)"
        }
      }
    },
    "sessionMetrics": {
      "type": "object",
      "properties": {
        "iterations": {
          "type": "integer",
          "minimum": 0
        },
        "commits": {
          "type": "integer",
          "minimum": 0
        },
        "linesAdded": {
          "type": "integer",
          "minimum": 0
        },
        "linesDeleted": {
          "type": "integer",
          "minimum": 0
        },
        "testsRun": {
          "type": "integer",
          "minimum": 0
        },
        "testsPassed": {
          "type": "integer",
          "minimum": 0
        },
        "testsFailed": {
          "type": "integer",
          "minimum": 0
        },
        "tokensUsed": {
          "type": "integer",
          "minimum": 0
        },
        "durationMs": {
          "type": "integer",
          "minimum": 0
        }
      }
    },
    "checkpoint": {
      "type": "object",
      "description": "Resumable state for continuity",
      "properties": {
        "lastCommit": {
          "type": "string",
          "description": "Git commit SHA"
        },
        "currentStep": {
          "type": "string"
        },
        "completedSteps": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "remainingWork": {
          "type": "string",
          "description": "Description of remaining work"
        },
        "savedAt": {
          "type": "string",
          "format": "date-time"
        }
      }
    }
  }
}
```

## Context Schema

**File**: `schemas/context.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://wiggy.dev/schemas/context.schema.json",
  "title": "Context",
  "description": "Structured checkpoint for agent continuity",
  "type": "object",
  "required": ["sessionId", "taskId", "savedAt"],
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "taskId": {
      "type": "string"
    },
    "phase": {
      "type": "string",
      "enum": ["understanding", "planning", "implementing", "testing", "reviewing", "complete"],
      "description": "Current phase of work"
    },
    "understanding": {
      "type": "object",
      "properties": {
        "requirements": {
          "type": "array",
          "items": { "type": "string" }
        },
        "relatedFiles": {
          "type": "array",
          "items": { "type": "string" }
        },
        "existingPatterns": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
    },
    "plan": {
      "type": "object",
      "properties": {
        "approach": {
          "type": "string"
        },
        "steps": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "description": { "type": "string" },
              "files": {
                "type": "array",
                "items": { "type": "string" }
              },
              "completed": { "type": "boolean" }
            }
          }
        }
      }
    },
    "implementation": {
      "type": "object",
      "properties": {
        "filesCreated": {
          "type": "array",
          "items": { "type": "string" }
        },
        "filesModified": {
          "type": "array",
          "items": { "type": "string" }
        },
        "commits": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "sha": { "type": "string" },
              "message": { "type": "string" },
              "timestamp": { "type": "string", "format": "date-time" }
            }
          }
        }
      }
    },
    "testing": {
      "type": "object",
      "properties": {
        "testsWritten": {
          "type": "array",
          "items": { "type": "string" }
        },
        "lastTestRun": {
          "type": "object",
          "properties": {
            "passed": { "type": "boolean" },
            "summary": { "type": "string" },
            "failureDetails": { "type": "string" }
          }
        }
      }
    },
    "blockers": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "description": { "type": "string" },
          "attemptedSolutions": {
            "type": "array",
            "items": { "type": "string" }
          },
          "resolved": { "type": "boolean" }
        }
      }
    },
    "savedAt": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

## Config Schema

**File**: `schemas/config.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://wiggy.dev/schemas/config.schema.json",
  "title": "WiggyConfig",
  "description": "Global Wiggy configuration",
  "type": "object",
  "properties": {
    "controlRepo": {
      "type": "object",
      "properties": {
        "remote": {
          "type": "string",
          "format": "uri",
          "description": "Git remote for the control repo"
        },
        "autoSync": {
          "type": "boolean",
          "default": true
        },
        "syncIntervalMs": {
          "type": "integer",
          "minimum": 5000,
          "default": 30000
        }
      }
    },
    "agents": {
      "type": "object",
      "properties": {
        "maxConcurrent": {
          "type": "integer",
          "minimum": 1,
          "maximum": 20,
          "default": 5
        },
        "defaultIterations": {
          "type": "integer",
          "minimum": 1,
          "default": 10
        },
        "lockTimeoutMs": {
          "type": "integer",
          "minimum": 60000,
          "default": 1800000,
          "description": "Lock timeout in ms (default 30 min)"
        },
        "staleLockThresholdMs": {
          "type": "integer",
          "minimum": 60000,
          "default": 1800000,
          "description": "When to consider a lock stale"
        }
      }
    },
    "notifications": {
      "type": "object",
      "properties": {
        "desktop": {
          "type": "boolean",
          "default": true
        },
        "webhooks": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "url": { "type": "string", "format": "uri" },
              "events": {
                "type": "array",
                "items": {
                  "type": "string",
                  "enum": ["task:waiting", "task:completed", "prd:completed", "agent:error"]
                }
              }
            }
          }
        }
      }
    },
    "opencode": {
      "type": "object",
      "description": "OpenCode-specific settings",
      "properties": {
        "serverUrl": {
          "type": "string",
          "format": "uri",
          "description": "URL of shared OpenCode server (if using)"
        },
        "useSharedServer": {
          "type": "boolean",
          "default": false
        }
      }
    },
    "prompts": {
      "type": "object",
      "description": "Paths to prompt templates (relative to control repo root)",
      "properties": {
        "requirements": { "type": "string", "default": "prompts/planning/requirements.md" },
        "userStories": { "type": "string", "default": "prompts/planning/user-stories.md" },
        "taskBreakdown": { "type": "string", "default": "prompts/planning/task-breakdown.md" },
        "implement": { "type": "string", "default": "prompts/execution/implement.md" },
        "test": { "type": "string", "default": "prompts/execution/test.md" },
        "pr": { "type": "string", "default": "prompts/execution/pr.md" },
        "selfReview": { "type": "string", "default": "prompts/review/self-review.md" }
      }
    }
  }
}
```

## TypeScript Types

These schemas are converted to TypeScript types for use in the codebase:

```typescript
// Generated from schemas - see src/types/index.ts

export type ProjectStatus = 'active' | 'archived'

export type PRDStatus = 'draft' | 'planning' | 'active' | 'paused' | 'completed' | 'cancelled'

export type TaskStatus =
  | 'backlog'
  | 'in_progress'
  | 'implemented'
  | 'tested'
  | 'pr_open'
  | 'pr_merged'
  | 'waiting_for_human'
  | 'cancelled'

export type SessionStatus = 'initializing' | 'running' | 'paused' | 'completed' | 'failed' | 'waiting'

export type Priority = 'critical' | 'high' | 'medium' | 'low'

export interface Project { /* ... */ }
export interface PRD { /* ... */ }
export interface Task { /* ... */ }
export interface Session { /* ... */ }
export interface Context { /* ... */ }
export interface WiggyConfig { /* ... */ }
```

## Validation

Use `ajv` for runtime validation:

```typescript
import Ajv from 'ajv'
import addFormats from 'ajv-formats'

const ajv = new Ajv({ allErrors: true })
addFormats(ajv)

// Load schemas
import projectSchema from '../schemas/project.schema.json'
import prdSchema from '../schemas/prd.schema.json'
import taskSchema from '../schemas/task.schema.json'
import sessionSchema from '../schemas/session.schema.json'

// Compile validators
export const validateProject = ajv.compile(projectSchema)
export const validatePRD = ajv.compile(prdSchema)
export const validateTask = ajv.compile(taskSchema)
export const validateSession = ajv.compile(sessionSchema)

// Usage
function saveTask(task: Task): void {
  if (!validateTask(task)) {
    throw new Error(`Invalid task: ${ajv.errorsText(validateTask.errors)}`)
  }
  // ... save to file
}
```
