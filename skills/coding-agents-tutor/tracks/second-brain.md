# Track 3 — A second brain

**Prerequisites:** Track 2 (permanent memory) helps but is not required. Nothing to
install.
**Time:** about 35 minutes.
**They end up with:** a knowledge base their agent writes into and reads out of, that
survives sessions and works across every project they open.

---

> **Say this before you start, and be straight about it.** Claude Code already ships an
> *auto memory*: it writes its own typed notes into `~/.claude/projects/<project>/memory/`
> and keeps a `MEMORY.md` index that loads at the start of every session (the first
> 200 lines or 25KB — past that is not loaded, though Claude Code does warn as the file
> approaches the limit). **It is on by default.** Have them run `/memory`, open the auto
> memory folder, and actually look at what it has been saving. If all they want is
> "remember my corrections", they already have it.
>
> The honest reason to build one anyway is **portability**. The built-in is per
> repository and machine-local by default: it does not follow you across projects and
> it does not sync in git. Those defaults can be moved — `autoMemoryDirectory`
> relocates the store, and `CLAUDE_CODE_PROJECT_DIR_NAME` can make projects share one,
> though only when `CLAUDE_CONFIG_DIR` is set alongside it — but a folder of markdown
> you own is portable across tools, machines and years, which none of that gives you.
>
> Be careful not to oversell the gap. Auto memory's `feedback` and `project` types
> cover corrections, preferences and non-obvious decisions — most of what you are about
> to write down. It skips what it can *derive from the codebase*. So this is a
> portability-and-ownership argument, not a "the built-in can't do it" argument.
>
> And building the index yourself teaches you why retrieval works, which is what lets
> you fix the built-in one on the day it starts pulling the wrong note.

## What they must understand by the end

Track 2 made the agent remember **a project**. This makes it remember **them**.

The difference is what the notes are for. Project memory answers *"how does this
repo work?"* A second brain answers *"what have I already figured out, and what did
I decide last time I was here?"*

The trap is thinking a second brain is a big pile of notes. It is not. It is a
small pile of notes with **good retrieval**. A thousand notes the agent cannot find
is worse than thirty it can.

---

## Beat 1 — find out what they keep losing

Do not start with structure. Start with pain. Ask:

> Think about the last month. What did you have to work out twice because you did
> not write it down the first time?

Get three real answers. Those three become the first three notes. Everything in
this track is built to catch that specific kind of thing, not to be a general
filing system.

**TRAP:** people build the taxonomy first and fill it never. The folder structure is
the easy part and it is worthless empty.

---

## Beat 2 — one fact per file

The unit is a single file holding a single fact, with enough front matter for the
agent to judge relevance without opening it.

```markdown
---
name: why-we-dropped-redis
description: Redis was removed from the pipeline in Aug 2026 because the cache hit
  rate never went above 12%.
type: decision
---

Measured over three weeks: 12% hit rate, and the invalidation logic was the source
of two outages. Replaced with an in-process LRU.

**Why it matters:** if someone proposes Redis again, this is the measurement that
says no. The hit rate is the number to re-check, not the outages.

Related: [[pipeline-architecture]]
```

Four types is enough: `decision`, `preference`, `reference`, `gotcha`. **These are
yours, not Claude Code's** — auto memory uses `user`, `feedback`, `project` and
`reference`, and its `reference` means something narrower (where to find an external
dashboard or tracker). Do not assume they interoperate.

**CHECK:** write the `description` as if it were the only thing anyone reads — because
when it comes to *choosing* what to open, the index line is genuinely all the agent
has. The frontmatter is not in context until the file is opened. Keeping the two in
sync is what makes the index line easy to write. "Redis notes" fails. The one above
passes.

Have them write their three from Beat 1 in this shape.

---

## Beat 3 — the index is the retrieval

This is the part people skip, and it is the part that makes it work.

One `MEMORY.md` at the root of the brain. One line per note. Nothing else — never
content, never a second copy.

```markdown
- [Why we dropped Redis](why-we-dropped-redis.md) — 12% hit rate, not worth the invalidation bugs
- [Prefers tables over prose](prefers-tables.md) — for anything comparative
- [Windows path gotcha in ffmpeg](ffmpeg-windows-paths.md) — backslashes break the filter graph
```

You want the index in context every session — cheap, one line each — so the agent can
open only the notes that matter. **Nothing loads `~/brain/MEMORY.md` on its own**, so
make it load: put an `@` import in `~/.claude/CLAUDE.md`, which is expanded into
context at launch.

