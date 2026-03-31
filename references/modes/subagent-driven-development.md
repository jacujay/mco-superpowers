# Subagent-Driven Development

> Mode: `subagent-dev` — Use when you have a plan and MCO is available.

**Announce at start:** "I'm using the superpowers subagent-driven-development mode with MCO."

## Core Concept

**MCO-powered execution + two-stage review (spec then quality) = high quality, fast iteration.**

Each task uses `mco run` with a single provider for focused execution. The coordinator maintains the big picture while MCO handles the agent orchestration.

**Provider rotation strategy** (avoids the same agent reviewing its own work):

> **Note:** Providers are loaded from `~/.openclaw/skills/superpowers/providers.json` at startup. If the file does not exist, activate `setup` mode first.

| Stage | Config Key | Reason |
|-------|------------|--------|
| Implementation | `implementation` | Strongest implementation capability |
| Spec Review | `specReview` | Independent from implementer |
| Quality Review | `qualityReview` | Independent from first two stages |
| Final Code Review | `implementation` | Global perspective, summary judgment |
| Parallel Dispatch | `parallelProviders[]` | Round-robin across listed providers |

## Prerequisites

- [ ] A written plan exists (from writing-plans mode)
- [ ] Plan has been reviewed and approved
- [ ] Git worktree is set up (use git-worktrees mode)
- [ ] Tracker file created at `docs/superpowers/tracker.md`
- [ ] `mco` is installed and configured (`mco doctor` passes)
- [ ] `providers.json` exists (or run `setup` mode first)

## Execution Modes

### Safe Mode (Default)

Tasks are executed **serially** — one task at a time. Each task goes through the full implement → review → fix cycle before the next starts.

Use when: tasks share files, have dependencies, or you want maximum safety.

### Parallel Mode

Independent tasks are dispatched to **multiple `mco run` instances simultaneously**, each in its own worktree with a different provider.

Use when: 3+ tasks are truly independent (different files, different directories, no shared state).

**Requirements for parallel mode:**
- Each task works in a separate git worktree
- No two tasks modify the same file
- Each task uses a different provider (claude / codex / opencode)
- Run conflict check after all complete: `bash scripts/check-conflicts.sh`

## Per-Task Workflow

For each task in the plan:

### 1. Implementation

Update tracker: 🔄 in progress

Dispatch via MCO (providers read from `providers.json`):

```bash
# Load providers.json first:
#   implementation = $.implementation
#   specReview = $.specReview
#   qualityReview = $.qualityReview

mco run \
  --repo {WORKTREE_PATH} \
  --prompt "[Paste implementer prompt from references/prompts/implementer-prompt.md
   with TASK_DESCRIPTION, PLAN_SECTION, and SPEC_SECTION filled in]" \
  --providers {implementation} \
  --json
```

Wait for MCO to complete. Check status:
- **DONE** → proceed to spec review
- **DONE_WITH_CONCERNS** → evaluate concerns, proceed to spec review
- **BLOCKED** → investigate, unblock or escalate
- **NEEDS_CONTEXT** → provide context, re-dispatch

### 2. Spec Compliance Review

Dispatch spec reviewer:

```bash
mco run \
  --repo {WORKTREE_PATH} \
  --prompt "[Paste spec reviewer prompt from references/prompts/spec-reviewer-prompt.md
   with IMPLEMENTER_REPORT, TASK_REQUIREMENTS, and FILE_LIST filled in]" \
  --providers {specReview} \
  --json
```

Or via sessions_send to a reviewer agent:
```
sessions_send to:reviewer message:"Spec compliance review for Task N: [details]"
```

**If issues found:**
1. Send issues back for fixes (re-dispatch with `qualityReview` provider):
   ```bash
   mco run --repo {WORKTREE_PATH} --prompt "[Fix the following issues: ...]" --providers {qualityReview} --json
   ```
2. Re-review after fixes (dispatch spec reviewer again with `{specReview}`)
3. Max 3 iterations

**Do NOT proceed to quality review until spec compliance passes.**

### 3. Code Quality Review

Only after spec compliance passes:

```bash
mco run \
  --repo {WORKTREE_PATH} \
  --prompt "[Paste code quality reviewer prompt from references/prompts/code-quality-reviewer-prompt.md
   with IMPLEMENTER_REPORT, TASK_DESCRIPTION, BASE_SHA, and HEAD_SHA filled in]" \
  --providers {qualityReview} \
  --chain \
  --json
```

**If Critical or Important issues found:**
1. Send back for fixes (re-dispatch with `implementation` provider):
   ```bash
   mco run --repo {WORKTREE_PATH} --prompt "[Address quality issues: ...]" --providers {implementation} --json
   ```
2. Re-review after fixes (dispatch quality reviewer again with `{qualityReview}`)
3. Max 3 iterations

### 4. Task Completion

When both reviews pass:
1. Update tracker: ✅ completed
2. Record verification evidence in tracker
3. Move to next task

### 5. Repeat

Continue until all tasks are done.

## After All Tasks Complete

1. **Final code review** — dispatch a full code review:
   ```bash
   mco run \
     --repo {WORKTREE_PATH} \
     --prompt "[Paste code reviewer prompt from references/prompts/code-reviewer-prompt.md
      with full feature context]" \
     --providers claude \
     --json
   ```

2. **Finish branch** — transition to finish-branch mode:
   > "All tasks complete and reviewed. I'm using the superpowers finish-branch mode."

**OpenClaw enhancement:** Broadcast completion:
```
sessions_send announce:true message:"[superpowers] 🏁 All tasks done for {feature}: {N} tasks, all reviews passed."
```

## Parallel Mode Details

When using parallel mode:

### Setup
```bash
# Create separate worktrees for each parallel task
git worktree add .worktrees/task-3 -b feature/task-3
git worktree add .worktrees/task-4 -b feature/task-4
git worktree add .worktrees/task-5 -b feature/task-5
```

### Dispatch (each with a different provider)
```bash
# Launch all in parallel — claude, codex, opencode
mco run --repo .worktrees/task-3 --prompt "[implementer prompt for task 3]" --providers claude --json &
mco run --repo .worktrees/task-4 --prompt "[implementer prompt for task 4]" --providers codex --json &
mco run --repo .worktrees/task-5 --prompt "[implementer prompt for task 5]" --providers opencode --json &
wait

# Synthesize results (single provider)
mco run \
  --repo . \
  --prompt "Synthesize the results from the three parallel tasks. Each task's result is in the context above." \
  --providers claude \
  --json
```

### Monitor
Use `process action:list` to check running `mco` processes.

### Integrate
After all complete:
1. Run conflict check: `bash scripts/check-conflicts.sh`
2. Merge each task branch into the feature branch
3. Run full test suite on merged result
4. Proceed with reviews on the integrated code

## Critical Rules

- **Never start on main/master** without explicit consent
- **Never skip either review stage** (spec then quality, in that order)
- **Never make agents read the plan file directly** — paste the relevant section
- **Never proceed with unresolved review issues**
- **Spec compliance MUST pass before code quality review**
- **Update tracker after every status change**
- **In safe mode: one task at a time, no parallel implementation**
- **Provider rotation**: implement→claude, spec→codex, quality→opencode — do not skip the rotation
