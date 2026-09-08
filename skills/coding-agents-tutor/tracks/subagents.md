# Track 1 — Deploying subagents

**Prerequisites:** Claude Code or Codex. Nothing to install.
**Time:** about 30 minutes.
**They end up with:** a parallel review that runs several agents at once over their
own repo and writes one report.

---

## What they must understand by the end

One sentence: **a subagent starts with an empty head.**

It does not get your conversation. It does not know what you agreed three messages
ago. What it *does* get is: the task message Claude composes for it, your `CLAUDE.md`
hierarchy, a git status snapshot, any skills preloaded in its definition, and
whatever else it reads off the disk itself.

Two caveats that matter more than they look, and that most write-ups skip:

- **A *fork* is the exception.** A fork inherits the whole conversation. Fork *mode*
  is permission for Claude to request one, not the default type: when Claude spawns a
  subagent **without requesting a type** it gets the general-purpose agent, with a
  fresh context. But Claude can request the `fork` type, and does so freely once fork
  mode is on — which it is by default in interactive sessions **from Claude Code
  v2.1.232**. On older builds it is off unless you set `CLAUDE_CODE_FORK_SUBAGENT=1`.
  You can also start one yourself: `/subtask` on v2.1.212+, `/fork` on v2.1.161–211.
- **The built-in Explore and Plan agents skip `CLAUDE.md` and git status entirely**,
  and there is no setting to change that.

Everything else in this track follows from that. Parallel agents are not "the same
agent, times ten" — they are ten strangers who all have to be briefed in writing.
That is why people who try to scale agents without fixing their files get ten times
the mess instead of ten times the work.

---

## Beat 1 — prove the empty head

Before any theory. Have them run this in a project they know:

> Ask your agent to delegate to a **named custom subagent** (not a fork) with a task
> like: *"State what we are working on, using only what you already have."*

It will not know the conversation. It may still describe the project accurately —
from `CLAUDE.md` and the git snapshot it was given — and that is the real lesson:
**what a subagent knows is what was put in front of it, not what you said.**

**TRAP — worth knowing before you run it.** Fork mode is on by default in an
interactive session, so Claude *can* answer this with a fork, which inherits the
conversation and will answer perfectly. That looks like the demo failing when it is
a different feature. Naming the agent removes the ambiguity.

**Then ask them:** why is that a feature and not a bug?

Let them answer. Steer to: because a fresh context is the only reason you can run
twenty of them — if each one carried the full conversation you would run out of
context on the third.

**TRAP:** most people meet this as a bug. They fan out five agents, the results
come back inconsistent, and they conclude parallel agents do not work. The agents
were fine. The brief was missing.

---

## Beat 2 — write one subagent down

A subagent you will use twice belongs in a file, not in a prompt you retype.

**Claude Code** — `.claude/agents/<name>.md` in the project (or `~/.claude/agents/`
for every project):

```markdown
---
name: test-auditor
description: Finds tests that assert nothing useful. Use when reviewing test quality.
tools: Read, Grep, Glob
---

You audit test files for tests that pass but prove nothing.

Look for:
- assertions on mocks rather than on behaviour
- tests with no assertion at all
- a test whose name claims more than its body checks

Report each as: file, line, what it claims, what it actually checks.
Report nothing else. Do not fix anything.
```

**Codex** — same idea, different shape entirely: subagents are **TOML** files in
`.codex/agents/` or `~/.codex/agents/`, with `name`, `description` and
`developer_instructions`. Not markdown, not frontmatter. The discipline transfers;
the file does not.

Have them write one for something they actually care about in their repo.

**CHECK:** read their file. It should have (a) a `description` that says *when* to
use it, not just what it is, and (b) a body that ends with a hard limit on scope —
"report only", "do not fix", "one file at a time". Agents with no stated limit
wander.

---

## Beat 3 — the brief is the whole job

Give them this as a rule and make them apply it:

> A subagent prompt must answer four things:
> 1. **What to look at** — the exact path, glob, or file list.
> 2. **What to produce** — the shape of the output, not a vibe.
> 3. **What not to do** — the scope limit.
> 4. **What "done" looks like** — so it stops.

Have them rewrite their Beat 2 agent's prompt against those four. Most people's
first version is missing 2 and 4.

