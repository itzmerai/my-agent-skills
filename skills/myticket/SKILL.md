---
name: myticket
description: Archive a ticket to a durable local file and turn every mention of another ticket into a path to the archived copy. Use when the user runs /myticket, pastes a ticket and asks to "save this ticket", "file this", "archive the ticket", "keep a copy of this ticket", or asks "what does ENG-1234 say" / "do we have that ticket" / "which tickets does this one depend on". Also runs automatically as the first step of /mytask and /myprobe, so a ticket entering the workflow is kept without being asked for. Exists because a ticket pasted into a session dies with it, and the next session — the /myprobe AFTER arm days later, /mychecklist against the finished branch — starts blind; and because a reference to another ticket, whether bare text or a tracker URL, is a dead end that nothing in a session can open. Writes one file per ticket, named by the ticket ID, with the body kept verbatim and the reporter's prose never rewritten: a reference that has an archived copy gains an absolute path beside it, and a reference that does not is left exactly as written. Scans both directions on every run, so a ticket archived later backfills the mentions already sitting in older files. Matches only the ID shapes the project actually uses, so UTF-8, SHA-256 and RFC-7231 are not mistaken for tickets, and never touches an ID inside a fenced code block. Resolves its folder from CLAUDE.md, then from an existing folder under .claude, and only asks when neither exists. Never fetches from Linear, Jira or GitHub — it archives what it was handed. Do NOT use to analyse or classify a ticket (use /mytask), to write tests from one (/mytdd, /myprobe), or to check a branch against one (/mychecklist).
---

# 🗂️ /myticket — Local Ticket Archive

A ticket pasted into a session lasts as long as the session. A ticket that mentions another ticket points at something nothing here can open. This fixes both: one durable file per ticket, and every mention resolved to the archived copy's path.

```
ticket  →  resolve folder   CLAUDE.md → an existing folder under .claude/ → ask once
        →  resolve keys     which ID shapes this project actually uses
        →  write            <folder>/<TICKET>.md — body verbatim, header added
        →  forward pass     this ticket's mentions   → archived paths
        →  backward pass    older files' mentions of THIS ticket → resolved now
        →  report           the path, what resolved, what is still missing
```

The backward pass is what makes export order stop mattering. File ENG-2451 on Monday with a bare mention of ENG-2449; file ENG-2449 on Thursday and Monday's file is corrected then.

## Rules

- **Archive what you were handed. Never fetch.** No Linear, Jira or `gh` call to pull a ticket you do not have. If the user names a ticket that is not in the archive and not in the session, say so and stop — inventing a ticket body from a plausible-sounding ID is the one failure this skill cannot survive.
- **The body is verbatim.** What the reporter wrote is the record. Do not summarise it, reformat it, fix its typos, or reorder it. The only addition inside the body is a resolved path beside a mention.
- **Annotate, never substitute.** A tracker URL stays; the local path is added after it. The two serve different readers — the URL is what *you* open in a browser, the path is what the *session* opens — and deleting either to serve the other loses information.
- **Match the project's keys, not a generic pattern.** A bare `[A-Z]+-\d+` matches `UTF-8`, `SHA-256`, `ISO-8601`, `RFC-7231`, `AES-256` and `HTTP-2`. Resolve the key set first and match only those prefixes.
- **Never annotate inside a fenced code block.** Ticket bodies carry log lines, stack traces and commit messages that contain ticket IDs. Annotating inside a fence corrupts the snippet, and the snippet is usually the evidence.
- **Never resolve a ticket's mention of itself.** Every file names its own ID in its heading.
- **A mention with no archived copy is left untouched in the body.** No placeholder, no invented path, no "not found" marker inline. It is listed once in the header table, because that table is the list of what to file next.
- **Paths are absolute.** A relative path breaks the moment a skill runs from a different sub-project, and one archive at the workspace root is meant to serve them all.
- **Both passes run every time**, including on a re-export that changes nothing. They are a grep over small Markdown files; skipping them to save nothing is how an archive goes stale.
- **The archive lives outside git.** Warn if the resolved folder sits inside a repository — an archive that gets committed is a ticket leak, and ticket text is the user's employer's, not the repo's.
- **Ask once per project, never again.** The folder and the key set are recorded in `CLAUDE.md` the first time and read from there afterwards.
- **When another skill calls this one, the folder question still comes first.** `/mytask` and `/myprobe` run this before they print anything, so asking here is an interruption of nothing. Never defer the question, never pick a folder silently to avoid asking, and never hand the question back to the calling skill to ask later — later means after a full report the user then has to re-read.
- **Never commit, stage or push.** Read-only on git except inspection.
- **Always print the absolute path of the file you wrote**, ready to paste — so it can be opened or handed to another skill without being asked for.

