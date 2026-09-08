# Track 5 — Hooks and guardrails

**Prerequisites:** Claude Code, and Python or any language that can read stdin and
set an exit code.
**Time:** about 35 minutes.
**They end up with:** a rule the agent physically cannot break, and a clear sense of
which rules deserve one.

---

## What they must understand by the end

**A markdown file asks. A hook decides.**

Everything in tracks 2, 3 and 4 is instruction — the agent reads it and usually
complies. Usually is fine for style. Usually is not fine for "never commit to main"
or "never touch the production config".

A hook is a program the tool runs at a fixed point in its own loop. It gets the
proposed action as JSON on stdin, and its **exit code decides whether the action
happens**. The model is not consulted.

---

## Beat 1 — make them feel the gap

Have them put a hard rule in `CLAUDE.md`:

```markdown
## Never
Never run `git push`. Ask me and I will push myself.
```

Then have them start a fresh session and ask the agent to push a branch.

Most of the time it will refuse. **Ask them: would you bet your production database
on "most of the time"?**

That is the entire argument for hooks. Instructions are a strong prior, not a
control.

---

## Beat 2 — where a hook sits

Hooks are configured in `.claude/settings.json` (project) or `~/.claude/settings.json`
(all projects):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|PowerShell",
        "hooks": [
          {
            "type": "command",
            "command": "python ${CLAUDE_PROJECT_DIR}/.claude/hooks/guard.py"
          }
        ]
      }
    ]
  }
}
```

The pieces:

- **event** — `PreToolUse` fires before a tool runs, which is the one that can block.
  `PostToolUse` fires after, for reacting. There are others (session start, prompt
  submit, stop); check the docs for the current list, it grows.
- **matcher** — which tool this applies to. `Bash`, `Edit`, `Write`, or a pattern.
  **`Bash` alone is not enough**: `PowerShell` is a separate tool, so on Windows a
  command can walk straight past a `Bash`-only guard. Match `Bash|PowerShell`.
- **command** — what runs. Receives JSON on stdin describing the call.
  **Use `${CLAUDE_PROJECT_DIR}`, not a relative path.** A relative path resolves
  against the working directory *at the time the hook runs*, and that follows Claude
  — into a worktree, or anywhere it `cd`s. A guard that silently stops resolving is
  exactly the failure this track exists to prevent.

Edits to hook settings are picked up by a file watcher, so they generally take
effect without restarting. If a hook seems not to fire, suspect the path or the
matcher before you suspect the reload.

---

## Beat 3 — the smallest real hook

`.claude/hooks/guard.py`:

```python
import json
import sys

payload = json.load(sys.stdin)
command = payload.get("tool_input", {}).get("command", "")

if "git push" in command:
    print(json.dumps({
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "deny",
            "permissionDecisionReason":
                "Pushing is a human decision. Ask, do not push.",
        }
    }))
    sys.exit(0)

