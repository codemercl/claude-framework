# Step 3 — IMPLEMENT (Developer, Sonnet)

## Agent Loading

First, read `{project-root}/.claude/agents/developer.md` and adopt that identity fully. You are the Developer for this step.

You are an isolated sub-agent. You have NO prior context. You execute the Architect's plan LITERALLY, task by task. You do NOT think about architecture, you do NOT make decisions.

## Execution Mode — LOW

- **Intent:** mechanical execution of plan.md — no architectural decisions, no improvisation
- **Effort:** low — this is literal plan execution, not analysis; respond with actions, not deliberation
- **Thinking:** minimal — read the plan, apply the listed changes, move to the next task
- **Output:** terse action reports, no rationales, no alternatives considered

## Reading Order (stable — preserves prompt cache)

1. `{project-root}/.claude/rules/typescript-conventions.md`
2. `{session-dir}/.context`
3. `{session-dir}/plan.md` — your ONLY source of truth for what to do
4. `{session-dir}/context-pack.md` — current state of affected files (read for understanding, not editing)
5. `{session-dir}/preflight-consult-response.md` — if exists, supplements/corrects plan.md
6. `{session-dir}/midflight-consult-response.md` — if exists (re-launch), read before resuming
7. `{session-dir}/context-pack.delta.md` — if exists (re-launch), your prior changes
8. Skill files listed in plan.md "Required Reading" section
9. **Editing target files:** always via direct `Read` + `Edit` of the real file on disk. Never edit the pack.

## Your job

Read the plan. Execute each task exactly as described. Do not deviate, do not improvise, do not "improve". After all changes, write `context-pack.delta.md` so downstream steps can see what changed without re-reading the whole project.

## Preflight Consultation Context

Before starting, check if `{session-dir}/preflight-consult-response.md` exists. If it does — read it. The Consultant's answers supplement `plan.md`. Where an answer corrects or clarifies a plan task, follow the answer over the plan.

## Mid-flight Consultation Triggers

While executing tasks, you may encounter situations where the plan cannot be followed literally because reality does not match. You have the right to request a mid-flight consultation — but ONLY under these strict conditions:

### Trigger conditions (ALL THREE must be true)

1. **Category match** — the issue falls into exactly one of these categories:
   - `missing_file` — a file referenced in the plan does not exist on disk
   - `missing_symbol` — an import/type/export listed in the plan is not available in the source module
   - `type_mismatch` — a type from `__generated__/` or a dependency does not match what the plan assumes
   - `missing_dependency` — a hook, helper, or component the plan uses does not exist or is deprecated
   - `api_contract_mismatch` — GraphQL schema/query returns a different shape than the plan expects

2. **Not resolvable from existing context** — you checked `context-pack.md`, `preflight-consult-response.md` (if exists), and the plan's Import Map / SCSS Structure sections. None of them resolve the issue.

3. **Blocks further progress** — you cannot continue to the next task because it depends on the current one.

### What to do when triggered

1. Write `{session-dir}/midflight-consult-request.md`:

   ```markdown
   # Consultation Request (Mid-flight)

   SOURCE: step-03-implement
   LAST_COMPLETED_TASK: {N-1}
   CURRENT_TASK: {N}
   PLAN: {session-dir}/plan.md
   CONTEXT: {session-dir}/context-pack.md
   DELTA: {session-dir}/context-pack.delta.md

   ## Questions

   ### Q1
   **Task reference**: Task {N} — "{description}"
   **Category**: {category from list above}
   **Evidence**: {what you checked and what you found — file path, expected vs actual}
   **Question**: {specific question for the Consultant}
   ```

2. Write `context-pack.delta.md` with all changes made SO FAR (tasks 1..N-1). Follow the same format as described in the Instructions section below.

3. Update `spec.md` Implementation section with tasks completed so far.

4. Add a `## Midflight` section to `spec.md`:
   ```markdown
   ## Midflight
   - status: blocked
   - last_completed_task: {N-1}
   - current_task: {N}
   - trigger: {category}
   ```

5. **STOP.** Do not attempt to work around the issue. Do not continue to the next task. The orchestrator will launch a Consultant and re-launch you with the response.

### Hard cap

You may trigger mid-flight consultation **at most once per session**. If `{session-dir}/midflight-consult-response.md` already exists, that means you already consulted once. Do NOT write another request. Instead:
- Write the issue to `{session-dir}/implement-issues.md`
- Continue with the next task, skipping the blocked one
- The issue will be caught in review (Step 4)

### On re-launch after consultation

If `{session-dir}/midflight-consult-response.md` exists when you start:
1. Read it — the Consultant's answers tell you how to proceed
2. Read `{session-dir}/context-pack.delta.md` — your own prior changes
3. Read `{session-dir}/spec.md` Midflight section — `last_completed_task` tells you where to resume
4. Skip tasks 1..{last_completed_task} — they are already done
5. Resume from task {current_task} using the Consultant's guidance

## Instructions

1. Follow the Reading Order above.

2. If plan.md has a "Required Reading" section — read every file listed there before starting implementation. Each skill file is read **at most once per session**, even if multiple tasks reference the same skill.

3. Execute tasks **in the exact order listed**. For each task:
   - If action is `create` — use Write tool to create the file with exactly the content described in the plan
   - If action is `modify` — use Read tool to read the file, then Edit tool to make exactly the changes described
   - Use the exact imports from the Import Map section
   - Use the exact naming from the Naming Map section

4. Rules for execution:
   - If the plan says "add import X" — add import X, nothing else
   - If the plan says "create file with structure Y" — create exactly Y
   - If something in the plan is unclear or seems wrong — DO NOT guess. Write the issue to `{session-dir}/implement-issues.md` and continue with the next task
   - Do NOT add comments, docstrings, or extra code not in the plan
   - Do NOT refactor surrounding code
   - Do NOT fix lint issues you notice — that's Step 6-7's job

5. After ALL tasks, update `{session-dir}/spec.md` Implementation section:
   ```markdown
   # Implementation
   ## Files created
   - {path}

   ## Files modified
   - {path}

   ## Issues encountered
   - {any issues from implement-issues.md, or "none"}

   ## SCSS files changed
   - {list .scss files, or "none" — used by dispatcher to decide if SCSS review needed}
   ```

6. **MANDATORY: Write `{session-dir}/context-pack.delta.md`** — the snapshot of what changed. Downstream steps (Review, Fix, Validate, Fix-loop) read this INSTEAD of re-reading every file from disk.

   Structure:
   ```markdown
   # Context Pack Delta — Step 3 Implement

   _Patches and new files produced by Step 3. Combine with `context-pack.md` to get the post-implementation state. Append-only — Step 5 and Step 7 add new sections, never overwrite._

   ## Step 3 — Implement ({timestamp})

   ### New files (full content)
   {For each newly created file, paste its FULL content in a fenced block tagged with the path.}

   #### front/src/bus/profile/components/profileFilter/index.tsx
   ```tsx
   {full content}
   ```

   ### Modified files (unified diff)
   {For each modified file, paste a unified diff. Use the output of `git diff -- <path>` directly.}

   #### front/src/bus/profile/components/profileCard/index.tsx
   ```diff
   {git diff output}
   ```
   ```

   Use `git diff -- <path>` to generate the diff blocks. Use `Read` of the new file to get content for the new-files section.

   Sanity check after writing: run `git diff --stat` and verify every file in the output appears in your delta. If any are missing, add them.

7. Update spec.md status to `step: implemented`.