**Why this matters more in parallel:** with one agent you correct it in the next
message. With eight running at once you cannot correct anything — they all finish
and hand you eight things. The brief is your only steering wheel.

---

## Beat 4 — fan out for the first time

Three agents, one command, different slices of the same problem.

Have them ask their agent for something shaped like:

> Review this repo across three dimensions, one subagent each, all at the same
> time: (1) error handling, (2) naming and clarity, (3) dead or unreachable code.
> Each returns at most 5 findings as `file:line — what — why it matters`.
> Then merge into one list, most severe first.

**CHECK:** ask for all three in **one request**. Claude decides how to batch the
delegation, and in an interactive session subagents run in the background, so asking
across three separate turns does not reliably serialise them either. (There is a
blunt override — `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` forces the foreground
everywhere — but you rarely want it.) What one request buys you is that Claude plans
the three briefs together, which is what makes them comparable.

**CHECK:** are the three outputs in the *same format*? If not, the merge is manual
work and the fan-out saved them nothing. Fix by putting the output shape in each
prompt, identically.

---

## Beat 5 — where the files come in

Now connect it back.

Ask them: *those three agents did not know your project conventions. How would you
tell all three at once, without pasting it into three prompts?*

The answer is a file: `CLAUDE.md` (Claude Code) or `AGENTS.md` (Codex and 20+ other
tools) at the repo root. Claude Code **injects** your CLAUDE.md hierarchy into each
subagent's starting context — they do not have to go and find it — so one file
briefs all of them for free.

**TRAP — the big exception.** The built-in **Explore and Plan** agents are the two
that skip `CLAUDE.md` and git status, and a "review this repo" fan-out may well route
to Explore. There is no setting to change it. When a rule *must* reach the subagent,
restate it in the delegation prompt as well as the file.

Have them add three real conventions to that file — not generic ones, three things
that are actually true about their repo. Then re-run Beat 4 and compare.

**CHECK:** the second run should reference their conventions. If it does not, check
in this order: was it an Explore/Plan agent (which never got the file), then is the
file too long, then is it too vague. Getting that order wrong sends people off
shortening a file that was never loaded.

> This is the hinge of the whole track. If they only remember one thing: **you scale
> agents by improving the files they read, not by writing longer prompts.**

---

## The project — a parallel review that ships a report

Now they build the real one.

> Build a review that runs **six** subagents at once over your repo — one per
> dimension: correctness, error handling, tests, naming, dead code, and
> performance. Each returns findings in one fixed format. The parent merges them,
> drops duplicates, sorts by severity, and writes `review.md`.

Rules to hold them to:

- The output format is defined **once** and pasted into all six prompts identically.
- Every agent has a cap (say 5 findings) so one chatty agent cannot flood the report.
- The merge step is the parent's job, not a seventh agent's.
- `review.md` must include which agent found what, so a bad agent is traceable.

**CHECK the report, not the process:** open `review.md`. Are the findings real? Have
them pick the top finding and verify it by hand. If the top finding is wrong, the
brief for that dimension is wrong — fix that prompt, not the merge.

---

## Beat 6 — scaling, and what actually breaks

Once six works, the honest picture:

- **There is a configurable ceiling, 20 by default.** With twenty already running,
  the next spawn fails outright with a no-retry error rather than queueing.
  `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` moves it up or down. Design a big fan-out in
  waves rather than assuming unlimited width.
- **Going wider is cheap; going deeper is not.** Twenty agents each reading three
  files is fine. Twenty agents each reading the whole repo will be slow and
  expensive, and most of them will return the same finding.
- **Split by dimension, not by file.** Six agents on six aspects of one repo gives
  six different answers. Six agents on six folders gives you six copies of the same
  answer.
- **Cost is roughly linear in agents.** Ten agents is ten times the tokens. That is
  fine when it replaces an hour; it is silly for something one agent does in a
  minute.
- **The merge is where quality dies.** Most bad fan-outs are good agents plus a
  careless merge. Give the merge as much thought as the prompts.

**Predict-then-run:** have them change one agent's cap from 5 to 1 and predict what
the report looks like before running it. Then run it.

---

## What next

- **Permanent memory** (track 2) — so the briefing file writes itself over time
- **Build your own skill** (track 4) — so this whole review is one command
