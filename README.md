# softkaze-skills

Skills that teach an AI agent to work with the applications of [Softkaze](https://softkaze.com). They follow the open
Agent Skills format: a folder with a `SKILL.md`, which serves Claude Code, Codex, Gemini CLI, Cursor, GitHub Copilot
and others without changes.

## Skills

| Skill | What it is for |
| --- | --- |
| [todomarkdown](plugins/todomarkdown/skills/todomarkdown/SKILL.md) | Read and edit [TodoMarkdown](https://apps.microsoft.com/detail/9np39wtpb45t) to do lists (`.tdmd` files) |
| [notesinging](plugins/notesinging/skills/notesinging/SKILL.md) | Coach singing with NoteSinging: write its exercises, review the takes it records and check the voice |

## Install

### Claude Code, as a plugin

```
/plugin marketplace add yuio/softkaze-skills
/plugin install todomarkdown@softkaze-skills
/plugin install notesinging@softkaze-skills
```

To receive the changes by themselves, turn on the auto-update of the marketplace in `/plugin` > Marketplaces.

### Any agent, by hand

Copy the folder of the skill, with its name, to the folder of skills of the user:

- `~/.claude/skills/<skill>` for Claude Code
- `~/.agents/skills/<skill>` for Codex, Gemini CLI, Cursor, GitHub Copilot and Grok

### Any chat

Paste the text of the `SKILL.md` in the conversation, or give its address:

```
https://raw.githubusercontent.com/yuio/softkaze-skills/main/plugins/todomarkdown/skills/todomarkdown/SKILL.md
```

## About this repository

The skills are written next to the code of each application and copied here when a version is published: changes
made here by hand are lost.
