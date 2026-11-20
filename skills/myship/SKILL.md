---
name: myship
description: Run /myatomicgit and /mypr together and print one reconciled block — the atomic commit commands, a single push, the base branch, and the PR brief — ready to paste. Use when the user runs /myship, or asks to "give me the commits and the PR", "wrap this up", "ship it", "commit commands plus PR description", "I'm done, what do I run", or finishes an implementation and wants everything needed to open the pull request in one go. Exists because running the two skills by hand produces outputs that contradict each other: /mypr writes a single-line commit message while /myatomicgit writes an atomic split, both print a push, and the reader is left to decide which to use. /myship settles all of it before printing. /myatomicgit runs first and owns the commits; /mypr's one-liner is suppressed rather than printed and retracted, since a retraction still makes the reader choose. One push, as the last line of the commands block. The PR brief's Changes list is exactly the set the commands stage, never the whole working tree, so a device-local file the commit plan deliberately held back can never be described as part of the pull request. The files /myatomicgit left unstaged are carried into the brief's notes automatically instead of being remembered by hand. Sequencing and reconciliation only — it adds no grouping logic, no commit-message style and no brief content of its own, and neither sub-skill is modified, so both still stand alone. Never stages, commits or pushes. Do NOT use to run only one of the two (use /myatomicgit or /mypr directly), to review a pull request that already exists (use /myloop), or to commit review fixes (/myfix prints its own).
---

# 🚢 /myship — Commits + PR Brief, Reconciled

Run `/myatomicgit`, then `/mypr`, and print **one** block with the overlaps already settled — so what you copy is a decision, not a choice between two answers.

```
working tree
      ↓
/myatomicgit   → units · left unstaged · atomic commit commands
      ↓          its staged set and its held-back list become inputs below
/mypr          → base branch · PR brief        (its one-liner commit: suppressed)
      ↓
ONE BLOCK      → units → left unstaged → commands + one push → base → brief → next
```

## Why this is not just the two run back to back

Run by hand, they contradict each other, and the contradiction reaches the user:

> *(You've already got the atomic two-commit split from /myatomicgit — use those instead of the single-line commit above if you want the fix and test as separate commits.)*

That parenthetical is the defect. Resolving it is the whole job.

## Rules

- **`/myatomicgit` runs first, always.** Its grouping *and* its left-unstaged decision are inputs to the brief. Run in the other order, `/mypr` describes a file set the commands will never stage.
- **Exactly one commit plan survives, and it is `/myatomicgit`'s.** `/mypr`'s one-liner is **suppressed** — not printed smaller, not struck through, not footnoted. A retraction in the output still hands the reader a decision, which is the thing this skill exists to take off their hands.
- **One `git push`, and it is the last line of the commands block.** Both sub-skills offer one; only one is printed.
- **The brief's Changes list is the staged set, nothing else.** Not `git status`, not the diff. If `/myatomicgit` held a file back, that file is not part of this pull request and must not be described as though it were — a brief that claims work the diff does not contain is worse than an incomplete one, because it reads as finished.
- **Carry the left-unstaged list into the brief's notes**, in `/myatomicgit`'s own words. It already did that analysis; re-deriving it is how the two lists drift apart.
- **The commands fence is git commands and blank lines. Nothing else.** `/myatomicgit`'s rule, unchanged: no `#` comments, no co-authored trailer, no prose. One `git add` per line, so a line can be dropped without re-editing the command.
- **Chain the skills, do not reimplement them.** No grouping logic, no commit-message style, no brief content originates here. If a commit message reads wrong, that is `/myatomicgit`; if the testing steps are thin, that is `/mypr`. Fix it there.
- **Merge the outputs; do not stream them.** This is the one place `/myship` departs from `/myloop`, which prints every stage verbatim. Here a faithful stream reproduces the exact contradiction above. Both skills still run in full and nothing they decide is discarded — only their *presentation* is combined.
- **Run straight through. Never ask permission to continue.** Invoking `/myship` is the go-ahead for both halves. No "shall I run /mypr now?", no pause between them.
- **Never stage, commit, or push.** Read-only on git — `git status`, `git diff`, `git log`, `git branch`, `git rev-parse`. Everything else is text for the user to run.
- **Neither sub-skill is modified.** Both must still work standalone exactly as before.
- **Stop if there is nothing to ship.** A clean tree means say so and print nothing else.

## Steps

1. **Run `/myatomicgit`.** Let it do its own work — read every changed file, group into units, order by dependency, hold back anything that does not trace to the task. Keep three things from it: the **units table**, the **left-unstaged list**, and the **staged set** (every path that appears in a `git add`).

2. **Run `/mypr`**, handing it the staged set from step 1 as the file list for the brief. It still detects the project type, resolves the base branch, and fills the template from the real diff. Discard only its step-3 one-liner.

3. **Reconcile, then print once.** Assemble the block below. Before printing, check all four:
   - one commit plan, `/myatomicgit`'s
   - one `git push`, last in the block
   - Changes == the staged set, exactly
   - the left-unstaged files named in the brief's notes and absent from every `git add`

## Output format

Print exactly this structure:

`````
### 📦 Units
| U | Files | Why these are one change |
|---|---|---|
| U1 | `app/Thing.php` | [one line] |
| U2 | `tests/ThingTest.php` | [one line] |

Order: U1 → U2 — [the dependency that fixes this order]

### 🚫 Left unstaged
- `docker/entrypoint.sh` — local-only, not part of this task

*(omit this section entirely when everything traces to the task)*

### ⌨️ Commands

```bash
git add app/Thing.php
git commit -m "fix(scope): what U1 does (TICKET)"

git add tests/ThingTest.php
git commit -m "test(scope): what U2 does (TICKET)"

git push -u origin <current-branch>
```

### ➡️ Base this PR on `develop`

### 📋 PR brief

```markdown
<the /mypr template, filled>
```

### ▶️ Next
Open the pull request, then run `/myloop <pr-url>` to review it.
`````

The `Changes` section inside the brief lists the staged set and only the staged set. The left-unstaged files go in the brief's closing note, phrased as `/myatomicgit` phrased them — *"device-local and intentionally left out of this PR."*

## Goal

Make the end of a task a single paste. Two skills run, four overlaps are settled before anything reaches the screen, and what you get back is a commit history someone can read plus a pull request that describes exactly the files those commits contain — with nothing left for you to adjudicate.
