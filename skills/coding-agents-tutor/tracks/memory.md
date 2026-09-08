# Track 2 — Permanent memory

**Prerequisites:** Claude Code or Codex. Nothing to install.
**Time:** about 25 minutes.
**They end up with:** a memory setup where the agent starts a brand-new session
already knowing their project, their conventions, and the decisions they have
already made.

---

## What they must understand by the end

**The agent has no memory. It has files it reads at the start of every session.**

That is not a limitation to work around — it is the whole mechanism. Once you see
memory as "files that get read", every memory problem becomes a file problem: what
is in it, how long it is, and whether it is still true.

---

## Beat 1 — see the cost of no memory

Have them open a **fresh** session in a project they have used the agent on before,
and ask:

> "What do you know about this project's conventions?"

Whatever it answers came from files, not from last week. Have them notice how much
of what they explained last time is simply gone.

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

**CHECK:** start a fresh session and ask the agent to state one convention from
`AGENTS.md`. If it cannot, the import is not resolving — check the path is relative
to the file doing the importing.

---

## Beat 4 — the hierarchy, and which file wins

There is more than one place memory lives:

| Where | Scope | Use it for |
|---|---|---|
| `~/.claude/CLAUDE.md` | every project you open | how *you* like to work |
| `<repo>/CLAUDE.md` | this project | what this project is |
| `<repo>/<subdir>/CLAUDE.md` | that subtree | rules that only apply in there |

More specific wins where they conflict, and the general ones still apply where they
do not. Have them put one genuinely personal preference in the user-level file —
something true across all their projects, like "explain before you refactor" — and
confirm it shows up in a different repo.

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

Then have them make a small real decision in the session and ask the agent to
record it.

**CHECK:** read the line it wrote. Does it capture *why*? "Chose Postgres" is
useless. "Chose Postgres over SQLite — needs concurrent writes from two services"
is memory.

**TRAP:** recording things the repo already tells you. "This project uses React" is
visible from `package.json`; writing it down costs context every session and earns
nothing. Memory is for what is **not** derivable.

---

## The project — survive a cold start

> Set up memory in a real project of yours. Then close the session completely, open
> a brand new one, and ask it to make a small change **without telling it anything
> about the project.**

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
