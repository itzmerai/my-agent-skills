# /myrepro

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myrepro` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Turn a ticket into steps someone else could run. Every step is written twice over: what it does **today**, and what it must do **once the work is done**. That pairing is the whole point — you cannot tell a fix from a coincidence without knowing what the broken state looked like.

> **Read-only.** This skill does not implement the ticket, fix code, or run a state-modifying git command. It reads code and may run existing tests or read-only commands to confirm the before-state.

## Install (Claude Code)

```bash
git clone --branch skill/myrepro https://github.com/itzmerai/my-agent-skills.git myrepro-skill
ln -s "$PWD/myrepro-skill/skills/myrepro" ~/.claude/skills/myrepro
```

Restart Claude Code, then type `/` and you should see `/myrepro`.

## Usage

Run it right after the ticket is verified, while the code is still broken:

```
/myrepro
ENG-1529: Login button does nothing on Android when the form is empty.
```

You get:
- **Classification** — bug or feature, which decides the shape of the plan.
- **Setup** — environment, account and role, data required, and the exact place to start.
- **A verification matrix** — every row carrying both halves: `Expect BEFORE` (today's code) and `Expect AFTER` (work done), across the happy path, edge cases, and negative paths, each mapped to a defect or acceptance criterion.
- **A real repro attempt** — the before-state marked **confirmed**, **not reproducible**, or **not runnable here**, with the command used and the actual output. A ticket that does not reproduce is a finding, not a detail to work around.
- **Regression checks** — nearby behavior at risk, where before and after are deliberately identical.
- **A suggested automated test** — file path, framework detected from the repo, and the assertion that would have caught it.
- **Gaps** — anything ambiguous or missing, phrased as questions for the ticket's author instead of guessed.

Write the plan early; run its AFTER column at the end, once the diff is confirmed clean.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
