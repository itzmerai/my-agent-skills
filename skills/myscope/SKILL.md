---
name: myscope
description: Audit every file change after completing a task to verify that all modifications are strictly within the task's scope. Use when the user runs /myscope, at the end of any coding task, before committing, opening a PR, or reporting work as done — and whenever the user asks to "audit changes", "check the diff", "verify scope", "make sure nothing else broke", or "review what changed". Reviews the full diff (staged and unstaged, plus new and deleted files) against the original task request, flags any out-of-scope edits (unrelated refactors, formatting churn, renamed symbols, dependency or config changes, deleted code), and specifically checks whether core functionality was touched (entry points, shared utilities, public APIs, auth, database schemas, build/CI config). Produces a per-file verdict (in-scope / out-of-scope / core-touched) with a justification tied to the task, and recommends reverting or isolating anything that isn't required. Do NOT use for planning a task before work begins or for general code-quality review unrelated to scope.
---

# 🔬 /myscope — Post-Implementation Scope Audit

Audit the complete change set against the task that caused it. This is the QA gate between "implementation done" and "commit / PR" — it answers one question: **did anything change that this task did not require?**

## Rules

- **Audit only.** Do NOT write, fix, refactor, or revert code. You report and recommend; the user decides and acts.
- **NO git state-modifying commands.** Never run `git add`, `git commit`, `git push`, `git restore`, `git checkout`, `git stash`, or `git reset`. Read-only inspection only: `git status`, `git diff`, `git log`, `git show`. Any revert command you recommend is **printed for the user to run themselves**.
- **Judge scope, not taste.** The bar is "did the task require this change," not "is this how I'd write it." Ugly-but-required code passes. Beautiful-but-unrequested code fails. General code quality belongs to `/code-review`, not here.
- **Every verdict cites evidence.** Name the file (and hunk or line where useful) and the specific task requirement it serves — or state plainly that no requirement covers it.
- **Audit the whole change set.** Staged, unstaged, untracked new files, and deleted files. A deletion is a change; an untracked file is a change.
- **Print the audit in the chat/terminal** using the exact output format below.

## Steps

1. **Establish the task.** Take it from the conversation, the plan, or the ticket. If the task is not clearly available, ask the user for it before auditing — a scope audit without a scope is worthless. Do not infer the task from the diff itself; that reasoning is circular and will rubber-stamp whatever was done.

2. **Collect the complete change set.**
   - `git status --porcelain` — staged, unstaged, untracked, deleted
   - `git diff` and `git diff --staged` — the actual hunks
   - `git diff --stat` and `git diff --staged --stat` — size overview
   - Read new untracked files directly; use `git show HEAD:<path>` to see what a deleted file contained
   - If this is not a git repository, or there is no meaningful baseline, say so and ask the user how the change set should be determined.

3. **Derive the scope boundary.** From the task, write down which files and areas the work legitimately requires *before* you look at verdicts. This is the yardstick.

4. **Classify every file** as `in-scope`, `out-of-scope`, or `core-touched`. These are not exclusive — a required change to an auth module is both in-scope and core-touched, and still deserves a flag.

5. **Run the core-functionality check** explicitly, even when every file looks in-scope:
   - Entry points — `main`, `index`, app bootstrap, routers
   - Shared utilities and helpers used across modules
   - Public APIs, exported signatures, and type contracts
   - Auth, permissions, session, and secrets handling
   - Database schemas, models, and migrations
   - Build and CI config, bundler and compiler settings
   - Dependency manifests and lockfiles
   - Environment and runtime config

6. **Hunt the usual drift patterns:**
   - Unrelated refactors riding along with the real change
   - Formatting or whitespace churn from an editor or formatter
   - Symbol renames that ripple past the task's boundary
   - Dependency added, removed, or version-bumped
   - Deleted code that the task never asked to remove
   - Debug leftovers — `console.log`, commented-out blocks, temporary flags
   - Generated, build, or lockfile output committed by accident

7. **Recommend a resolution for every out-of-scope item** — revert it, or isolate it into its own commit or PR. Print the exact commands for the user to run, and never run them.

## Verdict scale

- **Clean** — every change traces to a task requirement. Safe to commit.
- **Minor Drift** — small unrequested edits (formatting, a stray import, a debug line). Safe to strip or isolate; no behavior at risk.
- **Scope Violation** — meaningful unrelated changes, or core functionality touched without the task requiring it. Resolve before committing.

## Output format

Print exactly this structure:

````
### 🎯 Task Scope
- [what the task actually asked for]
- Legitimately touches: [files / areas the task requires]

### 📂 Change Set
[N modified · N added · N deleted · N untracked]

| File | Change | Verdict | Why |
|---|---|---|---|
| path/to/file | +12/-3 | ✅ in-scope / ⚠️ out-of-scope / 🔴 core-touched | [task requirement it serves, or why nothing covers it] |

**Always render the Change Set as a markdown table** — one row per file, never a vertical list of `File:` / `Change:` / `Verdict:` blocks and never separator lines between entries. Keep `Why` to one short line (roughly 12 words) so the columns stay readable; if a file needs a longer explanation, put the row in the table and add the detail underneath as a bullet.

### 🧠 Core Functionality Check
| Area | Touched? | Detail |
|---|---|---|
| Entry points | no | — |
| Shared utilities | yes | [what changed and whether the task required it] |
| Public APIs / exports | ... | ... |
| Auth / permissions | ... | ... |
| DB schema / migrations | ... | ... |
| Build / CI config | ... | ... |
| Dependencies / lockfile | ... | ... |

### 🚩 Out-of-Scope Findings
- `file:line` — [what changed] · [why the task does not require it] · [revert or isolate]

### ✅ Verdict
- Rating: [Clean / Minor Drift / Scope Violation]
- Before committing:
  - ...

### 🧾 Commands to run yourself (review each first)
```bash
git restore path/to/file          # revert unrequested change
git restore --staged path/to/file # unstage, keep for a separate commit
```
````

If the verdict is **Clean**, say so plainly and omit the Out-of-Scope and Commands sections rather than padding them with "none".

## Goal

Make sure a finished task changed exactly what it needed to change and nothing more — so the commit is honest, the diff is reviewable, and no core functionality was altered as a side effect. Runs after implementation (for example after `/ce-work` or a completed plan) and before `/mypr`.
