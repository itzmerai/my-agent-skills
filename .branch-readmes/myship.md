# /myship

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myship` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Run `/myatomicgit` and `/mypr` together, and print **one** block with the overlaps already settled — so what you copy is a decision, not a choice between two answers.

> **Prints, never runs.** No staging, no commits, no pushes. The decision to run the commands stays yours.

## Install (Claude Code)

```bash
git clone --branch skill/myship https://github.com/itzmerai/my-agent-skills.git myship-skill
ln -s "$PWD/myship-skill/skills/myship" ~/.claude/skills/myship
```

Restart Claude Code, then type `/` and you should see `/myship`.

## The problem

Run the two skills back to back and their outputs contradict each other. This is real output from doing exactly that:

```
(You've already got the atomic two-commit split from /myatomicgit — use those
 instead of the single-line commit above if you want the fix and test as
 separate commits. The push command is the same.)
```

That parenthetical is the bug. Two commit plans reached the screen and the reader was left to adjudicate — which is the job an orchestrator exists to do.

## Four overlaps, settled before anything prints

| | What collides | How it resolves |
|---|---|---|
| **1** | `/mypr` writes a **one-line** commit; `/myatomicgit` writes an **atomic split** | `/myatomicgit` wins. The one-liner is **suppressed** — not printed smaller, not struck through, not footnoted |
| **2** | Both print a `git push` | One, as the last line of the commands block |
| **3** | `/myatomicgit`'s **Left unstaged** list vs. the brief's device-local note | Carried across automatically, in `/myatomicgit`'s own words |
| **4** | The brief's **Changes** list comes from the working tree; the commits come from the staged set | Changes == the staged set, exactly |

### The fourth one is the quiet one

`/mypr` alone lists every changed file. On a tree holding a local `docker-compose.yml`, a tweaked entrypoint or a device-specific `.env` — the files `/myatomicgit` deliberately refuses to stage — the brief claims work the pull request will not contain.

Nobody catches it, because the brief looks complete. A description that overstates its diff is worse than one that is thin, because it reads as finished.

## Order is forced

`/myatomicgit` runs **first**, always. Its grouping *and* its held-back list are inputs to the brief. The other way round, `/mypr` describes a file set the commands will never stage.

```
working tree
      ↓
/myatomicgit   → units · left unstaged · atomic commit commands
      ↓          its staged set and its held-back list feed the step below
/mypr          → base branch · PR brief        (its one-liner: suppressed)
      ↓
ONE BLOCK
```

## What you get

```
### 📦 Units                      the grouping and why
### 🚫 Left unstaged              omitted when everything traces to the task
### ⌨️ Commands                    atomic commits, then ONE push, last
### ➡️ Base this PR on `develop`
### 📋 PR brief                   Changes == the staged set
### ▶️ Next                       open the PR, then /myloop <url>
```

The commands fence inherits `/myatomicgit`'s rule unchanged: **git commands and blank lines, nothing else.** No `#` comments, no co-authored trailer, no prose. One `git add` per line, so a line can be dropped without re-editing the command.

## How it differs from `/myloop`

`/myloop` streams each sub-skill's output in full and unmodified, one stage after another. `/myship` **merges** instead.

That is deliberate, and it is the one rule the two orchestrators disagree on. A faithful stream here reproduces the exact contradiction at the top of this page. Both skills still run in full and nothing either decides is discarded — only their presentation is combined.

## What it does not do

It contributes **no grouping logic, no commit-message style, and no brief content**. If a commit message reads wrong, that is `/myatomicgit`. If the testing steps are thin, that is `/mypr`. Fix it there — this skill is sequencing and reconciliation, and nothing else.

Neither sub-skill is modified, so both still stand alone: `/myatomicgit` when all you need is the commits, `/mypr` when a single commit is genuinely all the change needs.

## Pairs with

Runs at the end of the build, after `/myscope` and `/mychecklist` have confirmed the work is in scope and complete. Hands off to `/myloop` once the pull request exists.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
