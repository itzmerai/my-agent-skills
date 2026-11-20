# /myticket

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myticket` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

A ticket pasted into a session lasts as long as the session. A ticket that mentions another ticket points at something nothing in that session can open. This fixes both: one durable file per ticket, and every mention resolved to the archived copy's path.

> **Outside git by design.** The archive lives under the workspace `.claude/`, not inside a repository. Ticket text belongs to whoever wrote it, not to the repo — an archive that gets committed is a leak.

## Install (Claude Code)

```bash
git clone --branch skill/myticket https://github.com/itzmerai/my-agent-skills.git myticket-skill
ln -s "$PWD/myticket-skill/skills/myticket" ~/.claude/skills/myticket
```

Restart Claude Code, then type `/` and you should see `/myticket`.

## The cycle

```
ticket  →  resolve folder   CLAUDE.md → an existing folder under .claude/ → ask once
        →  resolve keys     which ID shapes this project actually uses
        →  write            <folder>/<TICKET>.md — body verbatim, header added
        →  forward pass     this ticket's mentions   → archived paths
        →  backward pass    older files' mentions of THIS ticket → resolved now
        →  report           the path, what resolved, what is still missing
```

## The two problems it solves

**A ticket outlives nothing.** It is pasted into chat, used once, and gone. The session that runs the `/myprobe` **AFTER** arm three days later, or `/mychecklist` against the finished branch, starts blind and asks for it again. Worse, what it gets back may be a re-paste that is no longer what the first session read.

**A reference to another ticket is a dead end.** *"Blocked by ENG-2449."* *"See ENG-2451 for the API side."* Even a full `linear.app` URL is a string — nothing in a session can open it. So the context the reporter thought they were providing never arrives.

## The backward pass

The forward pass is obvious: resolve this ticket's mentions against what the archive already holds. The backward pass is the one that matters.

On every run, the rest of the archive is grepped for mentions of **this** ticket, and any that are still bare get resolved now.

```
Monday    file ENG-2451  →  mentions ENG-2449, not archived yet  →  left as written
Thursday  file ENG-2449  →  forward:  nothing new
                            backward: ENG-2451.md's mention is filled in
```

Without it, the archive only ever resolves backwards in time, and the order you happened to file things in decides how useful it is.

## What it will not do

**It does not fetch.** No Linear, Jira or `gh` call to pull a ticket it was not handed. A ticket named but not present is reported missing, never reconstructed — a plausible body invented from a plausible ID is the one failure this skill could not survive.

**It does not rewrite the body.** No summarising, no reformatting, no fixing the reporter's typos. The only thing added inside the body is a path beside a mention.

**It does not substitute, it annotates.** A tracker URL stays and the local path is added after it. The URL is what *you* open in a browser; the path is what the *session* opens. Dropping either loses a reader.

**It does not invent an absence.** A mention with no archived copy is left exactly as written — no placeholder, no "not found" marker. It appears once in the header table, which is the list of what to file next.

## Three guards

These are most of the skill.

**Match the project's keys, not a pattern.** A bare `[A-Z]+-\d+` happily matches `UTF-8`, `SHA-256`, `ISO-8601`, `RFC-7231`, `AES-256` and `HTTP-2`. The key set is resolved first — from `CLAUDE.md`, from the ticket being filed, from the filenames already present — and only those prefixes match. A prefix joins the set by a ticket actually being filed under it, never by appearing in prose.

**Never annotate inside a code fence.** Ticket bodies carry log lines, stack traces and commit messages containing ticket IDs. Annotating inside a fence corrupts the snippet, and the snippet is usually the evidence.

**Never resolve a ticket's mention of itself.** Every file names its own ID in its heading.

## The file

```markdown
# ENG-2451 — Restrict billing checkout to admins

> **Captured** 2026-11-20 · **Source** pasted in session
> **Classified** 🐞 Bug · Confirmed · `fix/ENG-2451-billing-role-gate`

| Mentions | Archived at |
|---|---|
| ENG-2449 | `/home/you/work/.claude/tickets/ENG-2449.md` |
| ENG-2452 | — not archived |

---

<the ticket body, verbatim>
```

Paths are **absolute**: a relative one breaks the moment a skill runs from a different sub-project, and one archive at the workspace root is meant to serve them all.

The `Classified` line appears only when `/mytask` filed the ticket, and it is one line — type, status, branch. A full analysis ages and can be wrong; this file is the ticket of record, not a report.

## Re-filing

| | |
|---|---|
| body unchanged | says so, re-runs both passes, changes nothing else |
| body differs | overwrites, keeps the original `Captured`, adds `Updated`, prints how many lines differ |

A ticket rewritten underneath you is worth seeing, not silently absorbing.

## Storage

```
<workspace>/.claude/tickets/ENG-2449.md
<workspace>/.claude/tickets/ENG-2451.md
```

Resolved with the same ladder `/mytdd` and `/myprobe` use for their staging folder — `CLAUDE.md`, then an existing folder under `.claude/`, then asks once and records the choice. One `.claude/`, sibling folders, one record.

The workspace root is preferred over a single repo's, because cross-repo mentions are the common case: *"ENG-2449 is the app side of this."* An archive split per repository cannot resolve them.

## Pairs with

`/mytask` and `/myprobe` file tickets through it automatically, so a ticket entering the workflow is kept without anyone asking. `/myrepro`, `/mytdd` and `/mychecklist` read from it — a linked ticket routinely carries the role names, accounts or requirements the ticket in hand only alludes to.

It also runs perfectly well on its own: hand it a ticket to file, or an ID to read back.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
