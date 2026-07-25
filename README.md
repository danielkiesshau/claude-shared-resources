# Claude Shared Resources

A community-shareable collection of SWE-focused resources for [Claude Code](https://claude.ai/code): skills, agents, and other reusable building blocks.

## Goal

Make it easy to discover, reuse, and contribute Claude Code resources — skills, agents, and more — instead of everyone reinventing them privately.

## Contents

- `skills/` — custom skills and agents that can be dropped into any Claude Code project.

## Skills

| Skill | Description |
| --- | --- |
| [programming-with-great-fundamentals](skills/programming-with-great-fundamentals/SKILL.md) | Checklist of XP, System Design, SOLID, Hexagonal Architecture, DDD, DS&A, and Clean Code principles to apply when designing, writing, or reviewing non-trivial code. |

## Agents

| Agent | Description |
| --- | --- |
| [tech-docs-agent](skills/tech-docs-agent/tech-docs-agent.md) | Answers technical Software Engineering questions grounded in official documentation rather than memory, with links to every doc page used. |

## Usage

Copy a skill's directory into your project's `skills/` folder (or your global `~/.claude/skills/`), and Claude Code will pick it up automatically based on its `description` frontmatter.

## Contributing

Contributions are welcome — new skills, agents, or improvements to existing ones. Each skill should live in its own directory under `skills/` with a `SKILL.md` file, starting with frontmatter:

```yaml
---
name: skill-name
description: One-line description of what the skill does and when it should be used.
---
```
