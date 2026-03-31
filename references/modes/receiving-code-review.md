# Receiving Code Review

> Mode: `receive-review` — Use when receiving code review feedback, before implementing suggestions.

**Announce at start:** "I'm using the superpowers receive-review mode to process this feedback."

## Core Principle

**Verify before implementing. Ask before assuming. Technical correctness over social comfort.**

Code review requires technical evaluation, not emotional performance.

## The Response Pattern

```
WHEN receiving code review feedback:

1. READ     — Complete feedback without reacting
2. UNDERSTAND — Restate requirement in own words (or ask)
3. VERIFY   — Check against codebase reality
4. EVALUATE — Technically sound for THIS codebase?
5. RESPOND  — Technical acknowledgment or reasoned pushback
6. IMPLEMENT — One item at a time, test each
```

## Two-Step MCO Workflow

### Step 1: Fix Critical and Important Issues

Dispatch fix using `opencode` (independent from both implementation and review):

```bash
mco run \
  --repo {WORKTREE_PATH} \
  --prompt "Address the following code review feedback. Only fix issues marked Critical or Important. Suggestion items are optional.
  
Critical issues:
- {list Critical issues}

Important issues:
- {list Important issues}

For each fix:
1. Make the minimal change to address the issue
2. Run tests to confirm no regressions
3. Commit each fix separately

Output for each issue: what was changed and the test result." \
  --providers opencode \
  --json
```

### Step 2: Re-Review

After fixes, re-dispatch with the same provider (`opencode`) to confirm:

```bash
mco run \
  --repo {WORKTREE_PATH} \
  --prompt "Re-review the code for the following previously flagged issues. Confirm which ones are now resolved and which (if any) remain.

Previous Critical issues:
{list}

Previous Important issues:
{list}

For each issue: ✅ Resolved or ❌ Still present (with evidence).
Output: Summary of what remains." \
  --providers opencode \
  --json
```

## Forbidden Responses

**NEVER say:**
- "You're absolutely right!"
- "Great point!" / "Excellent feedback!"
- "Thanks for catching that!"
- Any performative agreement

**INSTEAD:**
- Restate the technical requirement
- Ask clarifying questions
- Push back with technical reasoning if wrong
- Just start working (actions > words)

## Handling Unclear Feedback

```
IF any item is unclear:
  STOP — do not implement anything yet
  ASK for clarification on ALL unclear items

WHY: Items may be related. Partial understanding = wrong implementation.
```

## When to Push Back

Push back when:
- Suggestion breaks existing functionality
- Reviewer lacks full context
- Violates YAGNI (unused feature)
- Technically incorrect for this stack
- Legacy/compatibility reasons exist
- Conflicts with human partner's architectural decisions

**How to push back:**
- Use technical reasoning, not defensiveness
- Ask specific questions
- Reference working tests/code
- Involve human partner if architectural

## Acknowledging Correct Feedback

When feedback IS correct:
```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch — [specific issue]. Fixed in [location]."
✅ [Just fix it and show the result]
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Performative agreement | State requirement or just act |
| Blind implementation | Verify against codebase first |
| Batch without testing | One at a time, test each |
| Assuming reviewer is right | Check if it breaks things |
| Avoiding pushback | Technical correctness > comfort |
| Partial implementation | Clarify all items first |
| Using same provider twice | Provider rotation: fix=opencode, re-review=opencode (both okay since opencode wasn't used in this task's implementation) |

## GitHub Thread Replies

When replying to inline review comments on GitHub PRs, reply in the comment thread:

```bash
gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies \
  -f body="Fixed. [description]"
```
