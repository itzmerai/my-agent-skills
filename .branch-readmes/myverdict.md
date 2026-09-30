# /myverdict

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myverdict` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Cross-verify review findings against the actual code before anyone acts on them. This is the gate between "findings triaged" and "findings implemented" — it answers two questions per finding: **is this real?** and **does this task want it?**

> **Read-only.** This skill does not fix code, write the review, or run a state-modifying git command. It rules on findings; `/myfix` implements the ones that survive.

## Install (Claude Code)

```bash
git clone --branch skill/myverdict https://github.com/itzmerai/my-agent-skills.git myverdict-skill
ln -s "$PWD/myverdict-skill/skills/myverdict" ~/.claude/skills/myverdict
```

Restart Claude Code, then type `/` and you should see `/myverdict`.

## Usage

Give it the task and the findings (from your triage step, a code-review tool, or pasted PR comments):

```
/myverdict
```

You get:
- **Findings cross-check** — every finding opened against the real code and marked ✅ confirmed / 🟡 partly right / ⛔ not reproducible / 🔁 already handled / ❓ needs runtime proof.
- **Two-axis classification** — **scope** (in-scope vs. belongs in its own ticket) and **impact** (fixes a defect / strengthens the implementation / adds no value).
- **Priority corrections** — mislabeled severities flagged in both directions, including the inflated P0 and the "minor" note that is really a data-loss bug.
- **A verdict per finding** — **address now / defer as follow-up / reject**, every rejection carrying a reason a reviewer can read.
- **A hand-off list** — exactly what to fix now, what becomes a follow-up ticket, and what is closed.

The point is that a finding is a claim, not a fact. Reviewers — human and AI alike — report issues that a guard upstream already covers, or that the code simply does not do.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
