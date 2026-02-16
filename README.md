# Replicate Skills

A collection of [Agent Skills](https://agentskills.io) for building AI-powered apps with [Replicate](https://replicate.com). 

Discover, compare, and run AI models using Replicate's API.

## Installing

These skills work with any agent that supports the [Agent Skills standard](https://agentskills.io/specification), including Claude Code, OpenCode, OpenAI Codex, and Pi.

### npx skills

Install using the [`npx skills`](https://skills.sh) CLI:

```
npx skills add https://github.com/replicate/skills
```

### Claude Code

Install using the [plugin marketplace](https://code.claude.com/docs/en/discover-plugins#add-from-github):

```
/plugin marketplace add replicate/skills
```

### OpenCode

OpenCode automatically discovers Claude Code skills installed under `~/.claude/skills/`.

You can also copy `skills/replicate` into `~/.config/opencode/skills/replicate`.