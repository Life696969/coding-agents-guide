# Track 3 — A second brain

**Prerequisites:** Track 2 (permanent memory) helps but is not required. Nothing to
install.
**Time:** about 35 minutes.
**They end up with:** a knowledge base their agent writes into and reads out of, that
survives sessions and works across every project they open.

---

> **Say this before you start.** Claude Code ships an *auto memory* that already does
> part of this: it writes its own typed notes into a per-project memory directory and
> keeps a `MEMORY.md` index that loads at the start of every session. If all they want
> is "remember my corrections", that is built already and they should just turn it on.
>
> This track is still worth doing, for two reasons. The built-in memory is **per
> repository and machine-local**; a second brain is one body of knowledge that follows
> them across every project and syncs in git. And building the index by hand is what
> teaches them *why* retrieval works — which is what lets them fix the built-in one on
> the day it starts pulling the wrong note.
>
> Teaching someone to hand-build a thing their tool already does, without mentioning
> that it already does it, is how a guide loses their trust.

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

Four types is enough: `decision`, `preference`, `reference`, `gotcha`.

**CHECK:** the `description` must be good enough to decide *relevance* on its own,
because that is all the agent reads when scanning. "Redis notes" fails. The one
above passes.

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

The agent loads the index every session — cheap, it is one line each — and opens
only the notes that turn out to matter. That is the whole retrieval mechanism, and
it works because the hooks are written to be *scannable*, not complete.

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
| Did not look | The instruction in the user memory file is too weak, or the index is not being read |
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
retrieval does, then test. Most people are surprised by how completely it breaks —
which is the lesson: **the index is the brain, the notes are just storage.**

---

## What next

- **Build your own skill** (track 4) — package the capture routine so it is one command
- **Deploying subagents** (track 1) — give twenty agents your accumulated judgement at once
