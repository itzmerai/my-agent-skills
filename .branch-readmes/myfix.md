# /myfix

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myfix` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Take triaged PR review findings and actually fix them in the code. Works the required severities first (P0 → P1 → P2; P3 optional), locates the real code behind each finding, applies the fix, and clearly flags anything that changes business logic or existing behavior.

> **Edits code, but never runs git.** No `commit`, `add`, or `push`. When the fixes are done it hands off to `/mypr` for the commit message + PR text, which you run yourself.

Pairs with `/myfindings`: that skill triages and gates; this one implements.

## Install (Claude Code)

```bash
git clone --branch skill/myfix https://github.com/itzmerai/my-agent-skills.git myfix-skill
ln -s "$PWD/myfix-skill/skills/myfix" ~/.claude/skills/myfix
```

Restart Claude Code, then type `/` and you should see `/myfix`.

## Usage

Run it after `/myfindings` (it reuses those findings) or paste findings directly:

```
/myfix
```

It will:
- **Confirm scope** — fixes P0–P2 by default (P3 only if you ask), and flags anything ambiguous up front.
- **Fix each finding** most-critical-first, in the smallest correct way, matching the surrounding code.
- **Surface impact** — per-finding notes when a fix changes logic, alters behavior, or touches other call sites.
- **Verify** with the project's build/test/lint command if one exists, reporting the real result.
- **Hand off** — tells you to run `/mypr` for the commit + PR text.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
