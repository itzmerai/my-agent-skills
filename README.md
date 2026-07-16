# /mypr

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `mypr` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Generate a one-line commit message, a push command, and a filled-in PR brief for the current changes — all printed in the terminal for copy-paste.

> **Output only.** This skill never runs `git commit`, `git add`, or `git push`. It reads the working tree with read-only inspection (`git status`, `git diff`, `git log`) and prints text for you to copy and run.

## Install (Claude Code)

```bash
git clone --branch skill/mypr https://github.com/itzmerai/my-agent-skills.git mypr-skill
ln -s "$PWD/mypr-skill/skills/mypr" ~/.claude/skills/mypr
```

Restart Claude Code, then type `/` and you should see `/mypr`.

## Usage

Run it when your changes are ready:

```
/mypr
```

You get:
- **Commit + push block** — a one-line, imperative commit message and a `git push -u origin <branch>` command (with your real branch filled in), together in one copy-ready block.
- **PR brief** — title, description, related issues, changes, testing instructions, and a checklist, in a fenced Markdown block. Mobile-project rows (Android/iOS) are included only when the project is mobile/cross-platform.

Nothing is committed or pushed — you copy and run it.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
