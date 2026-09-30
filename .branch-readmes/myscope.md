# /myscope

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myscope` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Audit the complete change set against the task that caused it. This is the QA gate between "implementation done" and "commit / PR" — it answers one question: **did anything change that this task did not require?**

> **Audit only.** This skill does not write, fix, or revert code, and it never runs a state-modifying git command. It prints the `git restore` commands; you run them.

## Install (Claude Code)

```bash
git clone --branch skill/myscope https://github.com/itzmerai/my-agent-skills.git myscope-skill
ln -s "$PWD/myscope-skill/skills/myscope" ~/.claude/skills/myscope
```

Restart Claude Code, then type `/` and you should see `/myscope`.

## Usage

Run it when implementation is finished, before you commit:

```
/myscope
```

It takes the task from the conversation — if there isn't one, it asks rather than inferring the task from the diff, which would only rubber-stamp whatever was done.

You get:
- **Task scope** — what the task asked for, and which files it legitimately touches.
- **Change set** — every modified, added, deleted, and untracked file with a verdict: ✅ in-scope / ⚠️ out-of-scope / 🔴 core-touched, each justified against a task requirement.
- **Core functionality check** — entry points, shared utilities, public APIs, auth, DB schema/migrations, build/CI config, and dependencies, checked explicitly even when the diff looks clean.
- **Out-of-scope findings** — unrelated refactors, formatting churn, symbol renames, dependency drift, deleted code, and debug leftovers, each with a recommendation to revert or isolate.
- **Verdict** — **Clean / Minor Drift / Scope Violation**, with ready-to-copy `git restore` commands.

Worth re-running after a fix pass, since fixes can introduce their own drift.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
