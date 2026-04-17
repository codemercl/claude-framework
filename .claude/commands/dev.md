---
name: dev
description: 'Structured development workflow with isolated sub-agents on different models: clarify (Sonnet) → plan (Opus) → preflight (Sonnet) → consult (Opus, conditional) → implement (Sonnet) → consult mid-flight (Opus, conditional) → review (Opus) → fix (Sonnet) → validate (Haiku) → present (Haiku). Usage: /dev <task description>'
---

# Dev Workflow

You are a thin dispatcher. You do NOT write code, do NOT analyze, do NOT make decisions. You ONLY:
1. Create session directory and spec.md
2. Launch sub-agents with the right model
3. Read result files between steps
4. Route to next step
5. Show brief status to user at checkpoints

ALL thinking and work happens inside sub-agents. You are just the router.

## CRITICAL: Variable Substitution

Before passing ANY prompt to a sub-agent, you MUST replace these placeholders with real values:
- `{SESSION_DIR}` → the actual session directory path (e.g. `/Users/.../.claude/sessions/dev-20260410-143022`)
- `{PROJECT_ROOT}` → the actual git repository root path

Sub-agents receive REAL paths, never placeholder strings.

### Pre-flight check (mandatory before EVERY Agent launch)

Before invoking the Agent tool, validate the prompt string in your head:
1. The prompt MUST NOT contain the literal substrings `{SESSION_DIR}`, `{PROJECT_ROOT}`, `{session-dir}`, or `{project-root}`.
2. The prompt MUST contain SESSION_DIR as an absolute path starting with `/`.
3. If either check fails — regenerate the prompt with real values. Do NOT launch.

Note: step files themselves use `{session-dir}` / `{project-root}` lowercase as descriptive references inside their instructions. That is intentional — the agent receives the real paths in its launch prompt and resolves those references on its own. The preflight check applies only to **launch prompts**, not to step file contents.

## Session Setup

Run this single bash block at the very start of /dev. It detects project root, cleans old sessions, creates the new session directory, and kicks off the validator baseline in the background. The heavy `yarn` calls run in a forked sub-shell — the outer Bash returns immediately, so Step 1 starts in parallel with the baseline.

```bash
# 1. Detect project root
PROJECT_ROOT=$(git rev-parse --show-toplevel)

# 2. Cleanup: remove dev sessions older than 7 days (silent, never fails)
find "$PROJECT_ROOT/.claude/sessions" -maxdepth 1 -type d -name 'dev-*' -mtime +7 \
  -exec rm -rf {} + 2>/dev/null || true
find "$PROJECT_ROOT/.claude/cache/baseline" -maxdepth 1 -type f -mtime +30 \
  -delete 2>/dev/null || true

# 3. Create new session directory
SESSION_DIR="$PROJECT_ROOT/.claude/sessions/dev-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$SESSION_DIR"

# 4. Baseline cache key = merge-base with origin/main
MERGE_BASE=$(git -C "$PROJECT_ROOT" merge-base origin/main HEAD 2>/dev/null \
             || git -C "$PROJECT_ROOT" rev-parse HEAD)
BASELINE_DIR="$PROJECT_ROOT/.claude/cache/baseline"
mkdir -p "$BASELINE_DIR"
BASELINE_TS="$BASELINE_DIR/ts-$MERGE_BASE.txt"
BASELINE_ESLINT="$BASELINE_DIR/eslint-$MERGE_BASE.txt"
BASELINE_STYLELINT="$BASELINE_DIR/stylelint-$MERGE_BASE.txt"

# 5. Persist context — Step 6 reads this to get baseline paths
cat > "$SESSION_DIR/.context" <<EOF
PROJECT_ROOT=$PROJECT_ROOT
SESSION_DIR=$SESSION_DIR
MERGE_BASE=$MERGE_BASE
BASELINE_TS=$BASELINE_TS
BASELINE_ESLINT=$BASELINE_ESLINT
BASELINE_STYLELINT=$BASELINE_STYLELINT
EOF

# 6. Kick off baseline in background — only if cache miss AND working tree clean.
#    Step 6 will wait for this PID before reading the baseline files.
if [ -f "$BASELINE_TS" ] && [ -f "$BASELINE_ESLINT" ]; then
  echo "baseline: cache hit ($MERGE_BASE)"
elif [ -n "$(git -C "$PROJECT_ROOT" status --porcelain)" ]; then
  echo "baseline: skipped (working tree dirty — validator falls back to filename filter)"
  touch "$SESSION_DIR/.no-baseline"
else
  (
    cd "$PROJECT_ROOT/front"
    yarn typescript 2>&1 | grep 'error TS'           | sort -u > "$BASELINE_TS"        || true
    yarn lint:ts    2>&1 | grep -E ' (error|warning) ' | sort -u > "$BASELINE_ESLINT"    || true
    yarn lint:scss  2>&1 | grep -E ' (✖|error|warning) ' | sort -u > "$BASELINE_STYLELINT" || true
  ) > "$SESSION_DIR/baseline.log" 2>&1 &
  echo $! > "$SESSION_DIR/baseline.pid"
  echo "baseline: started in background (pid $(cat "$SESSION_DIR/baseline.pid"))"
fi

echo "SESSION_DIR=$SESSION_DIR"
echo "PROJECT_ROOT=$PROJECT_ROOT"
```

