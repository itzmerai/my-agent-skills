---
name: myplanfix
description: Repair an implementation plan that /myreviewer rated Partially Aligned or Misaligned. Use when the user runs /myplanfix, or asks to "fix the plan", "align the plan to the ticket", "address the reviewer feedback on the plan", or "make the plan pass review". Re-reads the ticket line by line as a safety net and finds the misalignments itself rather than trusting the reviewer's list — fixing only what was named is what produces a fresh batch of findings on the next pass and keeps the review loop running. Rules on each reviewer finding before touching anything: a valid gap is fixed, a finding already covered by the plan is rejected with the section that covers it, a finding that contradicts the ticket is rejected with the ticket line, and a genuine ambiguity in the ticket is flagged for its author rather than guessed. Grounds every correction in evidence, in order of authority: the ticket governs scope, the PRD confirms intent where the project has a copy, and the codebase decides feasibility and the house conventions. A missing PRD is not a gap — the codebase structure and existing patterns stand in. Where the plan must choose between approaches, it reads the codebase, makes the call, and states the choice and its reason instead of leaving the decision open. Edits the plan document in place with targeted changes, preserves every section that is already correct, never renumbers implementation units, and ends by telling the user to run /myreviewer. Making no edits is a valid outcome when the findings do not hold up. Edits plan documents only — never writes production code, never runs state-modifying git, and never produces a review verdict or rating of its own (that is /myreviewer's job).
---

# 🧩 /myplanfix — Plan Repair

Fix a plan that came back **Partially Aligned** or **Misaligned**, so the next `/myreviewer` run comes back **Aligned**.

This is the plan-side counterpart to `/myfix`. `/myfix` implements review findings in code; this implements them in the plan — and like `/myfix`, it is not a second reviewer. It repairs and hands back.

## Why it re-reads the ticket

The loop everyone gets stuck in looks like this:

```
/myreviewer → "Partially Aligned, fix X"  →  fix X
/myreviewer → "Partially Aligned, fix Y"  →  fix Y
/myreviewer → "Partially Aligned, fix Z"  →  ...
```

It never converges because each pass patches only what the reviewer happened to name, and a fresh batch surfaces next time. So this skill walks the ticket line by line and finds the misalignments **itself** — a safety net under the reviewer's list, not a copy of it. Everything genuinely wrong gets fixed in one pass, which is what makes the next review come back clean.

That analysis is in service of fixing. It does **not** produce a verdict, a rating, or an alignment table — `/myreviewer` owns that.

## Rules

- **Fix the plan, nothing else.** Edits the plan document only. Never writes production code, never stages, commits, or pushes.
- **Repair, do not review.** No rating, no Aligned/Partially/Misaligned judgement of your own. Report what you changed and why.
- **The reviewer's findings are an input, not the work list.** Re-read the ticket yourself and fix everything that is actually wrong — missing, misread, unnecessary, or internally inconsistent — whether or not the review named it.
- **Never edit just to pass review.** Every change cites what justifies it: a ticket line, a PRD section, or a codebase pattern. A change that exists only to satisfy a reviewer is the failure mode this skill is built to avoid.
- **Preserve what is already correct.** Targeted edits. A section that holds up is left untouched, even when it sits next to one that did not.
- **Evidence has an order of authority.** The **ticket governs scope** — if the ticket does not ask for it, the plan should not plan it. The **PRD confirms intent** where the project keeps a copy. The **codebase decides feasibility** and which conventions apply. If the ticket and PRD contradict each other, surface it rather than quietly siding with one.
- **A missing PRD is not a gap.** Many projects do not keep one. Fall back to the ticket plus the codebase's existing structure, patterns and conventions, and say which you used.
- **Decide, do not punt.** When the plan has to choose — extend a table or add one, reuse a service or write a new one — read the codebase, pick the option that matches what is already there, and **state the choice and the reason**. Leaving the decision open is how a plan passes review and still stalls at build time.
- **Never renumber implementation units.** `ce-work` tracks tasks against `U1`, `U2`… Extend the unit a new requirement belongs to; if none fits, append a new U-ID at the end. Renumbering mid-flight breaks the task list.
- **A rejected finding carries its reason**, with the plan section or ticket line that disproves it. Never drop one silently.
- **Making no edits is a valid outcome.** If the plan is right and the findings do not hold up, say so with the evidence and change nothing. Editing a correct plan to appease a review makes it worse.
- **Print the report in the chat/terminal** using the exact output format below.

