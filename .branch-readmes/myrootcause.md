# /myrootcause

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myrootcause` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

A gate between *thinking* you know why something broke and *saying so*. An explanation that sounds right and an explanation that is right look identical in a ticket comment — the difference only shows up when someone acts on it.

> **Read-only.** This skill does not fix code or run a state-modifying git command. It reads code, logs and config, and may run read-only commands and existing tests to gather evidence.

## Install (Claude Code)

```bash
git clone --branch skill/myrootcause https://github.com/itzmerai/my-agent-skills.git myrootcause-skill
ln -s "$PWD/myrootcause-skill/skills/myrootcause" ~/.claude/skills/myrootcause
```

Restart Claude Code, then type `/` and you should see `/myrootcause`.

## Usage

Run it after the bug is confirmed, before the explanation reaches a ticket:

```
/myrootcause
ENG-1529: totals are wrong on the dashboard.
Suspected cause (from Claude): the rounding change in PR #123.
```

Hand it a cause and it audits that one. Hand it nothing and it investigates enough to form one, then audits it the same way.

## The five checks

| # | Check | Catches |
|---|---|---|
| 1 | **Timeline** — report date vs. ship date of the suspected change | Blaming a commit that shipped *after* the report |
| 2 | **Environment** — what commit and config the reported environment actually runs | A fix already on `main` that was never deployed there |
| 3 | **Reproduce where reported**, not only locally | "Works on my machine" solving a different problem |
| 4 | **Symptom coverage** — the cause must explain every symptom, *including what still works* | A plausible cause that can't account for the neighbouring feature being fine |
| 5 | **Evidence** — every claim cites a SHA, `file:line`, log timestamp, or a config value read from the real environment | A fluent, convincing, wrong explanation — including one an AI wrote |

Check 5 is a hard rule, not advice. Uncited reasoning is marked **Unproven** no matter how sound it reads.

## You get

- **A verdict** — **Proven**, **Disproven**, or **Unproven**, with the two lines of reasoning behind it.
- **A separate ruling: is this a code problem at all?** Stale deployment, config drift, data-specific state, wrong role, an already-shipped fix, a third-party outage. These are the cheapest causes to check and the most embarrassing to miss, and they are invisible if you only read code. When the answer is no, the skill says so and **ends the chain** — there's nothing to fix and no AFTER column to write.
- **A symptom coverage table** with a row for everything adjacent that *still works*. One unexplained row means the cause is incomplete or wrong.
- **An evidence ledger** — every assertion with its citation, and any that couldn't be cited struck out.
- **A handoff** — on a proven code cause, the environment, role, commit and trigger that `/myrepro` needs for its BEFORE column, captured while the code is still broken.

## Why it runs on every bug

"Obvious" is a feeling you have before you've checked. A ticket says *Save does nothing*, you find the dropped `onClick` in last week's refactor, and it looks closed — until check 4 asks why **Edit Profile still works**, which goes through the same handler. The cause was wrong and the bug looked like a one-liner the whole time.

Genuinely obvious bugs cost about thirty seconds here. The ones that only looked obvious are the entire return.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
