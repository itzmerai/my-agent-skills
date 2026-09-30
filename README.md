# my-agent-skills

A small, opinionated suite of **agent skills** for a clean pull-request workflow — from verifying a ticket, through reviewing the plan, reviewing the pull request, triaging review findings, fixing them, and writing the PR. It also bundles a few standalone, general-purpose skills (frontend design, SEO, and text humanizing) — see [Additional skills](#additional-skills).

Each skill is a single `SKILL.md`: a name, a trigger-rich description, and a set of rules + steps the agent follows. They're written in the [Claude Code skill format](https://docs.claude.com/en/docs/claude-code/skills), but the instructions are plain Markdown and **portable to any agentic coding tool** — drop them into whatever your agent reads for custom instructions, or adapt the steps directly.

> **Built for developers who are cautious about git.** Every skill is deliberately **read-only when it comes to git** — none of them ever run `commit`, `add`, `push`, `checkout -b`, `reset`, or any other state-modifying command. They inspect (`git status`, `git diff`, `git log`), then *print ready-to-copy commands* for you to run yourself. If you work across machines with device-specific/local-only files, or you just want the final say before anything touches history or a remote, these skills never publish or rewrite anything behind your back.

> **Designed to pair with [`compound-engineering-plugin`](https://github.com/EveryInc/compound-engineering-plugin).** These skills are the human-in-the-loop *checkpoints* around that plugin's heavier automation. The plugin's `ce-plan` creates the plan, `ce-work` builds it, and `ce-code-review` produces the findings; the `my*` skills verify the ticket before planning, gate the plan against the task, and triage → fix → ship the review findings. They also work standalone if you don't use the plugin.

## Quick start

Install as a Claude Code plugin — two lines, works in **every project**, and the whole team uses the same install:

```
/plugin marketplace add itzmerai/my-agent-skills
/plugin install mas
```

Restart Claude Code, then invoke any skill (namespaced under `mas`):

```
/mas:mytask   /mas:myrepro   /mas:myreviewer   /mas:myscope
/mas:mycodereview   /mas:myfindings   /mas:myverdict   /mas:myfix   /mas:mypr
```

Prefer bare command names like `/mytask`, or not using plugins? See [Install](#install) for the manual global option. Full details in [Installing as a plugin](#option-1--install-as-a-plugin-recommended-works-across-all-projects).

## Design principles

All nine workflow skills share the same philosophy:

- **You stay in control of git.** No skill ever runs a state-modifying git command (`commit`, `push`, `checkout -b`, …). They *recommend* and print ready-to-copy commands; you run them.
- **Verify against reality, don't guess.** Skills inspect the actual code/diff before concluding.
- **Single responsibility.** Each skill does one job and hands off to the next.
- **Copy-paste friendly output.** Structured, terminal-ready output you can paste into GitHub.

## The workflow

Skills in **`this repo`** interleave with steps from the **`compound-engineering-plugin`** (shown in parentheses):

```
── plan ────────────────────────────────────────────────────
/mytask   →  /myrepro      →  (ce-plan)  →  /myreviewer
 verify       before/after     create       plan
 ticket       test steps       plan         vs task
[this repo]  [this repo]      [plugin]     [this repo]

── build & prove ───────────────────────────────────────────
(ce-work)  →  /myscope     →  /myrepro     →  /mypr
 build        diff            run AFTER       commit msg
              vs task         column          + PR brief
[plugin]     [this repo]     [this repo]     [this repo]
                                                  ↓  PR now exists
── review loop ─────────────────────────────────────────────
/mycodereview →  /myfindings   →  /myverdict    →  /myfix
 review the      triage           is it real,       implement
 open PR         P0–P3            in scope?         what survives
[this repo]     [this repo]      [this repo]      [this repo]
                                                       ↓
                        /myfix ends with its own one-line commit
                        message + push command — /mypr is only for
                        opening the PR. Re-run /myscope + /myrepro
                        first if the fixes were non-trivial.
```

Two skills run twice on purpose. `/myrepro` is written early to capture the broken "before" state while the code is still broken — evidence you cannot recover once the fix lands — and its **AFTER** column is run later to prove the work landed. `/myscope` runs before the commit, and again after `/myfix`, because fixes drift too.

`/mypr` runs **once**, to open the pull request — it sits before `/mycodereview` because a PR has to exist before it can be reviewed. Fix commits afterwards don't need another PR brief, so `/myfix` prints its own commit message and push command. If you would rather catch problems before pushing, run `ce-code-review` (or `/code-review`) on the working diff during **build & prove** — it feeds `/myfindings` exactly the same way.

If you're not using the plugin, substitute your own planning/build/review steps — the `my*` skills only assume that (a) a task exists, (b) a plan exists to review, and (c) review findings exist to triage.

## Skills

| Command | What it does |
|---|---|
| **`/mytask`** | Classifies a task as bug / feature / invalid, verifies it against the actual codebase before any work starts, assesses impact, and recommends a Git branch name. Recommendation only. |
| **`/myrepro`** | Turns a ticket into before/after verification steps — classifies it bug vs feature, writes exact repro or baseline steps, runs them against today's code to confirm the before state, and pairs every step with what to expect once the work is done. Read-only. |
| **`/myreviewer`** | Reviews a plan against its originating task — cross-checks every requirement, flags gaps, scope creep, and wrong assumptions, and gives a verdict (Aligned / Partially / Misaligned). Review only. |
| **`/myscope`** | Audits the finished diff against the task — per-file verdict (in-scope / out-of-scope / core-touched), flags unrelated refactors, formatting churn, dependency and config drift, and checks whether core functionality was touched. Audit only. |
| **`/mycodereview`** | Reviews an already-opened GitHub pull request in a single pass — gathers the diff via `gh`, then reports an overview, code quality notes, suggestions, and risks. No subagents, no workflow fan-out. For your local working diff use `/code-review` instead. Review only. |
| **`/myfindings`** | Parses PR review findings, categorizes them by severity (P0–P3), counts and lists them, flags which fixes are required (P0–P2), notes logic impact, and asks you to confirm before proceeding. Gate only. |
| **`/myverdict`** | Cross-verifies each review finding against the actual code, classifies it by scope (in / out) and impact (fixes a defect / strengthens / no value), corrects mislabeled priorities, and rules address now / defer / reject. Read-only. |
| **`/myfix`** | Implements the triaged findings in code — works P0→P2 (P3 optional), locates the affected code, applies fixes, and flags behavior/logic changes. Ends with a one-line commit message + push command for the fixes. Edits code, never runs git. |
| **`/mypr`** | Generates a one-liner commit message, a push command, and a filled-in PR brief for the current changes — printed for copy-paste. Never runs git. |

## Additional skills

Alongside the PR-workflow core, this repo bundles a few **standalone, general-purpose skills** adapted from the community. They aren't part of the `/mytask → /mypr` pipeline — use them on their own. Like the core skills, none of them run git for you.

| Command | What it does | Adapted from |
|---|---|---|
| **`/myfrontend-design`** | Builds distinctive, production-grade frontend UIs (components, pages, landing pages) with a bold, intentional aesthetic direction — avoids generic "AI slop" design. | [claude-code-templates](https://github.com/davila7/claude-code-templates) (MIT) |
| **`/myseo-optimizer`** | On-page & technical SEO guidance: keyword strategy, meta tags, schema markup, Core Web Vitals, content structure, and an SEO checklist. | [claude-code-templates](https://github.com/davila7/claude-code-templates) (MIT) |
| **`/myhumanizer`** | Removes tell-tale signs of AI-generated writing (inflated symbolism, promotional language, em-dash overuse, rule-of-three, AI vocabulary) and adds natural voice. | [@blader/humanizer](https://github.com/blader/humanizer) |

> These skills are third-party adaptations, credited above and in each skill's `SKILL.md`. Their original terms apply.

## Pairing with compound-engineering-plugin

These skills were built to complement [EveryInc's `compound-engineering-plugin`](https://github.com/EveryInc/compound-engineering-plugin), acting as the human-in-the-loop gates around its automation.

### Installing the plugin

In Claude Code, add the marketplace and install the plugin:

```
/plugin marketplace add EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

This gives you `ce-plan`, `ce-work`, `ce-code-review`, and the rest of the `ce-*` skills. To update an existing install (the plugin moved to a root-native layout — refresh the marketplace *before* updating, or `/plugin update` alone keeps you on the old version):

```
/plugin marketplace update compound-engineering-plugin
/plugin update compound-engineering
```

### How the two fit together

Once the plugin is installed, the `my*` skills slot in as gates around it:

| Plugin step | Followed by | Why |
|---|---|---|
| `ce-plan` (creates a plan) | **`/myreviewer`** | Confirm the plan actually addresses the task before you let `ce-work` build it. |
| `ce-work` (builds the change) | **`/myscope`** | Audit the resulting diff against the task before review — catch scope creep and accidental core-functionality edits early. |
| `ce-code-review` (emits findings) | **`/myfindings`** → **`/myverdict`** → **`/myfix`** | Triage by severity, cross-verify each finding against the code and rule on it, then implement only what survives. |
| — | **`/mytask`** (before `ce-plan`) | Verify the ticket is real and in scope before planning starts. |
| — | **`/myrepro`** (after `/mytask`) | Capture how to reproduce and verify it, while the before-state still exists. |

`/myfindings` is built to consume review output like `ce-code-review`'s — paste its findings and it categorizes them into P0–P3. If your review tool already labels severities, `/myfindings` respects them; otherwise it infers and flags that it did.

You can use this repo without the plugin — the `my*` skills don't depend on it.

## Install

This repo is a **Claude Code plugin marketplace**, so the easiest way to install — for you and your collaborators — is via `/plugin`. A manual global install is also documented below.

### Option 1 — Install as a plugin (recommended, works across all projects)

In Claude Code, add this repo as a marketplace and install the plugin:

```
/plugin marketplace add itzmerai/my-agent-skills
/plugin install mas
```

That's it — no cloning, no symlinks. The skills are installed at the user level, so they're available in **every project**. Share those two lines with collaborators and they're set up in seconds.

Plugin skills are namespaced under the short plugin name **`mas`** (short for *my-agent-skills*), so the commands are:

```
/mas:mytask
/mas:myrepro
/mas:myreviewer
/mas:myscope
/mas:mycodereview
/mas:myfindings
/mas:myverdict
/mas:myfix
/mas:mypr
```

To update to the latest version later:

```
/plugin marketplace update my-agent-skills
/plugin update mas
```

### Option 2 — Manual global install (bare `/mytask` command names)

Prefer the short, un-namespaced commands (`/mytask` instead of `/mas:mytask`)? Clone the repo and symlink the skills into your global skills directory:

```bash
git clone https://github.com/itzmerai/my-agent-skills.git ~/my-agent-skills
mkdir -p ~/.claude/skills
for s in mytask myrepro myreviewer myscope mycodereview myfindings myverdict myfix mypr; do
  ln -s ~/my-agent-skills/skills/"$s" ~/.claude/skills/"$s"
done
```

Prefer copies over symlinks? Swap the `ln -s` line for `cp -r`. Want just one skill? Link only that one. Update later with `cd ~/my-agent-skills && git pull`.

**After either option, restart Claude Code** (or start a new session). Run `/help` or type `/` and you should see the nine skills listed.

> Requires **Claude Code**. To also use the companion `ce-*` skills, install the [compound-engineering-plugin](#installing-the-plugin) above — but the `my*` skills work on their own too.

### Other agentic tools

Each `SKILL.md` is self-contained Markdown. Copy the rules/steps into your tool's custom-instruction or rules file (e.g. a `.cursor/rules` file, a system prompt, or a project doc your agent reads), keeping the trigger phrases so the agent knows when to apply it.

## Usage

Each skill is triggered by its slash command, or just by describing what you want in natural language (the descriptions carry trigger phrases). All output is printed for you to read/copy — and **no skill runs git for you**; when git is needed, it prints the command and you run it.

### `/mytask` — verify a ticket before you build

Paste or describe the ticket, then run the command:

```
/mytask
ENG-1529: Login button does nothing on Android when the form is empty.
```

You get: a classification (bug / feature / invalid), what was actually checked in the code, an impact assessment, and a recommended branch name with a ready-to-copy `git checkout` command.

### `/myrepro` — before/after test steps from a ticket

Run it right after `/mytask`, while the code is still broken:

```
/myrepro
ENG-1529: Login button does nothing on Android when the form is empty.
```

It classifies the ticket bug vs feature, finds the real code path, and produces a **verification matrix** where every row carries both halves — what you see on today's code, and what you must see once the work is done — covering happy path, edge cases, and negative paths. It tries to actually run the repro and reports **confirmed / not reproducible / not runnable here** (a ticket that doesn't reproduce is a finding, not a detail). You also get regression checks and a suggested automated test. **Read-only.**

Run the plan's AFTER column at the end, once `/myscope` says the diff is clean.

### `/myreviewer` — check a plan against the task

Give it the task **and** the plan (e.g. the one `ce-plan` produced):

```
/myreviewer
Task: <the ticket / goal>
Plan: <the steps to review>
```

You get: a requirement-by-requirement table, flagged gaps / scope creep / wrong assumptions, and a verdict — **Aligned / Partially Aligned / Misaligned** — with specific fixes to make before building.

### `/myscope` — audit the diff against the task

Run it when implementation is done, before you commit:

```
/myscope
```

It reads the full change set (staged, unstaged, untracked, deleted), measures every file against the task, and returns a per-file verdict — **in-scope / out-of-scope / core-touched** — plus an explicit core-functionality check (entry points, shared utilities, public APIs, auth, DB schema, build/CI, dependencies) and a verdict of **Clean / Minor Drift / Scope Violation**. Anything unrequested comes with a printed `git restore` command for **you** to run. **Read-only on git.**

Worth re-running after `/myfix`, since fixes can introduce their own drift.

### `/mycodereview` — review an opened pull request

Run it once the PR exists (`/mypr` gets you there). Pass a PR number or URL, optionally followed by extra instructions:

```
/mycodereview 383
/mycodereview https://github.com/owner/repo/pull/383
/mycodereview 383 focus on the auth changes
```

You get: an overview of what the PR does, notes on code quality and style, specific suggestions, and potential issues/risks — focused on correctness, project conventions, performance, test coverage, and security. Run it with no argument and it lists the open PRs and asks which to review. Its output feeds straight into `/myfindings`.

Requires the [`gh` CLI](https://cli.github.com/), authenticated. The PR's diff is the only scope — for your uncommitted working changes, use Claude Code's built-in `/code-review` or `ce-code-review` instead.

### `/myfindings` — triage review findings

Paste the findings from your review (e.g. `ce-code-review` output):

```
/myfindings
<paste the review comments / findings here>
```

You get: counts and a grouped list by severity (P0–P3), a note that **P0–P2 are required** (P3 optional), an impact note, and a confirmation question before you proceed.

### `/myverdict` — rule on each finding before fixing it

Run it after `/myfindings` (it reuses those findings) or paste a review directly:

```
/myverdict
```

It opens the code behind every finding and confirms the claim is real — reviewers, human or AI, report things that are already handled or simply wrong. Each finding gets a verdict of **address now / defer as follow-up / reject**, decided on two axes: **scope** (does this task own it?) and **impact** (fixes a defect / strengthens / no value). Mislabeled priorities are corrected in both directions, and every rejection carries a reason. Only what survives goes to `/myfix`. **Read-only on code and git.**

### `/myfix` — implement the findings

Run it after `/myfindings` (it reuses those findings) or paste findings directly:

```
/myfix
```

It works P0 → P2 (P3 only if you ask), edits the affected code, flags anything that changes logic/behavior, verifies with the project's build/test command if one exists, then prints a one-line commit message and `git push` for the fixes. **It edits code but never commits or pushes** — `/mypr` is only for opening the PR in the first place.

### `/mypr` — commit message + PR brief

Run it when your changes are ready:

```
/mypr
```

You get a copy-ready block with a one-line commit message and a `git push` command, followed by a filled-in PR brief (title, description, changes, testing steps, checklist). Nothing is committed or pushed — you copy and run it.

### Typical end-to-end run

```
/mytask        →  verify the ticket, get a branch name
/myrepro       →  write before/after test steps, confirm the broken state
(ce-plan)      →  generate the plan
/myreviewer    →  confirm the plan matches the task
(ce-work)      →  build it
/myscope       →  audit the diff against the task
/myrepro       →  run the AFTER column to prove it works
/mypr          →  commit message + PR brief  →  open the PR
/mycodereview  →  review the opened PR
/myfindings    →  triage P0–P3, confirm what must be fixed
/myverdict     →  verify each finding is real, in scope, worth doing
/myfix         →  implement what survives
/myscope       →  re-audit: fixes drift too
/myrepro       →  re-run AFTER: fixes regress too
                  then run the commit + push commands /myfix printed
```

## Repository layout

```
.claude-plugin/         # plugin + marketplace manifests (makes this repo installable via /plugin)
├── plugin.json
└── marketplace.json
skills/
├── mytask/SKILL.md         # ── PR-workflow core ──
├── myrepro/SKILL.md
├── myreviewer/SKILL.md
├── myscope/SKILL.md
├── mypr/SKILL.md
├── mycodereview/SKILL.md
├── myfindings/SKILL.md
├── myverdict/SKILL.md
├── myfix/SKILL.md
├── myfrontend-design/SKILL.md   # ── additional standalone skills ──
├── myseo-optimizer/SKILL.md
└── myhumanizer/SKILL.md
.branch-readmes/        # per-skill README templates that seed the skill/* branches (main only)
├── mytask.md
├── myrepro.md
├── myreviewer.md
├── myscope.md
├── mypr.md
├── mycodereview.md
├── myfindings.md
├── myverdict.md
├── myfix.md
├── myfrontend-design.md
├── myseo-optimizer.md
└── myhumanizer.md
```

## Branches

`main` holds the full suite. Each skill also has its own **standalone branch** containing just that one skill (plus `LICENSE` and a focused README), so anyone can grab a single skill without the rest:

| Branch | Contains |
|---|---|
| `main` | All skills (the PR-workflow core + additional standalone skills) |
| `skill/mytask` | `skills/mytask/` only |
| `skill/myrepro` | `skills/myrepro/` only |
| `skill/myreviewer` | `skills/myreviewer/` only |
| `skill/myscope` | `skills/myscope/` only |
| `skill/mypr` | `skills/mypr/` only |
| `skill/mycodereview` | `skills/mycodereview/` only |
| `skill/myfindings` | `skills/myfindings/` only |
| `skill/myverdict` | `skills/myverdict/` only |
| `skill/myfix` | `skills/myfix/` only |
| `skill/myfrontend-design` | `skills/myfrontend-design/` only |
| `skill/myseo-optimizer` | `skills/myseo-optimizer/` only |
| `skill/myhumanizer` | `skills/myhumanizer/` only |

Clone a single skill directly:

```bash
git clone --branch skill/myfix https://github.com/itzmerai/my-agent-skills.git
```

### Maintaining the branches

**`main` is the source of truth.** Always edit a skill on `main`, then propagate the change to its branch — never the other way around. After updating, say, `skills/mypr/SKILL.md` on `main`:

```bash
# on main, after committing the SKILL.md change
git checkout skill/mypr
git checkout main -- skills/mypr            # pull just this skill's folder from main
git commit -m "sync mypr from main"
git push
git checkout main
```

If a skill branch's focused README needs updating too, edit `.branch-readmes/<skill>.md` on `main`, then on the branch run `git checkout main -- .branch-readmes/mypr.md && cp .branch-readmes/mypr.md README.md && rm -rf .branch-readmes && git commit -am "sync mypr readme"`.

> The `.branch-readmes/` folder on `main` holds the per-branch README templates. It exists only to seed the skill branches; the branch-creation script copies the right one to `README.md` and drops the folder, so it never ships on a skill branch.

## Contributing

Issues and PRs welcome — new skills that fit the "verify, don't guess; you control git; one job each" philosophy are especially appreciated.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
