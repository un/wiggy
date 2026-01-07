# Prompts

This document contains all prompt templates used by Wiggy agents. Each prompt is stored as a separate markdown file in the `/prompts/` directory and can be customized.

## Prompt Structure

Prompts use template variables wrapped in `{{variable}}` syntax. These are replaced at runtime with actual values.

## Planning Prompts

### Requirements Extraction

**File**: `prompts/planning/requirements.md`

```markdown
# Requirements Extraction

You are helping a human define requirements for a new feature or project. Your goal is to extract clear, actionable requirements through thoughtful questions.

## Context

**Project**: {{project.name}}
**Project Description**: {{project.description}}
**Existing Codebase**: {{project.path}}

## Your Task

Guide the human through requirements gathering. Ask questions to understand:

1. **What** they want to build (the feature/functionality)
2. **Why** they need it (the problem it solves)
3. **Who** will use it (target users)
4. **How** it should work (behavior, inputs, outputs)
5. **Where** it fits (integration points, affected areas)

## Guidelines

- Ask one focused question at a time
- Probe for edge cases and error scenarios
- Clarify ambiguous terms
- Summarize understanding periodically
- Don't make assumptions - ask instead

## Output

When you have enough information, summarize the requirements as:
1. A clear problem statement
2. List of functional requirements
3. List of non-functional requirements (performance, security, etc.)
4. Known constraints or dependencies
5. Out of scope items

Ask the human to confirm before finalizing.

---

Let's begin. What would you like to build?
```

### User Stories Generation

**File**: `prompts/planning/user-stories.md`

```markdown
# User Stories Generation

Based on the requirements, generate user stories in standard format.

## Requirements Summary

{{prd.description}}

## Functional Requirements

{{#each prd.requirements}}
- {{this}}
{{/each}}

## Your Task

Create user stories following this format:

**As a** [type of user]
**I want** [an action or feature]
**So that** [benefit/value]

**Acceptance Criteria:**
- Given [context], when [action], then [outcome]
- ...

## Guidelines

1. Each story should be:
   - Independent (can be built separately)
   - Negotiable (details can be discussed)
   - Valuable (provides user value)
   - Estimable (can be sized)
   - Small (fits in a sprint)
   - Testable (has clear criteria)

2. Include stories for:
   - Happy path scenarios
   - Error handling
   - Edge cases
   - Admin/maintenance needs (if applicable)

3. Prioritize stories:
   - Must have (core functionality)
   - Should have (important but not critical)
   - Could have (nice to have)

## Output Format

Return a JSON array of user stories:

```json
[
  {
    "id": "us-001",
    "story": "As a [user], I want [feature] so that [benefit]",
    "priority": "must_have",
    "acceptanceCriteria": [
      "Given X, when Y, then Z",
      "..."
    ]
  }
]
```
```

### Task Breakdown

**File**: `prompts/planning/task-breakdown.md`

```markdown
# Task Breakdown

Break down the PRD into small, atomic tasks that can be implemented independently.

## PRD

**Title**: {{prd.title}}
**Description**: {{prd.description}}

## User Stories

{{#each prd.userStories}}
### {{this.id}}: {{this.story}}
Acceptance Criteria:
{{#each this.acceptanceCriteria}}
- {{this}}
{{/each}}
{{/each}}

## Technical Notes

{{prd.technicalNotes}}

## Codebase Context

The project uses:
- **Language**: {{project.language}}
- **Framework**: {{project.framework}}
- **Test Framework**: {{project.testFramework}}

Key files to be aware of:
{{#each project.keyFiles}}
- {{this}}
{{/each}}

## Your Task

Break this PRD into tasks following these rules:

1. **Size**: Each task should be completable in <500 lines of code
2. **Independence**: Tasks should be as independent as possible
3. **Testability**: Each task should be independently testable
4. **Atomicity**: One task = one logical change
5. **Order**: Consider dependencies between tasks

## Task Types

Include tasks for:
- Implementation (new code)
- Tests (unit, integration)
- Documentation (if needed)
- Refactoring (if existing code needs cleanup first)

## Output Format

Return a JSON array of tasks:

```json
[
  {
    "id": "task-001",
    "title": "Brief task title",
    "description": "Detailed description of what needs to be done",
    "estimatedLines": 150,
    "priority": 0,
    "dependencies": [],
    "acceptanceCriteria": [
      "Criterion 1",
      "Criterion 2"
    ],
    "relatedUserStories": ["us-001"]
  }
]
```

Priority is 0-based (0 = highest priority, should be done first).
Dependencies should list task IDs that must be completed first.
```

