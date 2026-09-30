---
name: myrepro
description: Turn a ticket into concrete before-and-after verification steps. Use when the user runs /myrepro, whenever the user shares a ticket or issue and wants to know how to reproduce, test, or verify it, or asks to "repro this", "how do I test this", "write test steps", or "verify before and after". First classifies the ticket as a bug or a feature from its content. For a bug: writes exact reproduction steps (preconditions, environment, inputs, actions) that trigger the defect on the current code, states the observed broken behavior versus the expected behavior, then writes verification steps confirming the fix resolves it and that nearby behavior still works. For a feature: writes steps showing the baseline behavior before the change (feature absent or old behavior), then acceptance steps demonstrating the new behavior after implementation, mapped to each acceptance criterion in the ticket, including edge cases and negative paths. Where possible, runs the repro against the code to confirm the before state actually reproduces, and suggests an automated test that captures it. Every step states what to expect BEFORE (on today's code) and what to expect AFTER (once the work is done). If the ticket is ambiguous or missing details needed to reproduce, lists the gaps instead of guessing. Do NOT use for auditing file changes after work (use /myscope) or for triaging PR review findings (use /myverdict).
---

# 🧾 /myrepro — Before/After Verification Plan

Turn a ticket into steps someone else could run. Every step is written twice over: what it does **today**, and what it must do **once the work is done**. That pairing is the whole point — a step with only one half proves nothing, because you cannot tell a fix from a coincidence without knowing what the broken state looked like.

## Rules

- **Every step carries both expectations.** `Expect BEFORE` (current code) and `Expect AFTER` (work complete). A step missing either half is incomplete — fill it in or drop the step.
- **Classify first.** Bug and feature produce differently shaped plans. Decide which, and say why.
- **Ground every step in the real code.** Name actual routes, files, functions, commands, and fixture data — not "navigate to the relevant page". Read the code to find them.
- **Confirm the BEFORE state by running it where you can.** Report honestly which of these happened: *confirmed* (reproduced it), *not reproducible* (the code does not behave as the ticket claims — say so loudly, it may invalidate the ticket), or *not runnable here* (and why — needs a device, a production dataset, a third-party account).
- **Never guess a missing precondition.** If the ticket does not say which role, which environment, or which data, list it as a gap and ask. A plan built on invented setup is worse than no plan.
- **Do not implement anything.** No fixes, no refactors, no state-modifying git. Reading code and running read-only commands or existing tests is fine.
- **Write for someone else's hands.** Exact inputs, exact clicks or requests, exact expected strings and status codes.
- **Print the plan in the chat/terminal** using the exact output format below.

## Steps

1. **Classify the ticket** as **Bug** or **Feature** from its content. If `/mytask` already classified it, reuse that verdict and say so rather than re-litigating it.

2. **Extract what must be proven.** For a bug, the defect statement — what is broken, where, under what conditions. For a feature, every acceptance criterion in the ticket, listed and numbered so steps can map to them.

3. **Locate the code path.** Find the routes, components, handlers, models, or jobs involved, so the steps reference real names. Note the entry point a tester would start from.

4. **Write the BEFORE state.**
   - **Bug** — preconditions, environment, account/role, data fixtures, then exact actions that trigger the defect, and the observed broken behavior versus what the ticket says should happen.
   - **Feature** — the baseline: what happens today, whether that is "the feature does not exist", "the old behavior does X", or "the button is absent".

5. **Try to run it.** Reproduce the bug or capture the baseline using the project's own tooling — dev server, CLI, an existing test, a targeted query. Record the outcome as *confirmed*, *not reproducible*, or *not runnable here*, with the command used.

6. **Write the AFTER state**, mapped one-to-one to the defect or to each acceptance criterion. Cover:
   - The happy path for each criterion
   - **Edge cases** — empty, zero, maximum, boundary, duplicate, concurrent, offline
   - **Negative paths** — invalid input, missing permission, expired session, wrong state — and what the user should see instead of a crash

7. **Add regression checks.** Name the nearby behavior most likely to break: anything sharing the touched utility, route, component, table, or migration. Each still gets a before/after pair — for these, the two are usually *identical*, and that is exactly what makes them regression checks.

8. **Suggest an automated test.** Detect the project's framework and existing test layout, then name the file path to add, the case name, and the assertion that would have caught this. Prefer the cheapest level that captures it — unit over integration, integration over end-to-end.

9. **List the gaps.** Anything ambiguous, missing, or unverifiable, phrased as a question the ticket's author can answer.

## Output format

Print exactly this structure:

````
### 🧭 Ticket
- Type: **Bug** / **Feature** — [why, in one line]
- Under test: [the behavior being proven]
- Code path: `path/to/file.ts` → [route / function / component]

### ⚙️ Setup
- Environment: [local / staging / device / browser]
- Account & role: ...
- Data required: ...
- Start from: [exact URL, screen, or command]

### 🔬 Verification Matrix
| # | What you do | Expect BEFORE (today's code) | Expect AFTER (work done) | Covers |
|---|---|---|---|---|
| 1 | [exact action with exact input] | ❌ [the broken / absent behavior] | ✅ [the required behavior] | [defect / AC-1] |
| 2 | [edge case] | ... | ... | AC-2 |
| 3 | [negative path] | ... | ... | AC-3 |

### 🧪 BEFORE state — actually run?
- **Confirmed** / **Not reproducible** / **Not runnable here**
- Command used: `...`
- What happened: [the real output, quoted]

### 🧹 Regression Checks
| # | What you do | Expect BEFORE | Expect AFTER | Why it's at risk |
|---|---|---|---|---|
| R1 | ... | [same] | [same — unchanged] | [shares the touched utility / route] |

### 🤖 Suggested Automated Test
- File: `path/to/test.spec.ts` (framework detected: ...)
- Case: `it('...')`
- Asserts: [the one thing that would have caught this]

### ❓ Gaps — answer before testing
- [missing precondition, ambiguous criterion, unverifiable claim]
````

If the ticket is a **Feature**, the `Expect BEFORE` column is the baseline ("no such button", "returns the old shape") rather than a failure — keep the column, never drop it. If the BEFORE state turns out to be **not reproducible**, say so at the top and stop for a decision rather than writing an AFTER column for a defect that may not exist.

## Goal

Make "it works now" provable instead of asserted. Write the plan after `/mytask` so the before-state is captured while the code is still broken — that evidence cannot be recovered once the fix lands — then run the AFTER column once `/myscope` confirms the diff is clean, and hand the automated test to whoever implements it.
