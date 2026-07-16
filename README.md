# /myreviewer

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myreviewer` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Review a plan against its originating task and judge whether the plan **truly addresses the task** and stays in scope. This is the QA gate between "plan created" and "start building."

> **Review only.** This skill does not create, rewrite, or implement the plan, and it never runs git. It suggests specific fixes; you apply them.

## Install (Claude Code)

```bash
git clone --branch skill/myreviewer https://github.com/itzmerai/my-agent-skills.git myreviewer-skill
ln -s "$PWD/myreviewer-skill/skills/myreviewer" ~/.claude/skills/myreviewer
```

Restart Claude Code, then type `/` and you should see `/myreviewer`.

## Usage

Give it the task **and** the plan (e.g. the one your planning skill produced):

```
/myreviewer
Task: <the ticket / goal>
Plan: <the steps to review>
```

You get:
- **Task intent** — the real requirements extracted from the task.
- **Plan vs task** — a requirement-by-requirement table marking each ✅ covered / ❌ gap / ⚠️ scope creep / 🔧 misalignment.
- **Issues** — gaps, scope creep, and wrong/unstated assumptions, each grounded in the specific requirement and plan step.
- **Verdict** — **Aligned / Partially Aligned / Misaligned**, with a short list of fixes to make before building.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
