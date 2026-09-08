# Track 4 — Build your own skill

**Prerequisites:** Claude Code or Codex. Nothing to install.
**Time:** about 30 minutes.
**They end up with:** a working skill of their own, installed, that runs one of their
real workflows the same way every time.

---

## What they must understand by the end

**A prompt runs once. A skill runs the same way forever.**

A skill is a folder with a markdown file in it. The frontmatter says *when* to use
it; the body says *how*. The agent loads the **body** only when the situation matches.

Be precise about the cost, because "free until used" is not true: every installed
skill's **description** sits in context every session, and once a skill fires its body
**stays** in context for the rest of the session. So a skill costs a line always, and
its full length from first trigger onward.

That still beats stuffing a long workflow into a memory file, which costs its full
length every session whether you need it or not — but it is a smaller win than it
sounds, and it is why a skill's body should be short too.

---

## Beat 1 — pick something they actually repeat

Ask them for a workflow they have explained to their agent **more than twice**.
Deploy steps. How they want a PR described. Their review checklist. How they name
branches.

**TRAP:** the instinct is to build something impressive and general. The best first
skill is small, boring and theirs. If they cannot name a workflow they repeat, this
track is the wrong one today — send them to track 2.

---

## Beat 2 — the anatomy

```
.claude/skills/<skill-name>/
  SKILL.md          <- required, the only required file
  reference.md      <- optional, loaded only if SKILL.md says to
  scripts/          <- optional
```

`~/.claude/skills/` for every project, `.claude/skills/` inside a repo for just that
one. **Codex** looks in `~/.agents/skills/` and `.agents/skills/` — same idea, and it
adds `references/` and `assets/`; it also *requires* `name` and `description` where
Claude Code treats both as optional.

**`name:` does not name the command.** For personal and project skills the invocation
name comes from the **directory**; `name:` is only the display label. Rename one
without the other and you will confuse yourself.

```markdown
---
name: pr-describe
description: Writes the PR description for the current branch in our house format.
  Use when opening a PR, when asked to describe a branch, or when the user says
  "write the PR". Not for commit messages.
---

# PR description

Read the diff against the base branch first. Never describe a branch you have not
diffed.

Produce exactly these sections:

## What changed
Two sentences maximum, in plain language.

## Why
The problem, not the solution.

## Risk
What could break, and what you checked. If nothing could break, say why.

Rules:
- No bullet lists of file names. The diff already shows those.
- If the diff touches migrations, say so in Risk explicitly.
```

---

## Beat 3 — the description is the whole trigger

This is the part that decides whether a skill ever fires, and it is where every
first skill goes wrong.

The agent decides mostly on `description`, so it must say **when to use it**, in the
words the person would actually use — including the cases where it should *not* fire.

Three related fields worth knowing:

- **`when_to_use`** — appended to the description in the listing. Purpose-built for
  exactly the trigger phrases you are about to write.
- **`paths`** — glob patterns that gate automatic activation to matching files. A
  cleaner fix than keyword-stuffing when a skill is only for one file type.
- **`disable-model-invocation: true`** — the deliberate "never fire on its own" switch.

**There is a hard cap** — and the numbers are Claude Code's. `description` and
`when_to_use` together are truncated at **1,536 characters** in the listing, and the
whole listing has a budget of about 1% of the context window; past that, descriptions
get dropped least-used first, though every skill *name* is kept. If a skill that used
to fire stops firing, that is a prime suspect, and `/doctor` will tell you.

**Codex differs on every number.** Its listing budget is about 2% of the context window
(or 8,000 characters when that is unknown), it has no 1,536-character cap, it has no
`/doctor`, and when it runs out of room it can **omit whole skills** with a warning
rather than just shortening descriptions. Its "never fire on its own" switch is
`allow_implicit_invocation: false` in `agents/openai.yaml`, not
`disable-model-invocation`.

| | |
|---|---|
| Bad | `description: Helps with pull requests.` |
| Good | `description: Writes the PR description for the current branch in our house format. Use when opening a PR, describing a branch, or the user says "write the PR". Not for commit messages.` |

Have them write theirs, then judge it: *if you only read this line, would you know
whether to open the skill?*

**CHECK:** have them list three phrasings they might genuinely type. Are all three
covered by the description's trigger words?

---

## Beat 4 — write the body like a procedure

The body is instructions to a capable colleague who has not seen your workflow, not
documentation about it.

- **Imperative.** "Read the diff first", not "this skill reads the diff".
- **Order matters.** Put the thing that must happen first, first.
- **State the failure mode.** "Never describe a branch you have not diffed" is worth
  more than three paragraphs of explanation.
- **Be specific about output.** If the shape matters, show the shape.
- **Keep it short.** Long skills get skimmed like long anything. Push detail into a
  `reference.md` the skill tells the agent to read only when needed.

Have them write theirs now, in one pass, then cut it by a third.

---

## Beat 5 — install and fire it

Put the folder in `~/.claude/skills/<name>/`. Claude Code watches that directory and
picks the skill up **in the current session, no restart needed**.

**TRAP — and this one hits first-timers specifically.** If `~/.claude/skills/` did not
exist when the session started, Claude Code is not watching it yet and you **do** need
a restart. That is the most likely state for someone writing their first skill, and it
presents exactly like the failure below — so restart once before you go blaming the
description.

Then have them trigger it *without naming it*, using one of their three natural
phrasings from Beat 3.

**CHECK — the real test:** did it fire on its own? If they had to say "use the
pr-describe skill", the description failed and the skill is useless in practice,
because in real work they will forget it exists. Send them back to Beat 3 and
rewrite the trigger.

This is the most common failure and it is worth being blunt about it.

---

## The project — a skill they will use tomorrow

> Build and install a skill for a workflow you genuinely repeat. Trigger it three
> times, on three different days' worth of real work, without naming it. Fix the
> description after each miss.

Rules:

- It must be **their** workflow, not a demo.
- The description must cover at least three natural phrasings.
- The body must be under roughly 100 lines, with anything longer moved to a
  reference file.
- It must fire without being named.

**CHECK:** have them show the output from a real run. Is it actually better than
what they got by explaining it each time? If not, the body is too vague — usually
because it describes the workflow instead of instructing it.

---

## Beat 6 — the honest limits

- **A skill is instructions, not enforcement.** The agent can still ignore it. If it
  *must* hold, you need a hook — track 5.
- **They compete.** Two skills with overlapping descriptions makes the choice
  ambiguous. Keep triggers disjoint.
- **They go stale like any doc.** When the workflow changes, the skill is now
  actively wrong and will confidently do the old thing.
- **Sharing is copying a folder.** That is the upside: no registry, no install step,
  works across machines and across agents. This tutor is itself a skill in a folder.

**Predict-then-run:** have them replace the description with just the skill's name,
predict whether it still fires, then test.

---

## What next

- **Hooks and guardrails** (track 5) — for the rules that must not be optional
- **Deploying subagents** (track 1) — package a whole fan-out as one skill
