---
name: mychecklist
description: Turn a ticket, spec, or requirements list into a line-by-line verification checklist, then check the current branch against its base and mark every line Done, Partial, or Not done with file evidence. Use when the user runs /mychecklist, pastes a ticket and asks "was this followed", "is this done", "check this branch against the ticket", "verify the PR matches the spec", "green every line if done", or wants a checkbox per requirement — even if they never say "checklist". Splits combined requirements into one checkable item per line ("gradient on the highlight card, the logo and sign-in" becomes three lines), phrases each as a statement that is true or false, turns instructions into outcomes, regroups misnested items into the ticket's own sections, and keeps exact values such as hex codes, sizes, route names and role names verbatim. Verifies each line by reading the actual code rather than file names, cites file and line for every verdict, marks anything unconfirmable from code alone as Partial with what needs a manual test, and notes when a requirement was already satisfied before the branch. Ends with a count and a Complete or Incomplete verdict naming the blocking items. Review only — never edits, stages, or commits code. This is the mirror of /myscope: /myscope asks whether anything was changed that should not have been, this asks whether everything was built that should have been. Do NOT use for judging code quality (use /mycodereview), for triaging review findings (use /myverdict), or for reviewing a plan before it is built (use /myreviewer).
---

# 📋 /mychecklist — Ticket Completeness Check

Verify a branch against its ticket, one requirement per line. Every line is a statement that is either true or false of the code, and every verdict carries the file that proves it.

This is the mirror of `/myscope`. `/myscope` asks *did I change anything I should not have* — scope creep. This asks *did I build everything I should have* — scope gaps. A branch can pass one and fail the other, which is why both run.

## Rules

- **One requirement per line.** Split combined items. "Gradient used on the highlight card, the logo and sign-in" is three lines, not one. "Users: list, profile, create/edit" is three lines. A line covering two things cannot be marked once.
- **Every line is true or false.** "Font is Poppins", not "Font: Poppins". If you cannot phrase it as a claim about the code, it is not a checklist line — it is context.
- **Turn instructions into outcomes.** "Use the real data" becomes "Real data is used everywhere, with no placeholders left." You verify end states, not intentions.
- **Read the code, not the file tree.** A file named `theme.css` existing proves nothing about the font. The declaration inside it does. Open what you cite.
- **Cite evidence on every line.** `file:line` and what it shows. A ✅ with no citation is an assertion, not a verdict.
- **Never invent requirements.** Only what the ticket says. If something obviously needed is missing from the ticket, note it in one line at the end — do not quietly add it to the checklist and then fail the branch for it.
- **Keep exact values verbatim**, in backticks: hex codes, sizes, route names, role names, config keys. `#F6F1F1` and "light grey" are not the same requirement.
- **Do not flag changes the ticket never asked for.** That is `/myscope`'s job. A branch that did extra work is not Incomplete here.
- **Review only.** Never edit, stage, or commit. Read-only git — `git status`, `git diff`, `git log`, `git show`.
- **Print the checklist in the chat/terminal** using the exact output format below.

## Steps

1. **Get the ticket.** The user pastes it, or names where it lives. If there is no ticket there is nothing to check against — ask rather than inferring requirements from the diff, which would just mark everything done.

2. **Find the base branch.** Use the base the user names. Otherwise try `develop` first — it is this suite's default, matching `/mytask`'s branch recommendation — then `origin/HEAD`, then `main`, then `master`. **Say which one you used.** Always diff with three dots so only this branch's own work shows:

   ```bash
   git diff <base>...HEAD --stat
   git diff <base>...HEAD
   ```

3. **Turn the ticket into a checklist.** Apply the splitting and phrasing rules above, then **group the lines into sections that match the ticket** — commonly Style, Scope, Implementation rules, Done when. Fix misnesting while you are there: notes and done-when items routinely end up buried under Scope in the original. Show this checklist before verifying it, so the user can correct a misreading before the evidence hunt.

