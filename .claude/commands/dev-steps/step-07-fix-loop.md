# Step 7 — FIX LOOP (Developer, Sonnet)

## Agent Loading

First, read `{project-root}/.claude/agents/developer.md` and adopt that identity fully. You are the Developer in validation-fix mode for this step.

You are an isolated sub-agent. You fix errors reported by the Validator. Do not touch anything else.

## Execution Mode — LOW

- **Intent:** mechanical fixes for linter/TS errors — one error at a time, minimal change
- **Effort:** low — each error has a standard fix pattern; don't architect, don't refactor
- **Thinking:** minimal — read error, identify fix pattern, apply, next
- **Output:** terse fix-loop-log with one line per error, no commentary

## Reading Order (stable — preserves prompt cache)

Read files in this exact order. Stable files first, volatile files last. Step 7 may run up to 3 times in a session — keeping the prefix stable lets each subsequent invocation reuse cache from the previous turn.

1. `{project-root}/.claude/rules/typescript-conventions.md` — TS error fix patterns
2. `{session-dir}/.context` — paths
3. `{session-dir}/plan.md` — intended types, naming, structure
4. `{session-dir}/context-pack.md` — pre-implementation snapshot
5. `{session-dir}/context-pack.delta.md` — accumulated changes from Step 3 and any prior Step 5/7 iterations
6. `{session-dir}/escalation.md` — if exists, the architect's revised approach; treat it as authoritative over plan.md
7. `{session-dir}/escalation-confirmed.md` — if exists, the user's confirmation/notes
8. `{session-dir}/validate-log.md` — current errors (most volatile)
9. **Editing target files:** always via direct `Read` + `Edit` of the real file on disk. Use pack/delta only for understanding context.

## Your job

Read validation log. Fix every error. Do not touch anything else. If `escalation.md` exists, follow its Revised Approach instead of guessing.

## Instructions

1. Follow the Reading Order above. Do NOT shuffle it.

2. For each error:
   - Read the file mentioned in the error
   - Fix the specific issue

4. Common fixes:
   - **TS2322 type mismatch**: add proper type annotation or type guard
   - **TS2339 property does not exist**: check import or add to type
   - **no-explicit-any**: replace `any` with proper type, or add comment + ticket reference
   - **import/order**: reorder imports: Core > Platform > Elements > Hooks > Templates > Views > Components > Other
   - **max-nesting-depth**: flatten nested SCSS selectors
   - **no-unused-vars**: remove the unused import or variable
   - **@typescript-eslint/no-unused-vars**: same as above

5. Write `{session-dir}/fix-loop-log.md`:
   ```markdown
   # Fix Loop

   ## Errors fixed
   - {file}:{line} — {error code}: {what was fixed}

   ## Total: {N} errors fixed
   ```

6. **Append to `{session-dir}/context-pack.delta.md`** — do NOT overwrite. Add a new section at the end:

   ```markdown
   ## Step 7 — Fix Loop iteration {N} ({timestamp})

   ### Modified files (unified diff)
   {For each file you touched in this iteration, paste `git diff -- <path>` output. Only the diffs since the previous delta section.}
   ```

   This keeps Step 6 (validate) and any subsequent Step 7 iteration informed of the latest state.

7. Update spec.md status to `step: fix-loop-done`.
