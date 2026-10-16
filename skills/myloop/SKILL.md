---
name: myloop
description: Run the PR review pipeline end to end — /mycodereview, /myfindings, /myverdict, /myfix, /myscope — printing each skill's full result before the next one starts. Use when the user runs /myloop, or asks to "run the review chain", "review this PR and fix it", "do the whole review flow", or "chain the review skills". Replaces running those five by hand after every review. Each step is the existing skill run exactly as written; this skill contributes no review, triage, or fix logic of its own. Control flow only: /mycodereview always continues to /myfindings so the triage states plainly whether there are findings; the chain stops after /myfindings when nothing is P0, P1 or P2; when /myverdict corrects a severity in either direction the chain returns to /myfindings for a second verification and then back through /myverdict, bounded so the same finding's severity can only be corrected once; and /myscope audits the fixes against /myverdict's address-now list rather than the original ticket, so already-committed files do not bury the actual changes. A finding about the pull request description is not a code change — /myfix states what to change as a before and after for the user to apply. Ends with one block: the /myscope result, the git commands, and the pull request description correction when one was flagged. Never commits or pushes. Do NOT use to run a single skill, or for the plan-side loop (use /myreviewer then /myplanfix).
---

# 🔁 /myloop — Chained Review Pipeline

Run the five review skills in order, each printing its full result before the next begins. Everything `/myloop` does is sequencing — the thinking stays in the skills.

```
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

1. **`/mycodereview`** — review the PR. Print the result. Continue regardless of how many findings there are.

2. **`/myfindings`** — triage the review's findings into P0–P3. Print the result.
   **Stop condition:** if nothing is P0, P1 or P2, the chain ends here. Say so plainly — "no P0–P2 findings; nothing to verdict" — and go to the final block if a PR description correction was raised, otherwise end.

3. **`/myverdict`** — cross-verify the P0–P2 findings against the code and rule each one. Print the result.

4. **Severity re-verification**, only when step 3 corrected a label in either direction:
   - Re-run **`/myfindings`** with the corrected severities. Print the updated triage.
   - **Stop condition:** if the corrected set is now 0 P0–P2, the chain ends here.
   - Otherwise re-run **`/myverdict`** on the surviving set, then continue. Apply the once-per-finding cap above — a finding whose severity has already been corrected keeps its current value and is reported as unsettled rather than sent round again.

5. **`/myfix`** — implement what `/myverdict` ruled *address now*. Print its result, including its own commit message; the duplication with the final block is deliberate, so neither place sends you scrolling.

6. **`/myscope`** — audit the resulting working-tree diff. The yardstick is **`/myverdict`'s address-now list**, not the original ticket: the original work was already judged by `/mycodereview`, and measuring against the ticket drags already-committed files into the report and buries the fixes. The question here is narrower and more useful — *did `/myfix` change only what it was told to?*

7. **Final block** — the three things worth having in one place at the end.

## Final block

Print exactly this after `/myscope`:

````
── Final ──────────────────────────────────────────────

### 🔍 Scope audit
[/myscope's verdict, repeated here in full]

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

Stop running five skills by hand after every review. One command walks the whole pipeline, prints every stage so nothing is hidden, stops the moment there is genuinely nothing left to do, and ends with the audit, the git commands, and the PR description fix in one place instead of scattered up the scrollback.
