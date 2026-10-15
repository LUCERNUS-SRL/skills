# Lucernus Skills

Skills we use at Lucernus to keep agent-written code reviewable and grounded in real-world patterns.

They follow the open [Agent Skills](https://agentskills.io) format, so they work in Claude Code, Codex, and any other agent that reads `SKILL.md`.

## Installation

```bash
npx skills@latest add LUCERNUS-SRL/skills
```

The installer asks which skills to take, which agents to install them for, and whether to install them in the current project or globally for every project.

To install a single skill:

```bash
npx skills@latest add LUCERNUS-SRL/skills --skill=<name>
```

To update, run `npx skills@latest update`, and re-run `add` to pick up new skills.

## Why these skills exist

An agent can write code faster than a person can review it. That speed only helps if a human can still read the result, judge it, and own it once it ships.

Left alone, an agent also falls back on its own habits. It invents structure where proven conventions already exist, and its choices change from one run to the next.

These skills push the other way. They make the agent produce work a reviewer can follow, and they tie its decisions to patterns that have held up in real codebases rather than to its own taste.

## Skills

**User-invoked** skills run only when you type their name, for example `/name`. **Model-invoked** skills can also be picked up by the agent on its own when a task calls for them.

**User-invoked**

- **[clean-implement](./skills/clean-implement/SKILL.md)**: Brief you on a ticket, let you choose its direction and slice it, then build one slice per session: a test-first draft, refined in passes, reviewed by you, landed as one commit.

**Model-invoked**

- **[clean-code](./skills/clean-code/SKILL.md)**: Clean Code smells, and the pass that finds them in a diff or the code you name. clean-implement runs it on every slice.
