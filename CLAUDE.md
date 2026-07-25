# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

A community-shareable collection of Claude Code resources — skills, agents, and other reusable building blocks. There is no application code, no build system, and no test suite; the content itself (skill/agent definitions) is the deliverable.

## Structure

Each skill lives in its own directory under `skills/` with a `SKILL.md` file that starts with YAML frontmatter:

```yaml
---
name: skill-name
description: One-line description of what the skill does and when it should be used — be specific about trigger conditions.
---
```

The body is the instructions Claude follows when the skill is invoked. See `skills/programming-with-great-fundamentals/SKILL.md` for the existing example: a checklist (XP, System Design, SOLID, Hexagonal Architecture, DDD, DS&A, Clean Code) meant to be walked through before writing non-trivial code and re-checked after, skipping sections that don't apply to the task at hand.

## Working in this repo

- Changes here are almost always edits to a `SKILL.md` file, or adding a new skill/agent directory — there's nothing to compile, lint, or test.
- Keep the frontmatter `description` field specific and trigger-oriented, since that's what a Claude session uses to decide whether to load the skill.
- Skill bodies should read as instructions to an AI agent, not documentation for a human reader — write imperative guidance, not prose explanation.
- Update the skills table in `README.md` when adding, renaming, or removing a skill.