4. **Verify every line against the code.**
   - ✅ **Done** — fully implemented. Cite the file and line.
   - ⚠️ **Partial** — started but incomplete. Say exactly what is missing, not just that something is.
   - ❌ **Not done** — no implementation found. Say where you looked, so the user can tell a genuine gap from a search that missed it.

   Three cases need care:

   - **"Nothing changed" lines** (permissions, actions, data stay the same) are verified by scanning the diff *for* changes: policies, gates, auth methods, action handlers, form fields that persist, queries, models, migrations, routes, middleware. Any hit makes the line ⚠️ or ❌ with the offending lines quoted. Finding nothing is the pass.
   - **Lines code cannot settle** — "works at phone width", "roles still work" — are ⚠️, never ✅. State what the code suggests and what manual test would confirm it. A guess recorded as ✅ is worse than an honest ⚠️.
   - **Already true before the branch** — the requirement holds, but this branch did not make it hold. Mark ✅ **(pre-existing)**. It satisfies the ticket, and the note matters: either the ticket was stale or someone misread what needed doing.

5. **Report** using the output format, ending with the count and verdict.

## Output format

Print exactly this structure:

````
Base branch: `develop` (diffed with `git diff develop...HEAD`)

## [Section from the ticket]
- ✅ [requirement as a true/false statement] — `path/file.ext:42` [what it shows]
- ⚠️ [requirement] — [what exists] but [exactly what is missing] (`path/file.ext:21`)
- ❌ [requirement] — [where you looked and found nothing]
- ✅ [requirement] **(pre-existing)** — `path/file.ext:7`, unchanged in this branch

## [Next section]
- ...

## Summary
✅ N · ⚠️ N · ❌ N

## Verdict: Complete / Incomplete
Blocking: [the ⚠️ and ❌ lines, named — omit this line when Complete]

[Only if the ticket itself looked incomplete:]
Not in the ticket: [the one thing that seemed required but was never asked for]

No code was modified. This was a review only.
````

**Complete** only when every line is ✅. One ⚠️ makes it Incomplete — "mostly done" is the state this skill exists to make visible, not to round up.

## Example — the split

The value is mostly in step 3, so here is what it does to a real paste:

**Pasted ticket**

```
Style to match (taken from the mobile app):
* Font: Poppins.
* Background `#F6F1F1`, white cards with 16px corners and a `#F1F5F9` border.
* Brand gradient `#E1059B → #4E016A`, used for the dashboard's highlight card,
  the logo and sign-in.
* Scope (every page in the redesign):
   * Users: list, profile, create/edit.
   * Data, permissions and actions stay the same. Visual and UX change only.
   * The redesign uses placeholder data instead of real photos. Use the real data.
   * Existing admin actions and roles still work (admin vs super_admin gates).
```

**Becomes**

```
## Style
- [ ] Font is Poppins
- [ ] Background is `#F6F1F1`
- [ ] Cards are white with 16px corners and a `#F1F5F9` border
- [ ] Brand gradient `#E1059B → #4E016A` is used on the dashboard highlight card
- [ ] Brand gradient is used on the logo
- [ ] Brand gradient is used on the sign-in page

## Scope
- [ ] Users: list
- [ ] Users: profile
- [ ] Users: create/edit

## Implementation rules
- [ ] Data, permissions, and actions are unchanged; visual and UX change only
- [ ] Real data is used everywhere, with no placeholders left in place of real photos

## Done when
- [ ] Existing admin actions and roles still work (admin vs super_admin gates)
```

Four bullets became thirteen lines. The gradient bullet split three ways, Users split three ways, and the last three items were lifted out of Scope — they were never scope, they were rules and a done-when, misnested in the original.

## Goal

Make "it's done" something you can point at line by line. Run it alongside `/myscope` once the work is built: `/myscope` catches what crept in, `/mychecklist` catches what never got built — and between them, the branch either matches its ticket or you know exactly which lines do not.
