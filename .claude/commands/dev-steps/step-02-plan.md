# Step 2 — PLAN (Architect, Opus)

## Agent Loading

First, read `{project-root}/.claude/agents/architect.md` and adopt that identity fully. You are the Architect for this step.

You are an isolated sub-agent. You have NO prior context. You do NOT write code. You write an exhaustive, unambiguous implementation document that a Developer agent (Sonnet) will follow LITERALLY to write code.

## Execution Mode — HIGH

- **Intent:** architectural decision-making — naming, imports, task decomposition, trade-offs
- **Effort:** high — surface alternatives you considered and rejected; trade-offs matter
- **Thinking:** deep — reason through edge cases, cross-domain impacts, and convention conflicts before committing to the plan
- **Output:** structured plan.md with explicit "why" lines in Architecture Decision section

## Your job

Read the task context and user answers. Produce a plan so detailed that a developer who has never seen this codebase can implement it by following your document step by step.

## Reading Order (stable — preserves prompt cache)

Read files in this exact order. Stable files (rules, context) first, volatile files last. This keeps the prompt cache warm across the multiple turns the architect needs.

1. Project convention rules (most stable, role-wide):
   - `{project-root}/.claude/rules/typescript-conventions.md` — always
   - `{project-root}/.claude/rules/scss-conventions.md` — only if SCSS may be involved
   - `{project-root}/.claude/rules/graphql-conventions.md` — only if GraphQL may be involved
   - `{project-root}/.claude/rules/redux-conventions.md` — only if Redux may be involved
2. `{session-dir}/.context` — paths and baseline references
3. `{session-dir}/context-pack.md` — **PRIMARY source for project code.** Built by the Analyst in Step 1. Read this INSTEAD of re-reading source files.
4. `{session-dir}/spec.md` — task and context
5. `{session-dir}/user-answers.md` — answers to clarification questions
6. `{session-dir}/plan-feedback.md` — if exists, prior plan-revision feedback
7. **Fallback only:** if `context-pack.md` is missing a file you need, fall back to a direct `Read` of that source file. Note the miss in your plan output so the Analyst can fix the pack later.

## Instructions

1. Follow the Reading Order above. Do NOT shuffle it — the order is intentional for prompt-cache reuse across your turns.

2. Treat `context-pack.md` as your default source of truth for current code. Only fall back to `Read` of real files when the pack is incomplete.

2. Write `{session-dir}/plan.md` — THE implementation blueprint:

```markdown
# Implementation Plan

## Task
{one-line summary}

## Success Criteria
{3-5 verifiable semantic goals from user-answers.md. These describe "task done correctly" from the user's perspective, not convention checks. Copy from questions.md → Success criteria section, refined based on user-answers.md. Each criterion must be checkable by reading the diff or running the app — NOT by running a linter.}

- [ ] {criterion 1 — user-facing outcome}
- [ ] {criterion 2 — regression check, what must NOT break}
- [ ] {criterion 3 — edge case or responsive behavior}
- [ ] {criterion 4 — optional}
- [ ] {criterion 5 — optional}

## Architecture Decision
{WHY this approach was chosen over alternatives. What was considered and rejected. Explicitly tie decisions to Success Criteria — "chose X because it satisfies criterion 2 without breaking criterion 1".}

## Naming Map
{entity name consistency across all layers — fill in only what applies}
- GraphQL operation: `{QUERY_ENTITY_ACTION}` or `{MUTATION_ENTITY_ACTION}`
- Generated type: `{OperationNameType}`
- Hook: `{useQueryEntityAction}` or `{useMutationEntityAction}`
- Directory: `{entityAction}/`
- Component: `{EntityActionComponent}` (if smart) or `{EntityAction}` (if element)
- CSS root class: `.{entityAction}`

## Ordered Tasks

### Task 1: {description}
- **File**: {exact path from project root}
- **Action**: create | modify
- **What to do**:
  {EXACT description of what to write or change. For new files — full structure with imports, types, function signature, return statement. For modifications — which lines to change and what to change them to. Reference existing patterns from files you read.}
- **Depends on**: Task N | none

### Task 2: {description}
- **File**: {exact path}
- **Action**: create | modify
- **What to do**:
  {same level of detail}
- **Depends on**: Task N | none

{...continue for all tasks...}

## Import Map
{For each new/modified .ts/.tsx file, list the exact imports it needs, grouped:}

### {file path}
```typescript
// Core
import React, { ReactElement, FC } from 'react';

// Elements
import { Button } from 'ui-kit';

// Hooks
import { useQuerySomething } from '../../hooks/useQuerySomething';

// Other
import styles from './styles/index.module.scss';
```

## SCSS Structure
{For each new component with styles, specify:}

### {component path}/styles/
- `index.module.scss`: root class `.{folderName}`, which breakpoints to include
- `desktop.scss`: {what styles go here}
- `palette.scss`: {which color variables needed}
- `mq/desktop.scss`: {what responsive styles}

## Required Reading
{List rules and skills the Developer agent MUST read before implementing. Only include what's relevant to this task.}

- `{project-root}/.claude/rules/typescript-conventions.md` — {always include}
- `{project-root}/.claude/rules/scss-conventions.md` — {only if SCSS tasks}
- `{project-root}/.claude/rules/graphql-conventions.md` — {only if GraphQL tasks}
- `{project-root}/.claude/rules/redux-conventions.md` — {only if Redux tasks}
- `{project-root}/.claude/skills/scss-module.md` — {only if creating new styles}
- `{project-root}/.claude/skills/use-query-hook.md` — {only if creating query hook}
- {etc. — list only what applies}

## Validation Checklist
- [ ] No cross-domain bus/ imports
- [ ] All types end with `Type` suffix
- [ ] `PropsType` for all component props
- [ ] `ReturnedType` for all hook returns
- [ ] No `as Type` casting
- [ ] No `interface`, only `type`
- [ ] Import order correct with comment headers
- [ ] Root CSS class = folder name
- [ ] `px()` only in mq/ files
- [ ] Colors through palette.scss
```

5. The plan must be **self-contained** — the implement agent should NOT need to read any other project files to understand what to do. Copy relevant code snippets, type definitions, and patterns into the plan itself.

6. Update spec.md status to `step: planned`.
