# /myatomicgit

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myatomicgit` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Read what a task changed, group the files by what they actually do, and print ready-to-copy `git add` / `git commit` commands — one commit per unit, in dependency order.

> **Prints only.** It runs no git command, stages nothing, commits nothing. The block is yours to copy and run.

## Install (Claude Code)

```bash
git clone --branch skill/myatomicgit https://github.com/itzmerai/my-agent-skills.git myatomicgit-skill
ln -s "$PWD/myatomicgit-skill/skills/myatomicgit" ~/.claude/skills/myatomicgit
```

Restart Claude Code, then type `/` and you should see `/myatomicgit`.

## What it produces

```
### 📦 Units
| U  | Files                                      | Why these are one change |
| U1 | database/seeders/PermissionSeeder.php      | the permission the rest depends on |
| U2 | app/Policies/{Invoice,Receipt}Policy.php   | the new capability |
| U3 | …/Requests/*.php + InvoiceController.php   | the gate that uses U2 |

### 🚫 Left unstaged
- docker/php/entrypoint.sh — local-only, not part of this task

### ⌨️ Commands
```bash
git add database/seeders/PermissionSeeder.php
git commit -m "chore(taxi): add manage_invoicing permission (ENG-2448)"

git add app/Policies/InvoicePolicy.php
git add app/Policies/ReceiptPolicy.php
git commit -m "feat(taxi): add Invoice and Receipt authorization policies (ENG-2448)"
```
```

The analysis is above so you can check the grouping. The fenced block is **git commands and blank lines, nothing else** — no comments, no co-authored trailer, no prose — so it copies clean.

## How it groups

**By function, not by folder.** Files belong together when they serve one change, not when they share a directory. Six request classes and the controller gating them are one unit; two policies are another; the seeder they both need is a third.

**Ordered by dependency, so every commit stands alone:**

```
data the change needs  →  the capability it adds  →  the code that uses it
                       →  tests adapted to it     →  tests that prove it
```

Check out any single commit and the tree makes sense — nothing is half-wired to something landing two commits later.

**Plan units win.** When `ce-plan` has numbered the implementation units, commits map 1:1 to them rather than inventing a fresh numbering.

**Convention is read, not imposed.** It reads `git log` and mirrors what the repo already does — the types in use, whether a scope appears, where the ticket sits, the casing.

## What it refuses to stage

The reason a skill that emits `git add` needs care: a file can be tracked, modified, and still have no business in a commit.

Docker files, `.env*`, entrypoint scripts, editor and tooling config, local overrides — these are frequently tracked and frequently edited for one machine. They're reported under **Left unstaged** with a reason, never folded into the nearest commit because they happened to be dirty. Confirm one is local-only and it's recorded under `do-not-commit` in the project's `CLAUDE.md`, so it's never offered again.

Anything ambiguous gets a question rather than a guess. Committing a machine-specific file is much more expensive to undo than asking.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
