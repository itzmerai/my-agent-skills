# /myfindings

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myfindings` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Parse PR review findings, categorize them by severity (P0–P3), summarize the counts, list each one, flag which fixes are **required (P0–P2 only)**, note logic/behavior impact, and ask you to confirm before proceeding. A validation checkpoint / quality gate.

> **Gate only.** This skill triages and asks — it does not fix code or run git. Pair it with `/myfix` to implement the fixes.

## Severity levels

- 🔴 **P0 (Critical)** — blocking: broken functionality, crashes, security flaws.
- 🟠 **P1 (High)** — major issues affecting core behavior or reliability.
- 🟡 **P2 (Medium)** — important improvements, edge cases, maintainability.
- ⚪ **P3 (Low)** — minor: style, formatting, non-blocking suggestions.

## Install (Claude Code)

```bash
git clone --branch skill/myfindings https://github.com/itzmerai/my-agent-skills.git myfindings-skill
ln -s "$PWD/myfindings-skill/skills/myfindings" ~/.claude/skills/myfindings
```

Restart Claude Code, then type `/` and you should see `/myfindings`.

## Usage

Paste the findings from your review (any review tool's output):

```
/myfindings
<paste the review comments / findings here>
```

You get:
- **Findings summary** — counts per severity (P0–P3).
- **Details** — each finding grouped under its severity with a one-line explanation.
- **Required fixes** — an explicit note that P0–P2 must be addressed (P3 optional).
- **Impact note** — whether fixing these may change business logic / affect behavior / strengthen the implementation.
- **Confirmation question** — before you proceed.

If your review tool already labels severities, `/myfindings` respects them; otherwise it infers and flags that it did.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
