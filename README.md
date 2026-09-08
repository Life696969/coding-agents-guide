# The Intermediate Coding Agents Guide

**For people who already use a coding agent and have hit the ceiling.**

You can open Claude Code or Codex, ask for a change, and get one. But you are still
working one prompt at a time, still re-explaining your project every session, and
the agent still forgets everything the moment you close the window.

That ceiling is not the model's. It is an information problem. This repo has the fix
in two forms — one to read, one to do.

---

## What is in here

| | What it is |
|---|---|
| **[`guide/`](guide/)** | A PDF and its markdown source. Read it in one sitting. Covers subagents, permanent memory, and using an agent as a second brain. |
| **[`skills/coding-agents-tutor/`](skills/coding-agents-tutor/)** | A skill you install into Claude Code or Codex. It turns your agent into a teacher: it asks what you want to learn, then walks you through building it for real. |

The guide is the reading. The skill is the doing. **The skill is the part that
actually changes how you work**, because you finish each track with something that
runs.

---

## Install the skill

**Claude Code**

```bash
git clone https://github.com/Life696969/coding-agents-guide.git
mkdir -p ~/.claude/skills
cp -r coding-agents-guide/skills/coding-agents-tutor ~/.claude/skills/
```

The `mkdir` is not optional. Without it `cp` exits 0 and quietly makes
`~/.claude/skills` *itself* the skill folder — you get `~/.claude/skills/SKILL.md`
instead of `~/.claude/skills/coding-agents-tutor/SKILL.md`, and the skill never loads,
with no error.

**Codex**

```bash
mkdir -p ~/.agents/skills
cp -r coding-agents-guide/skills/coding-agents-tutor ~/.agents/skills/
```

On Windows, copy the folder `skills\coding-agents-tutor` into `%USERPROFILE%\.claude\skills\`.

Claude Code watches the skills directory, so it normally picks the new skill up
**without a restart** — *unless `~/.claude/skills/` did not exist when your session
started*, which is the usual case the first time. Then it is not watching that
directory yet and you need one restart. Codex detects changes automatically too.

Now just say:

> teach me about coding agents

It will ask what you want to learn and give you the list.

---

## The seven tracks

Each one ends in a project that runs. Pick one, do it properly, come back for the
next.

| # | Track | Needs | What you build |
|---|---|---|---|
| 1 | **Deploying subagents** | nothing | A review that runs six agents at once over your repo and writes one report |
| 2 | **Permanent memory** | nothing | A setup where a brand-new session already knows your project |
| 3 | **A second brain** | nothing | A knowledge base your agent writes into and reads out of, across every project |
| 4 | **Build your own skill** | nothing | A skill of your own that fires without you naming it |
| 5 | **Hooks and guardrails** | Python | A rule the agent physically cannot break |
| 6 | **Editing video with an agent** | ffmpeg | A frame-exact edit that re-runs when you change one number |
| 7 | **3D with Blender** | Blender | A scene generated and rendered entirely by script |

Tracks 1–4 need nothing but the agent you already have. Start there.

---

## The one idea, if you read nothing else

**An agent does not remember. It reads.**

Every session, every subagent, every parallel run starts from nothing and rebuilds
its understanding from two things: the prompt you gave it, and the files it can
open.

Once that lands, three problems turn out to be one problem:

| What it feels like | What it actually is |
|---|---|
| "It forgets everything between sessions" | Nothing wrote the facts to a file it reads at startup |
| "Parallel agents give inconsistent results" | Each one was briefed differently, or not at all |
| "It keeps making the mistake I corrected last week" | The correction went into a chat log, which was thrown away |

So the work is not better prompting.

> **You scale agents by improving the files they read, not by writing longer
> prompts.** A prompt runs once. A file runs forever.

---

## Read the guide

- **[`guide/intermediate-coding-agents.pdf`](guide/intermediate-coding-agents.pdf)** — the one to send someone
- **[`guide/intermediate-coding-agents.md`](guide/intermediate-coding-agents.md)** — same thing, readable here on GitHub

---

## Verified

Everything was checked on **8 September 2026**, on Windows 11, with **Claude Code
2.1.202** and **ffmpeg 8.1**. Every claim about hooks, memory, subagents and skills
was checked against the official documentation on that date rather than written from
memory.

Tracks 6 and 7 need tools that are not part of any agent — ffmpeg and Blender — and
each track checks for them before it starts rather than four beats in.

Agent behaviour changes between versions. If something here does not match what you
see, trust what you see and check the official docs:
[Claude Code](https://code.claude.com/docs), [agents.md](https://agents.md/).

---

## Licence

MIT. Take it, change it, teach it to someone else.

---

Made by **Mudit Jain** — [@ai_with_mudit](https://instagram.com/ai_with_mudit).

There is a beginner guide too, for people who are not at this level yet. DM
**`code`** on Instagram for that one, or **`inter`** for this one.
