# /mychecklist

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `mychecklist` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Verify a branch against its ticket, one requirement per line. Every line is a statement that is either true or false of the code, and every verdict carries the file that proves it.

> **Review only.** This skill does not edit, stage, or commit code. Read-only git — `git status`, `git diff`, `git log`, `git show`.

## Install (Claude Code)

```bash
git clone --branch skill/mychecklist https://github.com/itzmerai/my-agent-skills.git mychecklist-skill
ln -s "$PWD/mychecklist-skill/skills/mychecklist" ~/.claude/skills/mychecklist
```

Restart Claude Code, then type `/` and you should see `/mychecklist`.

## Usage

Paste the ticket and ask:

```
/mychecklist
Style to match: Font Poppins. Background #F6F1F1, white cards with 16px corners.
Brand gradient #E1059B → #4E016A on the dashboard highlight card, the logo and sign-in.
Scope: Users list, profile, create/edit. Data and permissions stay the same.

green every line if done
```

It diffs the branch against its base with `git diff <base>...HEAD`, then greens, ambers or reds every line.

## The mirror of /myscope

| | asks |
|---|---|
| **`/myscope`** | did I change anything I **shouldn't** have? — *scope creep* |
| **`/mychecklist`** | did I build everything I **should** have? — *scope gaps* |

A branch can pass one and fail the other. `/myscope` reads the diff and measures it against the task; `/mychecklist` reads the ticket and measures the code against it. Run both once the work is built, before `/mypr`.

## What it does to a ticket

Four bullets become thirteen checkable lines. A combined requirement — *"gradient on the highlight card, the logo and sign-in"* — splits three ways, because a line covering three things cannot be marked once. Instructions become outcomes: *"use the real data"* becomes *"real data is used everywhere, with no placeholders left."* Misnested items get lifted back out of Scope, where notes and done-when criteria routinely end up in the original.

## You get

- **A checklist, shown before verification**, so you can correct a misreading before the evidence hunt starts.
- **A verdict per line** — ✅ Done, ⚠️ Partial, ❌ Not done — each citing `file:line` and what it shows. A ✅ without a citation is an assertion, not a verdict.
- **Honest ⚠️ for anything code can't settle** — "works at phone width", "roles still work" — with what manual test would confirm it. A guess recorded as ✅ is worse than an amber.
- **A `(pre-existing)` note** when a requirement was already true before the branch. It still satisfies the ticket, but the note tells you the ticket was stale or someone misread the work.
- **A count and a verdict.** **Complete** only when every line is ✅. One ⚠️ makes it Incomplete — "mostly done" is the state this skill exists to make visible, not to round up.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