## Steps

1. **Gather the inputs.** The plan file (`ce-plan` always writes one — find it in `docs/plans/`), the ticket, and `/myreviewer`'s findings. Then look for a PRD: `docs/prd/`, `PRD.md`, `docs/brainstorms/*-requirements.md`, `docs/requirements*`. State what you found, including "no PRD in this project". If the ticket or the plan is missing, ask — there is nothing to align against.

2. **Re-read the ticket line by line.** Extract every requirement and acceptance criterion as its own item, splitting combined ones. This is your yardstick, independent of the review.

3. **Compare the plan against that list** and classify each difference:
   - **Missing** — a requirement the plan never addresses
   - **Misread** — addressed, but the plan solves a different problem than the ticket describes
   - **Unnecessary** — planned work the ticket never asked for
   - **Inconsistent** — the plan contradicts itself, or one unit undoes another

4. **Rule on each reviewer finding** before editing anything:

   | Ruling | When | What happens |
   |---|---|---|
   | ✅ **Valid** | a real gap or error | fixed |
   | 🔁 **Already addressed** | the plan covers it | rejected — cite the section |
   | ⛔ **Contradicts the ticket** | the fix would break a requirement | rejected — cite the ticket line |
   | ❓ **Ambiguous** | the ticket genuinely does not say | flagged for the author, plan untouched |
   | ⏭️ **Out of scope** | valid against the PRD, not this ticket | rejected — belongs in its own ticket |

5. **Ground the corrections in the codebase.** Before writing a fix, check how the project already does this — the modules, patterns, naming and conventions in play — so the plan proposes the house way rather than a generic one. This is also where feasibility gets checked: a plan step that cannot work against the actual structure is a defect even when no reviewer flagged it.

6. **Apply the edits**, in place, targeted. Keep the plan's existing structure and U-IDs.

7. **Report and hand back.** List what changed, what you rejected and why, what you decided and on what evidence, and anything that needs the ticket's author. End by telling the user to run `/myreviewer`.

## Output format

Print exactly this structure:

````
### 📄 Inputs
- Plan: `docs/plans/...md`
- Ticket: [id / one line]
- PRD: `docs/prd/...md` — or: none in this project, using the ticket and codebase
- Review findings: N

### ⚖️ Reviewer findings
| # | Finding | Ruling | Why |
|---|---|---|---|
| 1 | [short] | ✅ Valid | [what the ticket or codebase shows] |
| 2 | [short] | 🔁 Already addressed | covered by [plan section] |
| 3 | [short] | ⛔ Contradicts the ticket | ticket says "[line]" |
| 4 | [short] | ❓ Ambiguous | [what the ticket fails to say] |

### 🕵️ Found on re-read — not raised in review
- **[Missing / Misread / Unnecessary / Inconsistent]** — [what, and the ticket line it comes from]

### ✍️ Edits applied
- **[Plan section / U-ID]** — [what changed] — *because* [ticket line / PRD section / `path/file.ext` pattern]

### 🧭 Decisions made
- **[the choice]** — picked [option] over [option] because [what the codebase already does, with the path]

### ❓ For the ticket's author
- [the ambiguity, phrased as a question — left unfixed on purpose]

### ➡️ Next
Run `/myreviewer` to confirm the plan is now Aligned.

[N findings · N valid · N rejected · N ambiguous · N found on re-read · N edits applied]
````

If nothing was edited, say so plainly in **Edits applied** — "no changes; the findings did not hold up" — and let the rulings carry the explanation. The plan file is left exactly as it was.

## Goal

Get a plan to Aligned once, with changes that are right rather than changes that are agreeable. Run it when `/myreviewer` comes back short: it repairs what is genuinely wrong — including what the review missed — refuses what does not hold up, decides what the plan left open, and hands back a plan that is correct against the ticket, consistent with the codebase, and ready for `ce-work`.