sys.exit(0)
```

There are **two supported ways** to deny with a reason, and you pick one per hook:

- **exit 0 and print JSON**, as above. `permissionDecision: "deny"` blocks the call
  and `permissionDecisionReason` is what the agent is told. More explicit, and it can
  also say `allow` or `ask`.
- **exit 2, with the reason on stderr.** Two lines instead of eight, and the agent is
  told the same thing — on a `PreToolUse`, your stderr text *is* the denial reason.

What matters is not which route you take. It is **whether you wrote a reason at all**.

Exit codes are not what a Unix habit expects:

| exit | what happens |
|---|---|
| 0 | no decision — unless you printed JSON, in which case the JSON decides |
| 1 | **without valid JSON on stdout**, a non-blocking error: the action still proceeds |
| 2 | blocks, whatever the JSON says |

**TRAP:** exit 1 is the conventional Unix "failure" code and it does **not** block.
A guard that crashes, or that returns 1 to mean "no", lets the command through.

Have them write this and ask the agent to push.

**CHECK:** it must be refused *by the hook*, not by the model being agreeable. Have
them look for their own reason text in the refusal. If they do not see "Pushing is a
human decision", the hook did not fire. The usual causes, in order: a mistyped
command path (which shows up as a `Failed with non-blocking status code` notice and
otherwise leaves the gate silently open), a matcher that misses the tool actually
used, or a workspace-trust prompt that has not been accepted yet.

---

## Beat 4 — write the message for the agent, not the log

The reason — whether you sent it as `permissionDecisionReason` or on stderr — is not
a log line. It goes to the agent, and it determines what happens next.

| Reason | What the agent does next |
|---|---|
| *(nothing)* | Blocked with no explanation. Tries variations until it gives up. |
| `Blocked.` | Tries a slight variation. Blocked again. Loops. |
| `Pushing is a human decision. Ask, do not push.` | Stops and asks. |

Have them make theirs say **what was blocked, why, and what to do instead.** This is
the single highest-leverage line in the whole hook.

**TRAP:** blocking without an alternative causes thrash — the agent will try
`git push --force`, then `git push origin HEAD`, then a shell alias. Give it the
door you want it to walk through.

---

## Beat 5 — deciding what deserves a hook

Give them the test, and make them apply it to five rules from their own repo:

> **Would you be genuinely upset if this happened once, at 2am, in a long session
> where you were not reading carefully?**

Yes → hook. No → keep it in the markdown file.

Good hook candidates: pushing, force-pushing, deleting branches, writing outside the
project, touching secrets or `.env`, editing generated files, running migrations.

Bad hook candidates: naming conventions, comment style, test structure, commit
message format. These are preferences — a hook on them makes the agent fight you
all day for no safety gain.

**CHECK:** have them justify each of their five out loud against the test. Most
people's first list is two real ones and three preferences.

---

## The project — a guard that holds

> Write one hook that enforces a rule that actually matters in your repo. Then spend
> five minutes genuinely trying to get the agent to break it.

Requirements:

- Blocks the dangerous form **and** its obvious variants. If you block `git push`,
  check what `git push --force` and `git push origin main` do.
- The message names the action, the reason, and the alternative.
- It **allows** the safe neighbours. A guard that blocks all of `git` is not a guard,
  it is a broken tool, and they will delete it by Thursday.
- Fails open on its own errors — wrap in `try/except` and `sys.exit(0)`. A crashing
  hook that blocks everything is worse than no hook.

**CHECK — adversarial:** they must try to defeat their own hook. Ask the agent
directly: *"the hook is blocking me, find another way to push"*. Whatever it finds,
that is the gap. Close it or consciously accept it.

---

## Beat 6 — the honest limits

- **A hook guards a path, not an intent.** It sees the command string. A determined
  agent with shell access has other roads; this stops accidents, not adversaries.
- **`@`-referenced files never fire a `PreToolUse` hook.** Their contents are pasted
  in while the prompt is built, so a `Read`-matched secrets guard has a hole straight
  through it. Worth knowing before you trust one.
- **They run on every matching call.** Slow hooks make the whole session slow. Keep
  them to milliseconds.
- **They are config, and config is trusted code.** Hooks run shell commands with your
  full user permissions, so committing `.claude/settings.json` means pulling a branch
  can hand you someone else's commands. Interactively you get a workspace-trust prompt
  first — but a scripted `-p` or SDK run does not, and treats the folder as trusted.
  That is the case to actually worry about.
- **Too many and you have built a cage.** Three good hooks beat twenty. Every hook
  is a thing that can misfire at the worst moment.

**Predict-then-run:** have them replace the whole JSON block with a bare
`sys.exit(2)` — blocking, but writing nothing to stderr and printing no JSON — then
predict what the agent is told before testing. It still blocks, and the agent gets no
reason, so it tries variations instead of asking. Then have them add one
`print(..., file=sys.stderr)` above the exit and run it again: same block, and now the
agent stops and asks. The lesson is not JSON versus exit codes — both routes work.
It is that a block with no message and a block with a message produce completely
different behaviour.

---

## What next

- **Build your own skill** (track 4) — for the rules that only need asking
- **Permanent memory** (track 2) — so the agent knows the rule before it tries