Capture the printed `SESSION_DIR` and `PROJECT_ROOT` from this block — those are the literals you must substitute into every Agent launch prompt below. Run the preflight check before each launch.

Then write `$SESSION_DIR/spec.md`:
   ```markdown
   # Task
   {user's request from $ARGUMENTS}

   # Session
   - project-root: {PROJECT_ROOT}
   - session-dir: {SESSION_DIR}
   - status: starting

   # Context
   (Step 1 fills this)

   # Plan
   (Step 2 fills this)

   # Preflight
   (Step 2.5 fills this)

   # Midflight
   (Step 3 fills this, only if mid-flight consultation triggered)

   # Implementation
   (Step 3 fills this)

   # Review
   (Step 4 fills this)

   # Validation
   (Step 6 fills this)
   ```

## Error Handling

If any sub-agent fails, times out, or returns an error:
1. Tell the user which step failed
2. Ask: retry this step, skip to next step, or abort?
3. Do NOT proceed silently

## Workflow

### Step 1 — CLARIFY (Sonnet)

Launch Agent with `model: "sonnet"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-01-clarify.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, read `SESSION_DIR/questions.md`. Show the user:
- The Understanding section
- The Questions (numbered)

**STOP and WAIT for user answers.**

Write user's answers to `SESSION_DIR/user-answers.md` in this format:
```markdown
# User Answers

