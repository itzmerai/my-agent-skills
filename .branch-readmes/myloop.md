# /myloop

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myloop` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Run the PR review pipeline end to end instead of invoking five skills by hand after every review.

> **Orchestration only.** This skill chains the existing skills and contributes no review, triage, or fix logic of its own. It never commits or pushes — `/myfix` edits the working tree and the git commands print at the end for you to run.

## Install (Claude Code)

```bash
git clone --branch skill/myloop https://github.com/itzmerai/my-agent-skills.git myloop-skill
ln -s "$PWD/myloop-skill/skills/myloop" ~/.claude/skills/myloop
```

Restart Claude Code, then type `/` and you should see `/myloop`.

## The chain

```
/mycodereview  → review printed            (always continues)
      ↓
/myfindings    → P0–P3 triage printed
      ↓ 0 P0–P2?  STOP
/myverdict     → rulings printed
      ↓ severity corrected (either direction)?
      │     → /myfindings  (2nd verification, updated counts)
      │          ↓ 0 P0–P2 now?  STOP
      │     → /myverdict   (then continue — bounded)
      ↓
/myfix         → edits, or a before/after correction for non-code findings
      ↓
/myscope       → audits the fixes against the address-now list
      ↓
FINAL BLOCK    → /myscope result  →  git commands  →  PR description before/after
```

Each skill prints its full result before the next one starts. Nothing is summarised or batched — the point of the chain is that every stage stays readable.

## The control-flow decisions

**`/mycodereview` always continues**, even with zero findings. The triage stating "no findings" is the confirmation worth having; a chain that dies silently at step 1 tells you nothing.

**The chain stops after `/myfindings`** when nothing is P0–P2. There is no verdict to reach.

**A corrected severity sends it back to `/myfindings`.** When `/myverdict` demotes a P2 to P3 — or promotes a mislabelled "minor" that is really a data-loss bug — the triage counts are stale. The chain re-runs `/myfindings` for a second verification, and if the corrected set is now empty it ends there.

**That loop is bounded: a finding's severity may be corrected once.** If `/myverdict` changes the same finding's severity a second time, the cycle halts, the finding is carried at its latest value, and the unsettled severity is reported. A P2 and a P3 could otherwise trade places indefinitely.

**`/myscope` measures the fixes against the address-now list**, not the original ticket. The original work was already judged by `/mycodereview`, and auditing against the ticket drags already-committed files into the report and buries the actual changes. The narrower question is the useful one: *did `/myfix` change only what it was told to?*

**Past a stop condition, no steps are skipped.** Even when `/myverdict` rejected everything and `/myfix` has nothing to edit, the remaining steps run and say so.

## The final block

Three things in one place at the end, instead of scattered up the scrollback:

1. The `/myscope` verdict, repeated in full
2. The git commands — `add`, `commit`, `push` — for you to run
3. The PR description correction, **before and after**, when a finding raised one

A pull request description finding is not a code change, so `/myfix` does not rewrite it. It states what to change and you apply it. That section prints wherever the chain terminates, including an early stop where there is no scope audit and no diff to commit.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