Add this line to `~/.claude/CLAUDE.md`, **bare and not inside backticks or a code
fence** — import parsing skips code spans and fenced blocks, so a tidied-up version in
backticks silently does nothing:

    @~/brain/MEMORY.md

Imports in a user-scope file load without an approval prompt. One exception: in Cowork
desktop sessions, a user-scope import resolving outside the session's working directory
is skipped, so there the import will not fire.

That handles **retrieval**. Beat 4 adds a prose instruction, which handles **capture** —
when to write a new note. The two do different jobs; you want both.

**CHECK:** the index line must make sense to someone who has not read the note. If
the hook is just the title again, it is not earning its place.

---

## Beat 4 — make the agent write, not you

A brain you have to maintain by hand dies. Have them add this to their user-level
memory file (`~/.claude/CLAUDE.md`):

```markdown
## Second brain
Notes live in `~/brain/`. `MEMORY.md` there is the index; read it when a question
might have been answered before.

Write a new note when I decide something non-obvious, state a preference about how
I want things done, or hit a gotcha that cost more than ten minutes. One fact per
file, add one line to MEMORY.md. Check for an existing note first and update that
instead of duplicating. Never record what the code or git history already says.
```

Then have them work normally for a bit and watch whether notes appear.

**TRAP:** auto memory is running at the same time, with an overlapping mandate — the
agent will write to both stores and the learner will see notes appear in
`~/.claude/projects/<project>/memory/` that they did not build. Either turn it off for
this exercise — `"autoMemoryEnabled": false` in that project's `.claude/settings.json`,
or `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` — or tell them to expect the split. And note
that a `CLAUDE.md` instruction is context, not enforcement: it asks, it does not
compel, which is why the *index* is imported rather than requested.

**TRAP:** it will over-record at first — every trivial thing becomes a note. That is
fine and it is fixable: tighten the trigger ("cost more than ten minutes"), and
delete the junk. A brain that never prunes becomes noise, and noise is what makes
retrieval fail.

---

## Beat 5 — retrieval, honestly tested

Have them open a fresh session **in a different project** and ask something one of
their notes answers — phrased differently from how the note is written.

Three outcomes and what each means:

| What happened | What to fix |
|---|---|
| Found and used it | Nothing. Do it again next week to be sure. |
| Did not look | The `@` import is missing, mistyped, or wrapped in backticks so it never parsed |
| Looked but picked wrong | The `description` hooks are too similar to each other |

**CHECK:** make them run this test with a question they did *not* design the note
for. Testing retrieval with the note's own words proves nothing.

---

## The project — a brain that answers a real question

> Build `~/brain/` with **at least eight notes** drawn from things you actually
> know: decisions you have made, preferences about how you work, gotchas that cost
> you time. Write `MEMORY.md`. Wire up the instruction in your user memory file.
>
> Then ask your agent a question you have genuinely asked yourself before, in a
> fresh session, in an unrelated project — and see if it answers from your notes.

Rules:

- Eight real notes. Eight invented ones teach nothing, because retrieval only gets
  hard when the notes are similar to each other.
- At least two must be **preferences** (how you like work done), not facts. Those
  are the ones that change the agent's behaviour rather than its answers.
- At least one `[[link]]` between two notes.

**CHECK:** have them ask a question that should match *nothing*, and confirm the
agent says it does not know rather than stretching an unrelated note to fit. A brain
that always finds something is a brain that cannot be trusted.

---

## Beat 6 — where this actually goes wrong

- **Staleness beats volume.** Thirty current notes beat three hundred where a
  tenth are wrong. Wrong notes are actively harmful — the agent believes them.
- **It is not a diary.** No "today I worked on X". If it does not change a future
  decision, it is not a note.
- **Private things stay private.** This gets read by an agent and may be sent to a
  model provider. Nothing in it should be anything they would not paste into a chat.
- **Portability.** It is plain markdown in a folder. That is deliberate — it works
  with any agent, syncs in git, and is readable when the tool changes.

**Predict-then-run:** have them delete `MEMORY.md` (keeping the notes), predict what
retrieval does, then test. It does **not** collapse — the folder is still named in
their memory file and the agent has Glob and Grep, so it will often still find the
right note by filename or full-text match. What degrades is precision and speed: it
misses notes whose filenames do not echo the question, and it reads more to get there.
That is the lesson — **the index is not storage, it is the precision layer**, and you
feel its absence as noise rather than as failure.

---

## What next

- **Build your own skill** (track 4) — package the capture routine so it is one command
- **Deploying subagents** (track 1) — give twenty agents your accumulated judgement at once
