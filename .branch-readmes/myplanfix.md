# /myplanfix

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myplanfix` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Fix a plan that `/myreviewer` rated **Partially Aligned** or **Misaligned**, so the next review comes back **Aligned**.

> **Plan documents only.** This skill edits the plan file. It never writes production code and never runs a state-modifying git command.

## Install (Claude Code)

```bash
git clone --branch skill/myplanfix https://github.com/itzmerai/my-agent-skills.git myplanfix-skill
ln -s "$PWD/myplanfix-skill/skills/myplanfix" ~/.claude/skills/myplanfix
```

Restart Claude Code, then type `/` and you should see `/myplanfix`.

## The loop it exists to break

```
/myreviewer → "Partially Aligned, fix X"  →  fix X
/myreviewer → "Partially Aligned, fix Y"  →  fix Y
/myreviewer → "Partially Aligned, fix Z"  →  ...
```

It never converges, because each pass patches only what the reviewer happened to name and a fresh batch surfaces next time.

So `/myplanfix` re-reads the ticket line by line and finds the misalignments **itself** — a safety net under the reviewer's list, not a copy of it. Everything genuinely wrong gets fixed in one pass, which is what makes the next review come back clean.

## Usage

```
/myreviewer   →  Partially Aligned
/myplanfix    →  fixes the plan
/myreviewer   →  Aligned
```

It doesn't run `/myreviewer` for you — it repairs, reports, and tells you to run it.

## It is not a second reviewer

No rating, no alignment table, no verdict. `/myreviewer` owns that. This skill reads, repairs, and reports what it changed.

Before touching anything, every reviewer finding gets a ruling:

| Ruling | What happens |
|---|---|
| ✅ **Valid** | fixed |
| 🔁 **Already addressed** | rejected — cites the plan section that covers it |
| ⛔ **Contradicts the ticket** | rejected — cites the ticket line it would break |
| ❓ **Ambiguous** | flagged for the ticket's author, plan left untouched |
| ⏭️ **Out of scope** | rejected — belongs in its own ticket |

**Making no edits is a valid outcome.** If the plan is right and the findings don't hold up, it says so with the evidence and changes nothing. Editing a correct plan to appease a review makes it worse.

## Evidence, in order of authority

1. **The ticket governs scope** — if the ticket doesn't ask for it, the plan shouldn't plan it
2. **The PRD confirms intent**, where the project keeps a copy
3. **The codebase decides feasibility** and which conventions apply

A missing PRD is not a gap — plenty of projects don't keep one, and the ticket plus the codebase's existing structure stand in. Where the ticket and PRD contradict each other, that's surfaced rather than quietly resolved in favour of one.

## It decides, it doesn't punt

When the plan has to choose — extend a table or add one, reuse a service or write a new one — it reads the codebase, picks the option matching what's already there, and **states the choice and the reason**. Leaving the decision open is how a plan passes review and still stalls at build time.

It also never renumbers implementation units. `ce-work` tracks tasks against `U1`, `U2`…, so a new requirement extends the unit it belongs to, or appends a new U-ID at the end.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
