---
name: myatomicgit
description: Read every file a task added or modified, group them into logical implementation units, and print ready-to-copy git add and git commit commands — one commit per unit, in dependency order. Use when the user runs /myatomicgit, or asks to "split this into commits", "group my changes", "give me the git commands", "atomic commits for this", or finishes an implementation and wants the diff carved into coherent commits instead of one lump. Groups by what the files do rather than where they live: the data a change needs, then the capability it adds, then the code that uses it, then the tests adapted to it, then the tests that prove it — so every commit leaves the tree coherent on its own. Reuses the implementation unit numbers from the plan when one exists, deriving them from the diff when it does not. Writes one-line conventional messages in the project's own observed style, carrying the ticket. Never stages a file that does not trace to the task: local-only changes such as docker tweaks, env files and entrypoint scripts are reported as left unstaged, never folded into a commit, and confirmed ones are recorded so they are never offered again. Prints only — runs no git command, stages nothing, commits nothing. The printed block is git commands and nothing else: no comments, no co-authored line, no prose. Do NOT use to write a PR description (use /mypr), or to commit review fixes (/myfix prints its own).
---

# 📦 /myatomicgit — Atomic Commits From a Diff

Read what the task changed, group it by what the files actually do, and print the commands. One commit per unit, ordered so each one leaves the tree coherent.

Prints only. Nothing is staged, committed or pushed — you run the block.

## Rules

- **Print, never run.** No `git add`, no `git commit`, no `git push`, no `git stash`. Read-only inspection only — `git status`, `git diff`, `git log`, `git show`. The output is text to copy.
- **The command block is git commands and nothing else.** No `#` comments, no co-authored trailer, no prose, no explanation inside the fence. Blank lines between groups are the only separator. Everything you want to say goes *above* the block.
- **Never stage a file that does not trace to the task.** A file is in only when the task's work explains it. Everything else is reported under **Left unstaged** with a one-line reason — never quietly folded into the nearest commit because it happened to be dirty.
- **Treat local-only changes as radioactive.** Docker files, `.env*`, entrypoint scripts, editor and tooling config, anything matching a recorded `do-not-commit` entry: these are frequently tracked, frequently modified for one machine, and must never reach a commit. Call them out by name. When the user confirms a file is local-only, record it under `do-not-commit` in the project's `CLAUDE.md` so it is never offered again.
- **Group by function, not by folder.** Files belong together when they serve one change, not when they share a directory. Six request classes and the controller that gates them are one unit; two policies are another; the seeder they both depend on is a third.
- **Order by dependency, so every commit stands alone.** The data a change needs, then the capability it adds, then the code that uses it, then the tests adapted to it, then the tests that prove it. Someone checking out any single commit should find a tree that makes sense — not one half-wired to something that lands two commits later.
- **Reuse the plan's unit numbers when a plan exists.** `ce-plan` already numbers implementation units and `/myplanfix` forbids renumbering them, so commits map 1:1 to plan units. No plan, no `docs/plans/` entry → derive the units from the diff.
- **Match the project's own commit style**, read from `git log`, not from a template. Observe the types in use, whether a scope appears, where the ticket sits, and the casing. A repo's history is the convention; do not impose one.
- **One line per message.** No body, no bullet list, no wrapped paragraph.
- **A file appears in exactly one commit.** If something genuinely belongs to two units, the units were drawn wrong — redraw them.
- **Print the result in the chat/terminal** using the exact output format below.

## Steps

1. **Collect what changed.** Staged, unstaged, untracked and deleted, scoped to the task's work:

   ```bash
   git status --porcelain
   git diff
   git diff --cached
   ```

   Note deletions and renames — they need `git rm` and `git mv`, not `git add`. If the working tree is clean, say so and stop.

2. **Learn the project's convention** from its own history, rather than assuming one:

   ```bash
   git log --oneline -20
   ```

   Extract the shape: the types in use, whether scopes appear and what they refer to, where the ticket identifier sits, the casing. Mirror it exactly.

3. **Read the files, not just their paths.** A path suggests a grouping; the contents confirm or refute it. Two files in the same directory can belong to different units, and a seeder three directories away can be the thing both of them need. Open them.

4. **Separate what is not yours to commit.** Before grouping, set aside anything that does not trace to the task:
   - recorded `do-not-commit` entries
   - local-only shapes: `docker/`, `.env*`, entrypoint scripts, editor and tooling config, local overrides
   - changes from unrelated work sharing the working tree

   These go to **Left unstaged**, with a reason each. When one is ambiguous, ask rather than guess — committing a machine-specific file is far more expensive to undo than one question.

5. **Group into units.** Each unit is one coherent change: a thing added, a behaviour altered, a set of tests adapted. Use the plan's U-IDs when there is a plan. Give each unit a one-line rationale — if you cannot say in a sentence why these files are one change, they probably are not.

6. **Order the units by dependency** so each commit leaves the tree coherent. Data before the capability that reads it; capability before the code that calls it; implementation before the tests adapted to it; new proving tests last.

7. **Write one message per unit** in the project's style — type, scope, lowercase description, ticket. Describe what the commit does, not which files it touches.

8. **Print the analysis, then the block.**

## Output format

Print exactly this structure:

````
### 📦 Units
| U | Files | Why these are one change |
|---|---|---|
| U1 | `path/one.php` | [one line] |
| U2 | `path/two.php`, `path/three.php` | [one line] |

Order: U1 → U2 → … — [the dependency that fixes this order]

### 🚫 Left unstaged
- `docker/php/entrypoint.sh` — local-only, not part of this task
- `some/other.php` — unrelated work in the same tree

*(omit this section entirely when everything traces to the task)*

### ⌨️ Commands

```bash
git add path/one.php
git commit -m "chore(scope): what U1 does (TICKET)"

git add path/two.php
git add path/three.php
git commit -m "feat(scope): what U2 does (TICKET)"
```
````

The fenced block contains git commands and blank lines. Nothing else goes inside it — the analysis above is where anything explanatory belongs.

One `git add` per line rather than several paths on one, so a line can be dropped without re-editing the command. Add `git push` only when the user asks for it.

## Goal

Turn a finished implementation into a history someone can read. Each commit is one coherent change, ordered so the tree stands up at every step, with the files that were never yours to commit held back and named — and the whole thing printed as a block you copy, because the decision to run it stays yours.
