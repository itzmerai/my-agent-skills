# /mytask

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `mytask` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Analyze a task, ticket, or bug report and decide whether it's a **real bug**, a **new feature**, or **invalid / already implemented** — verified against the actual codebase *before* any work starts — then recommend a Git branching strategy.

> **Git-cautious by design.** This skill never runs a state-modifying git command. It only *recommends* a branch and prints a ready-to-copy `git checkout` command for you to run yourself.

## Install (Claude Code)

```bash
git clone --branch skill/mytask https://github.com/itzmerai/my-agent-skills.git mytask-skill
ln -s "$PWD/mytask-skill/skills/mytask" ~/.claude/skills/mytask
```

Restart Claude Code, then type `/` and you should see `/mytask`.

## Usage

Paste or describe the ticket, then run:

```
/mytask
ENG-1529: Login button does nothing on Android when the form is empty.
```

You get:
- **Classification** — 🐞 Bug / ✨ Feature / ❌ Invalid, with a Confirmed / Not Reproducible / Already Implemented status.
- **Findings** — what was actually checked in the code and what was found.
- **Project scope** — app only / backend-dashboard only / both.
- **Impact** — affected areas, priority, risk.
- **Branch recommendation** — a clean branch name (`fix/…`, `feat/…`, `chore/…`) plus a copy-ready `git checkout` command (not executed).

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