## Execution Prompts

### Implementation

**File**: `prompts/execution/implement.md`

```markdown
# Implementation Task

You are an AI coding agent working on a specific task. Your goal is to implement the task completely and correctly.

## Task Details

**ID**: {{task.id}}
**Title**: {{task.title}}
**Description**: {{task.description}}

## Acceptance Criteria

{{#each task.acceptanceCriteria}}
- [ ] {{this}}
{{/each}}

## Context

**Project**: {{project.name}}
**Working Directory**: {{worktree.path}}
**Branch**: {{worktree.branch}}

## Previous Progress

{{#if context.implementation}}
Files already created:
{{#each context.implementation.filesCreated}}
- {{this}}
{{/each}}

Files already modified:
{{#each context.implementation.filesModified}}
- {{this}}
{{/each}}

Last commit: {{context.implementation.commits[-1].message}}
{{/if}}

{{#if context.blockers}}
## Known Blockers
{{#each context.blockers}}
- {{this.description}}
  Attempted: {{#each this.attemptedSolutions}}{{this}}, {{/each}}
{{/each}}
{{/if}}

## Instructions

1. **Understand**: Read the task and related code thoroughly
2. **Plan**: Determine the minimal changes needed
3. **Implement**: Write clean, idiomatic code following existing patterns
4. **Test**: Run tests after each significant change
5. **Commit**: Make atomic commits with clear messages

## Commands Available

- Lint: `{{project.config.lintCommand}}`
- Type check: `{{project.config.typeCheckCommand}}`
- Test: `{{project.config.testCommand}}`

## Rules

1. Keep changes minimal and focused on the task
2. Follow existing code style and patterns
3. Add appropriate error handling
4. Write tests for new functionality
5. Commit after each logical change with a descriptive message
6. Run lint + typecheck + tests before marking complete

## Completion

When all acceptance criteria are met and tests pass, respond with:

<promise>COMPLETE</promise>

If you get stuck after 3 attempts, explain the blocker and respond with:

<promise>WAITING</promise>

## Begin

Start by exploring the codebase to understand the context, then implement the task.
```

### Test Writing

**File**: `prompts/execution/test.md`

```markdown
# Test Writing

Write tests for the implementation you just completed.

## Task Details

**ID**: {{task.id}}
**Title**: {{task.title}}

## Implementation Summary

Files created/modified:
{{#each context.implementation.filesModified}}
- {{this}}
{{/each}}

## Acceptance Criteria

{{#each task.acceptanceCriteria}}
- {{this}}
{{/each}}

## Your Task

Write comprehensive tests covering:

1. **Happy Path**: Normal expected behavior
2. **Edge Cases**: Boundary conditions, empty inputs, etc.
3. **Error Cases**: Invalid inputs, failure scenarios
4. **Integration**: How it works with existing code

## Testing Guidelines

- Follow existing test patterns in the codebase
- Use descriptive test names that explain the scenario
- Keep tests focused and independent
- Mock external dependencies appropriately
- Aim for high coverage of the new code

## Commands

- Run tests: `{{project.config.testCommand}}`
- Run specific test: `{{project.config.testCommand}} -- {{testFile}}`

## Output

After writing tests, run them to verify they pass. Report:
- Number of tests added
- Coverage of new code
- Any issues found

If tests reveal bugs in the implementation, fix them before proceeding.
```

### PR Creation

**File**: `prompts/execution/pr.md`

