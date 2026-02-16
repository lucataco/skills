# Replicate Skills

A collection of [Agent Skills](https://agentskills.io) for building AI-powered apps with [Replicate](https://replicate.com). 

Discover, compare, and run AI models using Replicate's API.

## Installing

These skills work with any agent that supports the Agent Skills standard, including Claude Code, OpenCode, OpenAI Codex, and Pi.

### Claude Code

Install using the [plugin marketplace](https://code.claude.com/docs/en/discover-plugins#add-from-github):

```
/plugin marketplace add replicate/skills
```

### OpenCode

OpenCode automatically discovers Claude Code skills installed under `~/.claude/skills/`.

You can also copy `skills/replicate` into `~/.config/opencode/skills/replicate`.

### npx skills

Install using the [`npx skills`](https://skills.sh) CLI:

```
npx skills add https://github.com/replicate/skills
```

### Clone / Copy

Clone this repo and copy the skill folders into the appropriate directory for your agent:

| Agent | Skill Directory | Docs |
|-------|-----------------|------|
| Claude Code | `~/.claude/skills/` | [docs](https://code.claude.com/docs/en/skills) |
| OpenCode | `~/.config/opencode/skills/` | [docs](https://opencode.ai/docs/skills/) |
| OpenAI Codex | `~/.codex/skills/` | [docs](https://developers.openai.com/codex/skills/) |
| Pi | `~/.pi/agent/skills/` | [docs](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent#skills) |
