# Dispatching Parallel Agents

> Mode: `parallel-dispatch` — Use when you have 3+ independent problems across different domains.

**Announce at start:** "I'm using the superpowers parallel-dispatch mode with MCO."

## When to Use

**Decision tree:**

```
Multiple failures or tasks?
  └─ Yes → Are they in different domains/files?
       └─ Yes → Do they share state or dependencies?
            └─ No → Are there 3+ of them?
                 └─ Yes → USE PARALLEL DISPATCH
                 └─ No  → Handle sequentially
            └─ Yes → Handle sequentially (shared state = serial)
       └─ No → Handle sequentially (same domain = likely related)
  └─ No → Handle individually
```

## When NOT to Use

- Failures/tasks that might be related (same root cause)
- Exploratory debugging (don't know what's wrong yet)
- Tasks requiring full system context
- Tasks that modify the same files
- Fewer than 3 independent problems

## The Pattern

### Step 1: Identify Independent Problem Domains

Analyze the tasks and group by domain:
- Different files → likely independent
- Different modules → likely independent
- Same error in multiple places → likely ONE root cause (don't parallelize)

### Step 2: Create Focused Agent Tasks

For each independent domain, create a focused prompt:

**Good prompt (focused):**
```
Fix the timeout handling in agent-tool-abort.test.ts.
The test expects AbortError but gets TimeoutError.
Look at how abort signals propagate in src/agents/tools/.
Only modify files in this directory.
```

**Bad prompt (too broad):**
```
Fix all failing tests.
```

**Each prompt must include:**
- Specific scope (which files/directories)
- Context about the failure
- Expected outcome
- Constraints (what NOT to touch)

### Step 3: Dispatch in Parallel via MCO

Each independent task gets its own `mco run` with a different provider:

```bash
# Domain 1 — claude
mco run \
  --repo . \
  --prompt "[focused prompt for domain 1]" \
  --providers claude \
  --allow-paths src/domain1 \
  --target-paths src/domain1 \
  --json &

# Domain 2 — codex
mco run \
  --repo . \
  --prompt "[focused prompt for domain 2]" \
  --providers codex \
  --allow-paths src/domain2 \
  --target-paths src/domain2 \
  --json &

# Domain 3 — opencode
mco run \
  --repo . \
  --prompt "[focused prompt for domain 3]" \
  --providers opencode \
  --allow-paths src/domain3 \
  --target-paths src/domain3 \
  --json &

wait
```

**Provider rotation**: distribute claude / codex / opencode across tasks. No two tasks use the same provider.

### Step 4: Review and Integrate Results

After all agents complete:

1. **Collect results** — check each `mco run` output
2. **Synthesize** — dispatch a summary pass:
   ```bash
   mco run \
     --repo . \
     --prompt "Synthesize results from 3 parallel MCO runs. Each result is in the context above. Identify: (1) what was fixed, (2) any conflicts between findings, (3) next steps." \
     --providers claude \
     --json
   ```
3. **Check for conflicts** — run `bash scripts/check-conflicts.sh`
4. **Integrate** — merge results, resolve any conflicts
5. **Verify** — run full test suite on integrated result (verification mode)

## Common Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Too broad | "Fix all tests" | One agent per test file/domain |
| Related failures | 3 tests fail, same module | Investigate as one problem first |
| Shared state | Two agents modify `config.ts` | Handle sequentially |
| Missing context | "Fix the bug" | Include error message, file, expected behavior |
| No verification | Trust agent reports | Run full tests after integration |
| Same provider twice | Two tasks both use `claude` | Rotate: claude/codex/opencode |

## Real-World Example

**Situation:** 6 test failures across 3 files

```
FAIL src/agents/agent-tool-abort.test.ts     (2 failures)
FAIL src/agents/batch-completion.test.ts      (2 failures)
FAIL src/agents/race-condition.test.ts        (2 failures)
```

**Analysis:** Different files, different domains (abort logic, batch handling, race conditions). Likely independent.

**Dispatch (MCO, different providers):**
```bash
# Task 1: Abort logic — claude
mco run --repo . --prompt "Fix 2 failures in agent-tool-abort.test.ts. Focus on abort signal propagation in src/agents/tools/. Expected: AbortError." --providers claude --target-paths src/agents/tools,src/agents/agent-tool-abort.test.ts --json &

# Task 2: Batch completion — codex
mco run --repo . --prompt "Fix 2 failures in batch-completion.test.ts. Focus on batch result aggregation in src/agents/batch/. Expected: all items resolved." --providers codex --target-paths src/agents/batch,src/agents/batch-completion.test.ts --json &

# Task 3: Race conditions — opencode
mco run --repo . --prompt "Fix 2 failures in race-condition.test.ts. Focus on lock acquisition in src/agents/concurrency/. Expected: sequential execution." --providers opencode --target-paths src/agents/concurrency,src/agents/race-condition.test.ts --json &

wait

# Synthesize
mco run --repo . --prompt "Synthesize results from 3 parallel fixes. Confirm all 6 tests pass." --providers claude --json
```

## Benefits

- **Speed** — N agents working simultaneously vs. sequential
- **Focus** — each agent has minimal, relevant context
- **Isolation** — separate worktrees prevent interference
- **Quality** — focused context + provider rotation = better solutions

## OpenClaw Advantages

OpenClaw's multi-agent architecture makes this pattern especially powerful:
- `mco run &` provides true process isolation
- Provider rotation prevents bias
- `sessions_history` enables result collection without polling
- Cron can monitor long-running parallel agents and notify on completion
