# my-agent-skills

A small, opinionated suite of **agent skills** for a clean pull-request workflow — from verifying a ticket, through reviewing the plan, triaging review findings, fixing them, and writing the PR.

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
/mas:mytask   /mas:myreviewer   /mas:mypr   /mas:myfindings   /mas:myfix
```

Prefer bare command names like `/mytask`, or not using plugins? See [Install](#install) for the manual global option. Full details in [Installing as a plugin](#option-1--install-as-a-plugin-recommended-works-across-all-projects).

## Design principles

All five skills share the same philosophy:

- **You stay in control of git.** No skill ever runs a state-modifying git command (`commit`, `push`, `checkout -b`, …). They *recommend* and print ready-to-copy commands; you run them.
- **Verify against reality, don't guess.** Skills inspect the actual code/diff before concluding.
- **Single responsibility.** Each skill does one job and hands off to the next.
- **Copy-paste friendly output.** Structured, terminal-ready output you can paste into GitHub.

## The workflow

Skills in **`this repo`** interleave with steps from the **`compound-engineering-plugin`** (shown in parentheses):

```
/mytask  →  (ce-plan)  →  /myreviewer  →  (ce-work)  →  (ce-code-review)  →  /myfindings  →  /myfix  →  /mypr
 verify      create        check plan       build         produce            triage &        implement   commit +
 ticket      plan          vs task                        findings           gate findings   fixes       PR text
[this repo] [plugin]      [this repo]      [plugin]       [plugin]           [this repo]     [this repo] [this repo]
```

If you're not using the plugin, substitute your own planning/build/review steps — the `my*` skills only assume that (a) a task exists, (b) a plan exists to review, and (c) review findings exist to triage.

## Skills

| Command | What it does |
|---|---|
| **`/mytask`** | Classifies a task as bug / feature / invalid, verifies it against the actual codebase before any work starts, assesses impact, and recommends a Git branch name. Recommendation only. |
| **`/myreviewer`** | Reviews a plan against its originating task — cross-checks every requirement, flags gaps, scope creep, and wrong assumptions, and gives a verdict (Aligned / Partially / Misaligned). Review only. |
| **`/myfindings`** | Parses PR review findings, categorizes them by severity (P0–P3), counts and lists them, flags which fixes are required (P0–P2), notes logic impact, and asks you to confirm before proceeding. Gate only. |
| **`/myfix`** | Implements the triaged findings in code — works P0→P2 (P3 optional), locates the affected code, applies fixes, and flags behavior/logic changes. Edits code; hands off to `/mypr` for git. |
| **`/mypr`** | Generates a one-liner commit message, a push command, and a filled-in PR brief for the current changes — printed for copy-paste. Never runs git. |

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
| `ce-code-review` (emits findings) | **`/myfindings`** → **`/myfix`** | Triage/gate the findings by severity, then implement the required (P0–P2) fixes. |
| — | **`/mytask`** (before `ce-plan`) | Verify the ticket is real and in scope before planning starts. |

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
/mas:myreviewer
/mas:mypr
/mas:myfindings
/mas:myfix
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
for s in mytask myreviewer mypr myfindings myfix; do
  ln -s ~/my-agent-skills/skills/"$s" ~/.claude/skills/"$s"
done
```

Prefer copies over symlinks? Swap the `ln -s` line for `cp -r`. Want just one skill? Link only that one. Update later with `cd ~/my-agent-skills && git pull`.

**After either option, restart Claude Code** (or start a new session). Run `/help` or type `/` and you should see the five skills listed.

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

### `/myreviewer` — check a plan against the task

Give it the task **and** the plan (e.g. the one `ce-plan` produced):

```
/myreviewer
Task: <the ticket / goal>
Plan: <the steps to review>
```

You get: a requirement-by-requirement table, flagged gaps / scope creep / wrong assumptions, and a verdict — **Aligned / Partially Aligned / Misaligned** — with specific fixes to make before building.

### `/myfindings` — triage review findings

Paste the findings from your review (e.g. `ce-code-review` output):

```
/myfindings
<paste the review comments / findings here>
```

You get: counts and a grouped list by severity (P0–P3), a note that **P0–P2 are required** (P3 optional), an impact note, and a confirmation question before you proceed.

### `/myfix` — implement the findings

Run it after `/myfindings` (it reuses those findings) or paste findings directly:

```
/myfix
```

It works P0 → P2 (P3 only if you ask), edits the affected code, flags anything that changes logic/behavior, verifies with the project's build/test command if one exists, then hands off to `/mypr`. **It edits code but never commits or pushes.**

### `/mypr` — commit message + PR brief

Run it when your changes are ready:

```
/mypr
```

You get a copy-ready block with a one-line commit message and a `git push` command, followed by a filled-in PR brief (title, description, changes, testing steps, checklist). Nothing is committed or pushed — you copy and run it.

### Typical end-to-end run

```
/mytask        →  verify the ticket, get a branch name
(ce-plan)      →  generate the plan
/myreviewer    →  confirm the plan matches the task
(ce-work)      →  build it
(ce-code-review) → get findings
/myfindings    →  triage P0–P3, confirm what must be fixed
/myfix         →  implement the P0–P2 fixes
/mypr          →  get the commit message + PR brief to paste
```

## Repository layout

```
.claude-plugin/         # plugin + marketplace manifests (makes this repo installable via /plugin)
├── plugin.json
└── marketplace.json
skills/
├── mytask/SKILL.md
├── myreviewer/SKILL.md
├── mypr/SKILL.md
├── myfindings/SKILL.md
└── myfix/SKILL.md
.branch-readmes/        # per-skill README templates that seed the skill/* branches (main only)
├── mytask.md
├── myreviewer.md
├── mypr.md
├── myfindings.md
└── myfix.md
```

## Branches

`main` holds the full suite. Each skill also has its own **standalone branch** containing just that one skill (plus `LICENSE` and a focused README), so anyone can grab a single skill without the rest:

| Branch | Contains |
|---|---|
| `main` | All five skills |
| `skill/mytask` | `skills/mytask/` only |
| `skill/myreviewer` | `skills/myreviewer/` only |
| `skill/mypr` | `skills/mypr/` only |
| `skill/myfindings` | `skills/myfindings/` only |
| `skill/myfix` | `skills/myfix/` only |

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
