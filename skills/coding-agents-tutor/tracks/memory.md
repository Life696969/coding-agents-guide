# Track 2 — Permanent memory

**Prerequisites:** Claude Code or Codex. Nothing to install.
**Time:** about 25 minutes.
**They end up with:** a memory setup where the agent starts a brand-new session
already knowing their project, their conventions, and the decisions they have
already made.

---

## What they must understand by the end

**The agent has no *conversational* memory. Everything that persists is a file.**

Claude Code has two such mechanisms, and it is worth knowing both before you build
anything:

- **`CLAUDE.md`** — instructions *you* write. This track is about these.
- **Auto memory** — notes *Claude* writes itself from your corrections, kept per
  repository and loaded every session. **It is on by default.** Run `/memory` to see
  what it has already been saving.

Once you see memory as "files that get read", every memory problem becomes a file
problem: what is in it, how long it is, and whether it is still true.

---

## Beat 1 — see the cost of no memory

Have them open a **fresh** session in a project they have used the agent on before,
and ask:

> "What do you know about this project's conventions?"

Whatever it answers came from files — `CLAUDE.md` if they had one, plus whatever
auto memory has quietly saved. Have them run `/memory` too, and notice the split:
some of what they explained last week *was* captured, and the rest is simply gone.

**TRAP:** if you skip the `/memory` step, this beat can backfire — auto memory may
answer well enough that the learner concludes they do not need any of this.

**Then ask:** how much of your last five sessions was re-explaining the same
things?

That number is what this track is worth to them.

---

## Beat 2 — the file the agent already looks for

**Claude Code** reads `CLAUDE.md`. **Codex and ~30 other tools** read `AGENTS.md`.
Both at the repo root, both plain markdown, no schema, no install.

Have them create one with only these four sections, and nothing else yet:

```markdown
# <project>

## What this is
One paragraph. What it does and who uses it.

## Commands
- build: <the exact command>
- test: <the exact command>
- run: <the exact command>

## Conventions
Three to five rules that are actually true here.

## Do not touch
Files or areas that need a human.
```

**TRAP:** the instinct is to write everything they know. Do not. A memory file is
read *every session* and competes with the actual work for the context window. Long
file, worse adherence. Aim well under 200 lines; if it is over, something in it
belongs somewhere else.

**CHECK:** the "Commands" entries must be commands they can paste and run right
now. "run the tests" is not a command. `npm test -- --run` is.

---

## Beat 3 — the Claude Code catch

If they use Claude Code, this is the thing that trips almost everyone:

**Claude Code reads `CLAUDE.md`. It does not read `AGENTS.md`.**

So if they wrote `AGENTS.md` for Codex or Cursor, Claude Code will ignore it. The
fix is not to maintain two copies — it is a two-line bridge:

```markdown
@AGENTS.md

## Claude Code
Anything that is only true for Claude goes below this line.
```

One source of truth in `AGENTS.md`, imported. Have them do this if they use both
tools; skip if they only use one.

**CHECK:** run `/context` in a fresh session and confirm `CLAUDE.md` appears under
**Memory files**. That is the direct answer to "did it load", and it beats inferring
it from the agent's behaviour.

---

## Beat 4 — the hierarchy, and which file wins

There is more than one place memory lives:

| Where | Scope | Use it for |
|---|---|---|
| `~/.claude/CLAUDE.md` | every project you open | how *you* like to work |
| `<repo>/CLAUDE.md` | this project | what this project is |
| `<repo>/<subdir>/CLAUDE.md` | loaded when Claude reads files there | rules local to that area |

**They do not override each other — they are all concatenated into context**, broadest
first. So there is no "more specific wins" rule to lean on: if two files genuinely
contradict, the agent picks one, and which one is not something you can predict.
Treat a contradiction as a bug to delete, not a precedence puzzle to solve.

For rules that should only apply to part of a repo, the cleaner tool is
`.claude/rules/` with a `paths:` frontmatter glob — those load only when Claude
**reads** a matching file, so they cost nothing the rest of the time. Note the verb:
a rule scoped to `src/api/**` is *not* in context while Claude is creating a brand
new file there, which is a trap if you scope rules to an area you are about to build.

**This beat is Claude Code only.** Codex has no `~/.claude/CLAUDE.md` hierarchy and no
`.claude/rules/`.

Have them put one genuinely personal preference in the user-level file — something
true across all their projects, like "explain before you refactor" — and confirm it
shows up in a different repo.

**TRAP:** putting project facts in the user-level file. It follows them into every
unrelated repo and quietly makes the agent worse everywhere else.

---

## Beat 5 — memory that updates itself

A file only they edit goes stale in a fortnight. The version that survives is the
one the agent maintains.

Have them add this to the memory file:

```markdown
## Decisions
When we decide something that is not obvious from the code — a library choice, a
pattern we rejected, a constraint from outside the repo — append it here as one
line with the date and the reason. Do not record what the code already says.
```

Then have them make a small real decision and ask the agent — **in these words** —
to *"add this to CLAUDE.md"*.

**TRAP:** an unqualified "remember this" goes to **auto memory**, not to the section
they just created. They will then open `CLAUDE.md`, find nothing, and think it broke.
Say the file name.

**CHECK:** read the line it wrote, in `CLAUDE.md`. Does it capture *why*? "Chose
Postgres" is useless. "Chose Postgres over SQLite — needs concurrent writes from two
services" is memory.

**TRAP:** recording things the repo already tells you. "This project uses React" is
visible from `package.json`; writing it down costs context every session and earns
nothing. Memory is for what is **not** derivable.

---

## The project — survive a cold start

> Set up memory in a real project of yours. **Turn auto memory off first** — set
> `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`, or `"autoMemoryEnabled": false` in that
> project's settings — so you are scoring your file and not Claude's own notes. Then
> close the session completely, open a brand new one, and ask it to make a small
> change **without telling it anything about the project.**

It should be able to: name what the project is, run the right test command, follow
at least one of their conventions unprompted, and avoid the "do not touch" area.

**CHECK — this is the real one.** Have them write down, before the cold start, the
four things they expect it to know. Then run it and score honestly. Anything it
missed is a gap in the file, and the fix is always the same: is it missing, is it
buried, or is it too vague?

Have them fix one gap and cold-start again.

---

## Beat 6 — keeping it honest

- **Prune on a schedule.** Every few weeks, delete lines that are no longer true.
  A wrong memory is worse than no memory — the agent trusts it.
- **If the agent keeps making the same mistake, the file is wrong.** Do not correct
  it in chat for the fifth time; that correction dies with the session. Put it in
  the file.
- **If the file stops being followed, it is too long.** Halve it.

**Predict-then-run:** have them delete the Commands section, predict what the next
cold start gets wrong, then run it.

---

## What next

- **A second brain** (track 3) — memory that spans projects and thinks with them
- **Deploying subagents** (track 1) — where good memory pays off ten times over
- **Hooks and guardrails** (track 5) — for rules the agent must not be able to skip
