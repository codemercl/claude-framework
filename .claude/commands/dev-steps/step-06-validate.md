# Step 6 — VALIDATE (Validator, Haiku)

## Agent Loading

First, read `{project-root}/.claude/agents/validator.md` and adopt that identity fully. You are the Validator for this step.

You are an isolated sub-agent. You run tools and report. You do NOT fix anything.

## Execution Mode — LOW

- **Intent:** run linters, parse output, format report — pure mechanical task
- **Effort:** low — this is tool execution + regex filtering, not analysis
- **Thinking:** minimal — no reasoning about errors, just diff against baseline and report
- **Output:** structured validate-log.md with counts and raw error lines, no opinions, no suggestions

## Your job

Run project linters and type checker. Compute the **regression delta** against the merge-base baseline (if available) so cascade errors are caught. Report results.

## Instructions

1. Read `{session-dir}/spec.md` — get the lists of created/modified files and SCSS files from Implementation section.

2. Source the session context to load baseline file paths:
   ```bash
   source "{session-dir}/.context"
   ```
   This sets `BASELINE_TS`, `BASELINE_ESLINT`, `BASELINE_STYLELINT`, `PROJECT_ROOT`, `SESSION_DIR`.

3. If a background baseline is still running, wait for it (max 180s):
   ```bash
   if [ -f "$SESSION_DIR/baseline.pid" ]; then
     PID=$(cat "$SESSION_DIR/baseline.pid")
     for i in $(seq 1 180); do
       kill -0 "$PID" 2>/dev/null || break
       sleep 1
     done
   fi
   ```

4. Get the list of changed files:
   ```bash
   cd "$PROJECT_ROOT" && git diff --name-only > "$SESSION_DIR/changed-files.txt"
   ```

5. Run TypeScript check from `front/`:
   ```bash
   cd "$PROJECT_ROOT/front"
   yarn typescript 2>&1 | grep 'error TS' | sort -u > "$SESSION_DIR/current-ts.txt" || true
   ```

6. Compute new TS errors (regressions, including cascades):
   ```bash
   if [ -f "$BASELINE_TS" ] && [ ! -f "$SESSION_DIR/.no-baseline" ]; then
     comm -23 "$SESSION_DIR/current-ts.txt" "$BASELINE_TS" > "$SESSION_DIR/new-ts-errors.txt"
     TS_MODE="baseline-diff"
   else
     # Fallback: filter current errors to changed files only (cascade detection unavailable)
     if [ -s "$SESSION_DIR/changed-files.txt" ]; then
       grep -F -f "$SESSION_DIR/changed-files.txt" "$SESSION_DIR/current-ts.txt" \
         > "$SESSION_DIR/new-ts-errors.txt" || true
     else
       : > "$SESSION_DIR/new-ts-errors.txt"
     fi
     TS_MODE="filename-filter"
   fi
   ```

7. Repeat 5–6 for ESLint:
   ```bash
   cd "$PROJECT_ROOT/front"
   yarn lint:ts 2>&1 | grep -E ' (error|warning) ' | sort -u > "$SESSION_DIR/current-eslint.txt" || true

   if [ -f "$BASELINE_ESLINT" ] && [ ! -f "$SESSION_DIR/.no-baseline" ]; then
     comm -23 "$SESSION_DIR/current-eslint.txt" "$BASELINE_ESLINT" > "$SESSION_DIR/new-eslint-errors.txt"
     ESLINT_MODE="baseline-diff"
   else
     if [ -s "$SESSION_DIR/changed-files.txt" ]; then
       grep -F -f "$SESSION_DIR/changed-files.txt" "$SESSION_DIR/current-eslint.txt" \
         > "$SESSION_DIR/new-eslint-errors.txt" || true
     else
       : > "$SESSION_DIR/new-eslint-errors.txt"
     fi
     ESLINT_MODE="filename-filter"
   fi
   ```

8. Stylelint — only if any `.scss` file is in `changed-files.txt`:
   ```bash
   if grep -q '\.scss$' "$SESSION_DIR/changed-files.txt"; then
     cd "$PROJECT_ROOT/front"
     yarn lint:scss 2>&1 | grep -E ' (✖|error|warning) ' | sort -u > "$SESSION_DIR/current-stylelint.txt" || true

     if [ -f "$BASELINE_STYLELINT" ] && [ ! -f "$SESSION_DIR/.no-baseline" ]; then
       comm -23 "$SESSION_DIR/current-stylelint.txt" "$BASELINE_STYLELINT" \
         > "$SESSION_DIR/new-stylelint-errors.txt"
       STYLELINT_MODE="baseline-diff"
     else
       grep -F -f "$SESSION_DIR/changed-files.txt" "$SESSION_DIR/current-stylelint.txt" \
         > "$SESSION_DIR/new-stylelint-errors.txt" || true
       STYLELINT_MODE="filename-filter"
     fi
   else
     STYLELINT_MODE="skipped"
     : > "$SESSION_DIR/new-stylelint-errors.txt"
   fi
   ```

9. Write `{session-dir}/validate-log.md`:
   ```markdown
   # Validation Results

   ## Mode
   - TypeScript: {TS_MODE}
   - ESLint: {ESLINT_MODE}
   - Stylelint: {STYLELINT_MODE}

   ## Changed files
   {contents of changed-files.txt}

   ## TypeScript (yarn typescript)
   - Baseline (merge-base origin/main): {wc -l of $BASELINE_TS, or "n/a"}
   - Current: {wc -l of current-ts.txt}
   - NEW (regressions, includes cascades): {wc -l of new-ts-errors.txt}
   - Errors:
   {first 30 lines of new-ts-errors.txt}

   ## ESLint (yarn lint:ts)
   - Baseline: {wc -l of $BASELINE_ESLINT, or "n/a"}
   - Current: {wc -l of current-eslint.txt}
   - NEW: {wc -l of new-eslint-errors.txt}
   - Errors:
   {first 30 lines of new-eslint-errors.txt}

   ## Stylelint (yarn lint:scss)
   - Status: {PASS / FAIL / SKIPPED}
   - NEW: {wc -l of new-stylelint-errors.txt, or "n/a"}
   - Errors:
   {first 30 lines of new-stylelint-errors.txt}

   ## Summary
   - New errors total: {sum of NEW counts}
   - All clean: {yes / no}
   - Cascade detection: {available — if all modes are baseline-diff | unavailable — if any uses filename-filter}
   ```

10. Do NOT fix anything. Do NOT suggest fixes. Only report.
