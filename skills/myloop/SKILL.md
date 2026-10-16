---
name: myloop
description: Run the PR review pipeline end to end — /mycodereview, /myfindings, /myverdict, /myfix, /myscope — printing each skill's full result before the next one starts. Takes the pull request as its argument (/myloop 1234) and resolves the ticket from context — the session first, where /mytask, /myrepro, /mychecklist or the plan document usually established it already, then the PR body or linked issue, and only asks when neither turns up anything; a ticket may also be named explicitly (/myloop 1234 ENG-1529). Use when the user runs /myloop, or asks to "run the review chain", "review this PR and fix it", "do the whole review flow", or "chain the review skills". Replaces running those five by hand after every review. Each step is the existing skill run exactly as written; this skill contributes no review, triage, or fix logic of its own. Control flow only: /mycodereview always continues to /myfindings so the triage states plainly whether there are findings; the chain stops after /myfindings when nothing is P0, P1 or P2; when /myverdict corrects a severity in either direction the chain returns to /myfindings for a second verification and then back through /myverdict, bounded so the same finding's severity can only be corrected once; and /myscope audits the fixes against /myverdict's address-now list rather than the original ticket, so already-committed files do not bury the actual changes. A finding about the pull request description is not a code change — /myfix states what to change as a before and after for the user to apply. Ends with one block: the /myscope result, the git commands, and the pull request description correction when one was flagged. Never commits or pushes. Do NOT use to run a single skill, or for the plan-side loop (use /myreviewer then /myplanfix).
---

# 🔁 /myloop — Chained Review Pipeline

Run the five review skills in order, each printing its full result before the next begins. Everything `/myloop` does is sequencing — the thinking stays in the skills.

```
[ticket]       → objective + acceptance criteria, pinned
      ↓          the yardstick for every step below
/mycodereview  → review printed            (always continues)
      ↓
/myfindings    → P0–P3 triage printed
      ↓ 0 P0–P2?  STOP
/myverdict     → rulings printed
      ↓ severity corrected (either direction)?
      │     → /myfindings  (2nd verification, updated counts)
      │          ↓ 0 P0–P2 now?  STOP
      │     → /myverdict   (then continue — bounded, see below)
      ↓
/myfix         → edits, or a before/after correction for non-code findings
      ↓
/myscope       → audits the fixes against the address-now list
      ↓
FINAL BLOCK    → /myscope result  →  git commands  →  PR description before/after
```

## Rules

- **Establish the ticket before anything runs, and keep it in front of every step.** This is the chain's biggest failure mode. `/mycodereview` takes a pull request and nothing else — it has no notion of the ticket — so if the chain does not supply one, `/myverdict` has no scope axis and, in its own words, *"collapses into a code review"* (`skills/myverdict/SKILL.md:25`). `/myfix` then implements that drift and `/myscope` audits against it. Every step inherits whatever the first missing input caused.
- **The ticket is the yardstick, not the pull request.** Restate the objective and the acceptance criteria when handing off to `/myverdict`, `/myfix` and `/myscope`. A finding can be perfectly correct about the code and still have nothing to do with what this ticket set out to do.
- **A fix outside the ticket is only applied when the ticket cannot close without it.** That is the test — not "is it an improvement", not "is it nearby", not "while we are in here". `/myverdict` already defers out-of-scope findings; hold it to that and let the deferred ones become their own tickets. The one standing exception is `/myverdict`'s own: a security or data-loss risk is addressed now, with a note explaining why the PR grew.
- **Chain the skills, do not reimplement them.** Every step is the existing skill, invoked as written, with its own output format intact. `/myloop` adds no review criteria, no severity rules, no fix logic. If a step behaves wrongly, that is a bug in that skill — fix it there, never patch around it here.
- **Print each result in full before starting the next step.** No summarising, no batching, no "findings omitted for brevity". The point of the chain is that you can read every stage.
- **Announce each step** with a header line so the stream stays navigable: `── 1/5 · /mycodereview ──`.
- **`/mycodereview` always continues to `/myfindings`**, even with zero findings. The triage stating "no findings" is the confirmation worth having; a chain that dies silently at step 1 tells you nothing.
- **Hand each skill its inputs from the step before.** `/myfindings` gets the review's findings; `/myverdict` gets the task plus the P0–P2 set; `/myfix` gets the address-now rulings; `/myscope` gets the working-tree diff plus the address-now list. Never make a skill re-derive what the previous one already produced.
- **Severity may be corrected once per finding.** `/myverdict` correcting a label sends the chain back to `/myfindings`, then forward through `/myverdict` again. If `/myverdict` changes **the same finding's** severity a second time, stop cycling: carry that finding at its latest value, note the unsettled severity in the output, and continue to `/myfix`. A P2 and a P3 can otherwise trade places indefinitely.
- **Follow the order. No jumps.** Once past a stop condition, every remaining step runs — including when `/myverdict` rejected everything and `/myfix` has nothing to edit. A step with no work says so; it is not skipped.
- **Never commit, stage, or push.** `/myfix` edits the working tree. The git commands are printed at the end for the user to run.
- **A pull request description finding is not a code change.** `/myfix` does not rewrite it. It states what to change as a **before** and an **after**, and the user applies it.

