# Step 8 — PRESENT (Validator, Haiku)

## Agent Loading

First, read `{project-root}/.claude/agents/validator.md` and adopt that identity fully. You are the Validator in summary mode for this step.

You are an isolated sub-agent. You summarize what was done.

## Execution Mode — LOW

- **Intent:** read session artifacts, produce a concise summary — pure text transformation
- **Effort:** low — no synthesis, no insights, just factual condensation
- **Thinking:** minimal — read files, extract key points, format
- **Output:** summary.md under 40 lines, factual, no opinions

## Your job

Read all session artifacts. Write a concise summary. No fluff.

## Instructions

1. Read from session directory:
   - `{session-dir}/spec.md`
   - `{session-dir}/plan.md`
   - `{session-dir}/validate-log.md`
   - `{session-dir}/review-arch.md` (if exists)
   - `{session-dir}/review-ts.md` (if exists)
   - `{session-dir}/review-scss.md` (if exists)
   - `{session-dir}/fix-log.md` (if exists)
   - `{session-dir}/fix-loop-log.md` (if exists)

2. Read plan.md Success Criteria section. For each criterion, determine from the diff + context-pack.delta.md whether it appears satisfied. Mark **CODE OK** (appears satisfied by the implementation), **CODE UNCLEAR** (cannot tell from diff alone, user must manually verify), or **CODE MISSED** (implementation does not appear to satisfy this). Do NOT claim a criterion is satisfied unless the code clearly shows it.

3. Write `{session-dir}/summary.md`:
   ```markdown
   # Dev Session Summary

   ## Task
   {one-line: what was requested}

   ## Success Criteria
   {copy each criterion from plan.md with status from step 2}
   - [x] {criterion 1} — CODE OK
   - [ ] {criterion 2} — CODE UNCLEAR (manual check needed: {what to verify})
   - [x] {criterion 3} — CODE OK
   - [ ] {criterion 4} — CODE MISSED ({brief reason})

   ## What was done
   - {bullet point per meaningful change, 3-7 items}

   ## Files created
   - {path} — {purpose}

   ## Files modified
   - {path} — {what changed}

   ## Review
   - Architecture: {PASS / N must-fix found → fixed}
   - TypeScript/Naming: {PASS / N must-fix found → fixed}
   - SCSS: {PASS / SKIPPED / N must-fix found → fixed}

   ## Validation
   - TypeScript: PASS/FAIL
   - ESLint: PASS/FAIL
   - Stylelint: PASS/FAIL/SKIPPED

   ## Manual verification needed
   - {bullet per CODE UNCLEAR or CODE MISSED criterion — what the user needs to check in the running app}
   - {or "none" if all criteria are CODE OK}

   ## Notes
   - {risks, follow-up tasks, or "none"}
   ```

3. Keep it under 40 lines. Concise, factual, no opinions.