## Steps

1. **Resolve the ticket and its ID.** Take it from the session first, then from the argument — pasted text, a Linear or Jira ticket, a GitHub issue. The ID is the filename, verbatim and hyphen intact: `ENG-2451` → `ENG-2451.md`. A ticket with no ID is still worth archiving: derive a slug from its title (`login-crash-on-empty-email.md`) and say that is what you did. If the argument is only an ID and the archive already holds it, this is a **read** — print the file and stop.

2. **Resolve the archive folder**, in this order — ask only as a last resort:
   1. A folder recorded in the project's `CLAUDE.md` → use it.
   2. An existing folder under `.claude/` already holding ticket files (commonly `.claude/tickets/`) → use it and say which.
   3. Neither → ask, proposing `.claude/tickets/`, then create it and record the choice in `CLAUDE.md` together with the key set from step 3.

   Prefer the `.claude/` of the **workspace root** over a single repo's, so one archive serves every sub-project — cross-repo mentions (*"ENG-2449 is the app side of this"*) are the common case, and an archive split per repo cannot resolve them. Confirm it sits outside any git repository; if it does not, say so before writing.

3. **Resolve the key set** — the ID shapes this project actually uses:
   1. Recorded in `CLAUDE.md` → use it.
   2. The prefix of the ticket being archived, plus every distinct prefix already present in the archive's filenames.
   3. A GitHub-tracker project uses `#123`; the same logic applies, and the filename becomes `issue-123.md`.

   Record the set, so `ENG` and `OPS` are recognised and `UTF`, `SHA` and `RFC` never are. A prefix is added to the set only by a ticket actually being archived under it — never by appearing in prose.

4. **Write the file.** Header block, then a divider, then the body:

   ```markdown
   # ENG-2451 — <title, or the first line if there is none>

   > **Captured** <YYYY-MM-DD> · **Source** <pasted in session / Linear / GitHub issue>

   | Mentions | Archived at |
   |---|---|
   | ENG-2449 | `/abs/path/.claude/tickets/ENG-2449.md` |
   | ENG-2452 | — not archived |

   ---

   <body, verbatim>
   ```

   The `Mentions` table is omitted entirely when the ticket mentions nothing. When `/mytask` invoked this skill, it adds one line under `Captured` — type, status, branch — and nothing more: a full analysis ages, can be wrong, and this file is the ticket of record, not a report.

   **If the file already exists:** compare bodies. Identical → say "already archived, unchanged", run both passes, change nothing else. Different → overwrite the body, keep the original `Captured` date, add `**Updated** <date>`, and print how many lines differ. A ticket rewritten underneath you is worth seeing, not silently absorbing.

5. **Forward pass.** Scan the body for mentions matching the key set, skipping fenced code blocks and the ticket's own ID. For each one the archive holds, add the path beside it — annotate the **first** occurrence only, so a ticket that names a dependency eight times does not become unreadable:

   | In the ticket | Becomes |
   |---|---|
   | `ENG-2449` | ``ENG-2449 (`/abs/…/ENG-2449.md`)`` |
   | `[ENG-2449](https://linear.app/…)` | ``[ENG-2449](https://linear.app/…) → `/abs/…/ENG-2449.md`` |
   | `ENG-2452`, not archived | `ENG-2452` — untouched |

   Then fill the header table with every mention found, resolved or not.

6. **Backward pass.** Grep the rest of the archive for mentions of **this** ticket. Where a file names it and has no path beside it, resolve it now under the same rules, and update that file's header table. Report each file touched.

7. **Report.** Print the path, what resolved, and what did not — the unresolved list is the next thing worth filing.

## Output format

Print exactly this structure:

````
### 🗂️ ENG-2451 archived
- File: `/abs/path/.claude/tickets/ENG-2451.md`
- Status: [new / unchanged / updated — n lines differ]
- Archive: `/abs/path/.claude/tickets/` · keys `ENG`, `OPS` [· recorded in CLAUDE.md]

### 🔗 References resolved
| Mention | Result |
|---|---|
| ENG-2449 | ✅ `/abs/path/.claude/tickets/ENG-2449.md` |
| ENG-2452 | — not archived, left as written |

### ↩️ Backfilled
| File | Mention resolved |
|---|---|
| `ENG-2449.md` | ENG-2451 |

[None — no other file mentions this ticket.]

### 📂 Open it
```bash
cat "/abs/path/.claude/tickets/ENG-2451.md"
```
````

Nothing committed. Nothing fetched.

## Goal

Make a ticket outlive the session it was pasted into, and make "see ENG-2449" something the next session can actually follow — so the skill running three days later reads the ticket and its dependencies from disk instead of asking for them again, and so the body it reads is still the one the reporter wrote.