## Steps

0. **Resolve the ticket.** The invocation is just the pull request — `/myloop 1234`. The ticket comes from context, in this order:

   1. **This session.** Usually it is already here: the ticket `/mytask` classified, the one `/myrepro` wrote steps for, the one `/mychecklist` checked, or the plan in `docs/plans/`. Use it.
   2. **The PR itself** — its body, title, or linked issue.
   3. **Ask.** Only when the first two turn up nothing.

   Pin the **objective** and the **acceptance criteria**, then print them at the top of the run **with where they came from** — "objective from ENG-1529, established earlier this session". If the chain anchors to the wrong ticket, that line is where you catch it, before four skills have built on it.

   Do not substitute the PR diff as the objective. "What the code does" and "what the ticket asked for" are the two things the chain exists to compare; collapsing them is what produces out-of-scope fixes.

   A ticket can also be named explicitly — `/myloop 1234 ENG-1529` — which skips the resolution order.

1. **`/mycodereview`** — review the PR. Print the result. Continue regardless of how many findings there are.

2. **`/myfindings`** — triage the review's findings into P0–P3. Print the result.
   **Stop condition:** if nothing is P0, P1 or P2, the chain ends here. Say so plainly — "no P0–P2 findings; nothing to verdict" — and go to the final block if a PR description correction was raised, otherwise end.

3. **`/myverdict`** — cross-verify the P0–P2 findings against the code and rule each one. **Hand it the step 0 ticket as the task.** Its scope axis is measured against that objective and those acceptance criteria, so a finding outside them is deferred rather than fixed. Print the result.

4. **Severity re-verification**, only when step 3 corrected a label in either direction:
   - Re-run **`/myfindings`** with the corrected severities. Print the updated triage.
   - **Stop condition:** if the corrected set is now 0 P0–P2, the chain ends here.
   - Otherwise re-run **`/myverdict`** on the surviving set, then continue. Apply the once-per-finding cap above — a finding whose severity has already been corrected keeps its current value and is reported as unsettled rather than sent round again.

5. **`/myfix`** — implement what `/myverdict` ruled *address now*, and nothing beyond it. If a fix starts pulling in work the ticket never asked for, stop at the boundary and note it as a follow-up rather than following it. Print its result, including its own commit message; the duplication with the final block is deliberate, so neither place sends you scrolling.

6. **`/myscope`** — audit the resulting working-tree diff. The yardstick is **`/myverdict`'s address-now list**, not the original ticket: the original work was already judged by `/mycodereview`, and measuring against the ticket drags already-committed files into the report and buries the fixes. The question here is narrower and more useful — *did `/myfix` change only what it was told to?* Keep the step 0 objective visible while auditing: an edit tracing to no acceptance criterion and to no address-now ruling is drift, whatever its merits.

7. **Final block** — the three things worth having in one place at the end.

## Final block

Print exactly this after `/myscope`:

````
── Final ──────────────────────────────────────────────

### 🔍 Scope audit
[/myscope's verdict, repeated here in full]

### 🎯 Traced to the ticket
| Fix applied | Traces to |
|---|---|
| [what changed] | [AC-n / the objective / security exception — why the ticket cannot close without it] |
*An edit that traces to nothing is drift. Say so here rather than letting it ship quietly.*

### ⌨️ Git commands
```bash
git add [the files /myfix touched]
git commit -m "[one line describing the fixes]"
git push
```

### 📝 PR description  *(only when a finding raised it)*
**Change:** [which part — title, summary, testing notes, linked ticket]

**Before**
> [the current text, verbatim]

**After**
> [the corrected text]

[Why: the finding that called for it]
````

Leave the PR description section out entirely when no finding raised it. When one did, include it **wherever the chain terminates** — including an early stop at step 2 or 4, where there is no scope audit and no diff to commit.

## Goal

Stop running five skills by hand after every review. One command walks the whole pipeline with the ticket pinned in front of it, prints every stage so nothing is hidden, keeps the fixes tied to what the ticket actually asked for, stops the moment there is genuinely nothing left to do, and ends with the audit, the git commands, and the PR description fix in one place instead of scattered up the scrollback.
