# MCO Provider Setup

> Mode: `setup` — Use when configuring MCO providers for the first time, or when user says "setup", "configure", or "配置".

**Announce at start:** "I'm using the superpowers setup mode to configure your MCO providers."

## When to Trigger

Activate this mode when:
1. User explicitly says "setup superpowers", "配置 superpowers", or "configure MCO providers"
2. `providers.json` does not exist in the skill directory AND a subagent-dev or executing-plans task is about to run
3. User says "change provider", "换 provider", or "修改 MCO 配置"

## Core Concept

**Interactive configuration of MCO coding agent providers.** The setup guides the user through a small number of questions to produce a `providers.json` file. This file is read by subagent-driven-development and parallel-dispatch modes — no SKILL.md modification needed.

## The Process

### Step 1: Greeting and Purpose

Say:
> "I'll help you configure your MCO (Multi-Agent Coding Orchestration) providers. This determines which AI providers handle implementation, spec review, and quality review tasks. You can change these settings anytime by editing `providers.json`."

### Step 2: Available Providers (Q1)

Ask ONE question:
> "Which providers do you want to use? Select by letter(s), e.g. `A,C` or `A,B,C,D`.\n\nA) **claude** — Anthropic Claude (strongest, recommended for implementation)\nB) **codex** — OpenAI Codex (stable, good for review)\nC) **opencode** — OpenCode / GPT-5-nano (free/cheap option)\nD) **gemini** — Google Gemini (backup)"

**Record answer.** If user picks fewer than 2, note that at least 2 providers are recommended for rotation.

### Step 3: Parallel Order (Q2)

Ask ONE question:
> "What order should providers be used for parallel tasks? (e.g. `A,B,C` means claude first, then codex, then opencode).\n\nDefault order: `A,B,C`\nYour selection: _"

Accept default or custom order. Validate that all selected providers are in Q1's list.

### Step 4: Stage Assignment Confirmation (Q3)

Confirm the three-stage assignment based on selections:
> "Here's your MCO provider configuration:\n\n- **Implementation** → {provider A}\n- **Spec Review** → {provider B}\n- **Quality Review** → {provider C}\n\nAccept this assignment? Reply `yes` to confirm, or tell me which stage to change."

If user wants to change: ask which stage and what provider.

### Step 5: Write Config File

After confirmation, write to `~/.openclaw/skills/superpowers/providers.json`:

```json
{
  "implementation": "{provider-key-1}",
  "specReview": "{provider-key-2}",
  "qualityReview": "{provider-key-3}",
  "parallelProviders": ["{key-1}", "{key-2}", "{key-3}"]
}
```

### Step 6: Confirm and Report

Say:
> "✅ MCO provider configuration saved to `providers.json`.\n\n**Current config:**\n- Implementation → {impl}\n- Spec Review → {spec}\n- Quality Review → {quality}\n\nTo change later: just say 'configure MCO providers' again, or edit `~/.openclaw/skills/superpowers/providers.json` directly."

## Configuration File Location

The config file is always at:
```
~/.openclaw/skills/superpowers/providers.json
```

**Never put API keys or secrets in this file.** Only provider names (e.g. `claude`, `codex`, `opencode`).

## Fallback

If user cancels or says "skip", say:
> "Setup cancelled. The default providers (claude / codex / opencode) will be used. You can run `superpowers setup` anytime to reconfigure."

## Anti-Patterns

- ❌ Asking all questions at once (one at a time, wait for answer)
- ❌ Writing the file before user confirmation
- ❌ Including API keys or tokens in the config
- ❌ Skipping the setup even when providers.json is missing

## Files

- Input: User's provider preferences
- Output: `~/.openclaw/skills/superpowers/providers.json`
- Template: `providers.json.default` (reference only)
