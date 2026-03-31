# MCO-Superpowers

Structured development workflow for multi-agent coding systems. Originally ported from [obra/superpowers](https://github.com/obra/superpowers) for OpenClaw agents, adapted for any MCO-powered development pipeline.

**One sentence**: Give your coding agents a structured development discipline — from brainstorming to shipped code, through evidence-based, test-driven methods.

---

## What It Does

Superpowers provides **13 workflow modes** that kick in at the right moment:

| Mode | Trigger |
|------|---------|
| `brainstorming` | New feature, design, creative work |
| `writing-plans` | Spec approved, time to plan |
| `executing-plans` | Run a plan without MCO |
| `subagent-dev` | Run a plan with MCO + two-stage review |
| `parallel-dispatch` | 3+ independent tasks in parallel |
| `tdd` | Writing any code (always active) |
| `debugging` | Bug or test failure |
| `verification` | Before claiming "done" |
| `git-worktrees` | Isolated development workspace |
| `finish-branch` | Ready to merge or PR |
| `request-review` | Need code review |
| `receive-review` | Got review feedback |
| `writing-skills` | Creating agent skills |

## Core Principle

> **Invoke relevant modes BEFORE any response or action.** Even a 1% chance a mode might apply → load and check it. The cost of checking is always lower than the cost of skipping.

## Standard Workflow

```
brainstorming → writing-plans → [subagent-dev | executing-plans]
                    ↑                            ↓
               plan review            per-task: implement → spec review
                                                             ↓
                                                    quality review → commit
                    ↓
            request-review → finish-branch
```

Throughout: `tdd` (write tests first), `verification` (prove it works), `debugging` (when broken).

## Quick Start

### For OpenClaw Agents

```bash
# Install as a skill
cp -r . ~/.openclaw/skills/superpowers/

# The agent activates modes automatically based on context.
# No manual invocation needed.
```

### For Any MCO Project

Reference the `references/` directory for prompt templates:

```
references/
├── modes/         # 13 workflow mode definitions
└── prompts/       # agent dispatch prompts
    ├── implementer-prompt.md
    ├── spec-reviewer-prompt.md
    ├── code-quality-reviewer-prompt.md
    └── ...
```

## File Structure

```
mco-superpowers/
├── SKILL.md                    # OpenClaw skill definition
├── README.md
├── references/
│   ├── modes/                 # 13 mode definitions
│   │   ├── brainstorming.md
│   │   ├── writing-plans.md
│   │   ├── test-driven-development.md
│   │   ├── systematic-debugging.md
│   │   └── ... (13 total)
│   └── prompts/               # agent dispatch templates
│       ├── implementer-prompt.md
│       ├── spec-reviewer-prompt.md
│       └── ...
├── assets/
│   └── tracker-template.md     # task tracking template
└── scripts/
```

## Key Features

- **TDD by default** — no production code without a failing test first
- **Two-stage review** — spec compliance + code quality, in that order
- **Provider rotation** — implement→claude, spec→codex, quality→opencode
- **File-based tracker** — persists across sessions, no lost state
- **Git worktree isolation** — never develop on main
- **Evidence-based completion** — run commands, show output, then claim done

## Provider Rotation Strategy

Avoids the same agent reviewing its own work:

| Stage | Provider |
|-------|----------|
| Implementation | `claude` |
| Spec Review | `codex` |
| Quality Review | `opencode` |
| Final Code Review | `claude` |

## License

MIT — use it, adapt it, ship with it.