## Q1: {original question text}
A: {user's answer}

## Q2: {original question text}
A: {user's answer}
```

### Step 2 — PLAN (Opus)

Launch Agent with `model: "opus"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-02-plan.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, read `SESSION_DIR/plan.md`. Show user:
- Task summary (1 line)
- Architecture Decision (2-3 lines)
- List of tasks (names only, not full details)
- Number of files to create/modify

**STOP and WAIT for user confirmation.** If user wants changes — write feedback to `SESSION_DIR/plan-feedback.md` and re-run Step 2.

### Step 2.5 — PREFLIGHT (Sonnet → conditional Opus)

**Step 2.5a — Detect blockers (Sonnet)**

Launch Agent with `model: "sonnet"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-02b-preflight.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, read `SESSION_DIR/preflight-blockers.md`.

**Routing:**
- If `STATUS: clear` — proceed to Step 3.
- If `STATUS: blocked` — proceed to Step 2.5b (consult).

**Step 2.5b — Consult (Opus) — only if blockers found**

Launch Agent with `model: "opus"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-consult.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
Consultation request file: {SESSION_DIR}/preflight-consult-request.md
Response file to write: {SESSION_DIR}/preflight-consult-response.md
```

After agent returns, read `SESSION_DIR/preflight-consult-response.md`. Check `PLAN_STATUS`:

- `OK` or `PLAN_PATCH` — proceed to Step 3. The Developer will read the response.
- `PLAN_BROKEN` — write the `## Plan Broken` section content to `SESSION_DIR/plan-feedback.md` and **re-run Step 2** (max 1 re-run). After re-run, go back to Step 2.5a. If Step 2.5b returns `PLAN_BROKEN` a second time — **STOP and show the user** the diagnosis from the response. Ask: rephrase the task, provide more context, or abort?

### Step 3 — IMPLEMENT (Sonnet)

Launch Agent with `model: "sonnet"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-03-implement.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, check for mid-flight consultation request:

**Mid-flight detection:**
1. Check if `SESSION_DIR/midflight-consult-request.md` exists AND `SESSION_DIR/midflight-consult-response.md` does NOT exist.
2. If yes — the Developer hit a blocker and stopped. Launch the Consultant:

   Launch Agent with `model: "opus"`:
   ```
   Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-consult.md and follow its instructions exactly.
   Session directory: {SESSION_DIR}
   Project root: {PROJECT_ROOT}
   Consultation request file: {SESSION_DIR}/midflight-consult-request.md
   Response file to write: {SESSION_DIR}/midflight-consult-response.md
   ```

   After Consultant returns, read `SESSION_DIR/midflight-consult-response.md`. Check `PLAN_STATUS`:
   - `OK` or `PLAN_PATCH` — **re-launch Step 3** (the Developer will read the response and resume from where it stopped).
   - `PLAN_BROKEN` — write to `SESSION_DIR/plan-feedback.md` and **re-run Step 2** → Step 2.5 → Step 3 from scratch. Hard cap: this path executes at most once per session.

3. If no mid-flight request — proceed normally.

Read `SESSION_DIR/spec.md` Implementation section. Proceed to Step 4.

### Step 4 — REVIEW (3x Opus, parallel)

Read `SESSION_DIR/spec.md` Implementation section to check which files were changed.

**Before launching reviewers**, run this bash block ONCE so all three agents share the same `git diff` output instead of computing it three times:

```bash
cd "$PROJECT_ROOT" && git diff > "$SESSION_DIR/diff.txt"
echo "diff.txt: $(wc -l < "$SESSION_DIR/diff.txt") lines"
```

Reviewers read `diff.txt` from disk; they MUST NOT run `git diff` themselves. Each reviewer also reads `context-pack.md` + `context-pack.delta.md` instead of re-reading source files.

Launch agents in parallel, all with `model: "opus"`:

Agent A — always:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-04-review-arch.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

Agent B — always:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-04-review-ts.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

Agent C — only if SCSS files listed in Implementation section:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-04-review-scss.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After all return, read `SESSION_DIR/review-arch.md`, `review-ts.md`, `review-scss.md`. Count must-fix items. If zero — skip Step 5, go to Step 6. Otherwise proceed to Step 5.

### Step 5 — FIX (Sonnet)

Launch Agent with `model: "sonnet"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-05-fix.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

### Step 6 — VALIDATE (Haiku)

Launch Agent with `model: "haiku"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-06-validate.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, read `SESSION_DIR/validate-log.md`. If all passed — go to Step 8. If errors — go to Step 7.

### Step 7 — FIX LOOP (max 3 iterations)

Launch Agent with `model: "sonnet"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-07-fix-loop.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, re-run Step 6. Max 3 total iterations of Step 6→7. If still failing — proceed to Step 7B (escalation), do NOT just stop and ask the user.

### Step 7B — ESCALATE (Architect, Opus) — only after Step 7 fails 3 times

Sonnet is stuck in a loop. Escalate to Opus to diagnose and propose a different approach. Do NOT skip this step in favor of asking the user — the architect's diagnosis IS what we show the user.

Launch Agent with `model: "opus"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-07b-escalate.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, read `SESSION_DIR/escalation.md`. Show the user:
- Diagnosis (2-4 sentences)
- Revised approach (file-level)
- Recommendation (APPLY / CONFIRM / ABORT)

Then route based on Recommendation:
- **APPLY** → re-run Step 7 once with `escalation.md` as additional input, then back to Step 6. If Step 6 still fails, give up and present remaining errors to user.
- **CONFIRM** → STOP and WAIT for user. On confirmation, write user's input to `SESSION_DIR/escalation-confirmed.md` and re-run Step 7 once, then Step 6.
- **ABORT** → present escalation.md to user and stop. Do not loop further.

Hard cap: Step 7B runs **at most once per session**. If Step 6 still fails after the post-escalation Step 7 run, present remaining errors to user and stop.

### Step 8 — PRESENT (Haiku)

Launch Agent with `model: "haiku"`:
```
Read {PROJECT_ROOT}/.claude/commands/dev-steps/step-08-present.md and follow its instructions exactly.
Session directory: {SESSION_DIR}
Project root: {PROJECT_ROOT}
```

After agent returns, read `SESSION_DIR/summary.md` and present to user.