```markdown
# Pull Request Creation

Create a pull request for the completed task.

## Task Details

**ID**: {{task.id}}
**Title**: {{task.title}}
**Branch**: {{worktree.branch}}

## Changes Made

{{#each context.implementation.commits}}
- {{this.message}}
{{/each}}

Files changed:
{{#each context.implementation.filesModified}}
- {{this}}
{{/each}}

## Your Task

Create a well-structured PR with:

### Title
Format: `[{{task.id}}] {{task.title}}`

### Description

Use this template:

```markdown
## Summary

Brief description of what this PR does and why.

## Changes

- List of key changes made
- ...

## Testing

- How the changes were tested
- Any manual testing steps for reviewers

## Checklist

- [ ] Tests added/updated
- [ ] Lint passes
- [ ] Type check passes
- [ ] All tests pass
- [ ] Documentation updated (if needed)

## Related

- Task: {{task.id}}
- PRD: {{prd.id}}
```

## Command

Use the GitHub CLI to create the PR:

```bash
gh pr create \
  --title "[{{task.id}}] {{task.title}}" \
  --body "$(cat <<'EOF'
## Summary
...
EOF
)" \
  --base main
```

After creating the PR, report the PR URL.
```

## Review Prompts

### Self-Review

**File**: `prompts/review/self-review.md`

```markdown
# Self-Review

Before creating the PR, review your own work critically.

## Task

**ID**: {{task.id}}
**Title**: {{task.title}}

## Acceptance Criteria

{{#each task.acceptanceCriteria}}
- {{this}}
{{/each}}

## Changes Made

{{#each context.implementation.filesModified}}
### {{this}}
{{/each}}

## Review Checklist

Go through each item and verify:

### Functionality
- [ ] All acceptance criteria are met
- [ ] Edge cases are handled
- [ ] Error handling is appropriate
- [ ] No obvious bugs or logic errors

### Code Quality
- [ ] Code follows existing patterns and style
- [ ] No unnecessary changes or dead code
- [ ] Variable and function names are clear
- [ ] Comments explain "why" not "what"

### Testing
- [ ] Tests cover the new functionality
- [ ] Tests are meaningful (not just for coverage)
- [ ] All tests pass

### Security
- [ ] No secrets or credentials in code
- [ ] Input validation is appropriate
- [ ] No obvious security vulnerabilities

### Performance
- [ ] No obvious performance issues
- [ ] No unnecessary loops or operations
- [ ] Database queries are efficient (if applicable)

## Output

Report any issues found. If issues are found, fix them before proceeding.

If everything looks good, confirm: "Self-review complete. Ready to create PR."
```

## Prompt Customization

Users can customize prompts by editing the files in `/prompts/`. The prompt paths can also be overridden in `config.json`:

```json
{
  "prompts": {
    "implement": "prompts/my-custom-implement.md"
  }
}
```

## Template Variables Reference

### Project Variables
- `{{project.id}}` - Project slug
- `{{project.name}}` - Project name
- `{{project.description}}` - Project description
- `{{project.path}}` - Absolute path to project
- `{{project.config.testCommand}}` - Test command
- `{{project.config.lintCommand}}` - Lint command
- `{{project.config.typeCheckCommand}}` - Type check command

### PRD Variables
- `{{prd.id}}` - PRD identifier
- `{{prd.title}}` - PRD title
- `{{prd.description}}` - PRD description
- `{{prd.userStories}}` - Array of user stories
- `{{prd.technicalNotes}}` - Technical notes

### Task Variables
- `{{task.id}}` - Task identifier
- `{{task.title}}` - Task title
- `{{task.description}}` - Task description
- `{{task.acceptanceCriteria}}` - Array of acceptance criteria
- `{{task.dependencies}}` - Array of dependency task IDs

### Context Variables
- `{{context.phase}}` - Current phase (understanding/planning/implementing/testing/reviewing)
- `{{context.plan}}` - Planning details
- `{{context.implementation}}` - Implementation progress
- `{{context.testing}}` - Testing progress
- `{{context.blockers}}` - Array of blockers

### Worktree Variables
- `{{worktree.path}}` - Absolute path to worktree
- `{{worktree.branch}}` - Git branch name
