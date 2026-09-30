---
name: myverdict
description: Cross-verify PR review findings before acting on them. Use when the user runs /myverdict, whenever PR review feedback arrives with prioritized findings (P0, P1, P2, or similar severity labels such as critical/major/minor), and whenever the user asks to "triage the review", "check the findings", "verify the PR comments", or "decide which review items to fix". For each finding, validate that the issue is real by checking it against the actual code, then classify it on two axes. Scope: in-scope (directly related to the task) or out-of-scope (unrelated to the task, belongs in a separate ticket). Impact: fixes a defect in the current implementation (bug, regression, broken edge case, security issue), strengthens the current implementation (robustness, error handling, test coverage, clarity) without changing behavior, or adds no value to the task. Also confirms whether the assigned priority matches the actual severity and flags any mislabeled findings. Vets the required severities only — P0, P1 and P2 — and skips P3 unless asked. Produces a per-finding verdict (address now / defer as follow-up / reject with reason) so that only changes that strengthen the task or fix real defects get applied, and out-of-scope suggestions don't creep into the PR. Do NOT use for writing the initial PR review itself, or for auditing changes that were already made (use /myscope for that).
---

# 🧪 /myverdict — Findings Cross-Verification

Cross-verify review findings against the actual code before anyone acts on them. This is the gate between "findings triaged" and "findings implemented" — it answers two questions per finding: **is this real?** and **does this task want it?**

## Rules

- **Vet P0–P2 only.** P0, P1 and P2 are the required set; **P3 is skipped by default** and vetted only if the user asks for it. Matches `/myfix`, which fixes the same set. If the findings carry no severities, triage them first (or ask the user to run `/myfindings`) rather than vetting everything blindly.
- **Skipped is not dropped.** List every P3 you skipped, briefly, so nothing disappears silently. You are deferring the decision, not making it.
- **Verify, never assume.** A finding is a claim, not a fact. Open the cited code and confirm it before accepting. Reviewers — human or AI — report issues that are already handled, no longer present, or simply wrong.
- **Every verdict cites evidence.** Name `file:line` and state what you found there. "Looks fine" is not a verdict.
- **Do not fix anything.** Read-only on code and git (`git status`, `git diff`, `git log`, `git show` only). Accepted findings are handed to `/myfix`; you never edit, commit, or push.
- **Do not write the review.** Producing the initial review is `/code-review` or `/mycodereview`. This skill only judges findings that already exist.
- **Scope is measured against the task, impact against the implementation.** A correct, well-intentioned suggestion that the task never asked for is still out-of-scope.
- **A rejection must carry a reason.** Never drop a finding silently.
- **Print the result in the chat/terminal** using the exact output format below.

## Steps

1. **Gather two inputs** — the **task** (what the PR set out to do) and the **findings** (from `/myfindings`, `/mycodereview`, a GitHub review, or pasted comments). If either is missing, ask before proceeding. Without the task there is no scope axis, and the whole skill collapses into a code review.

2. **Filter to the required set.** Keep P0, P1 and P2; set P3 aside. State the counts up front — "8 findings: 5 in scope for vetting (P0–P2), 3 P3 skipped" — so the user can ask for P3 if they want it.

3. **Validate each finding against the code.** Open the referenced file and lines. Classify reality as:
   - ✅ **Confirmed** — the issue exists as described.
   - 🟡 **Partly right** — a real problem, but mischaracterized (wrong cause, wrong location, overstated).
   - ⛔ **Not reproducible** — the code does not do what the finding claims.
   - 🔁 **Already handled** — covered elsewhere (a guard upstream, a framework default, an existing test).
   - ❓ **Needs runtime proof** — cannot be settled by reading; say what would settle it.

4. **Classify scope** against the task:
   - **In-scope** — concerns code this task wrote or changed, or behavior the task is responsible for.
   - **Out-of-scope** — pre-existing issues, adjacent modules, or improvements the task never touched.

5. **Classify impact** on the task:
   - **Fixes a defect** — bug, regression, broken edge case, security or data-loss risk.
   - **Strengthens** — robustness, error handling, test coverage, clarity; no behavior change.
   - **No value** — stylistic preference, speculative future-proofing, or churn.

6. **Audit the priority label.** Compare the assigned severity (P0–P3, critical/major/minor) against what the code actually shows, and flag both directions — an inflated P1 that is cosmetic, and a "minor" note that is really a data-loss bug. State the corrected level.

7. **Decide a verdict per finding** using the matrix below.

8. **Summarize the hand-off** — exactly which findings `/myfix` should implement, which become follow-up tickets, and which are closed with a reason.

## Decision matrix

| | **Fixes a defect** | **Strengthens** | **No value** |
|---|---|---|---|
| **In-scope** | ✅ Address now | ✅ Address now if small; otherwise defer | ❌ Reject |
| **Out-of-scope** | ⏭️ Defer as follow-up — *unless* it is a security or data-loss risk, which is addressed now with a note explaining why the PR grew | ⏭️ Defer as follow-up | ❌ Reject |

A finding whose real severity turns out to be P3 (step 6 may reveal an inflated label) still gets its verdict — you already did the work. The P3 filter applies to the *incoming* label, not to your corrected one.

Anything **Not reproducible** or **Already handled** is rejected regardless of its priority label, with the evidence that disproves it. Anything **Needs runtime proof** is held, not guessed — say what to run.

## Verdict scale

- **Address now** — real, in-scope, and the task is better for it. Goes to `/myfix`.
- **Defer as follow-up** — real and worth doing, but not this task's job. Becomes its own ticket.
- **Reject** — not real, already handled, or adds nothing. Closed with a reason a reviewer can read.

## Output format

Print exactly this structure:

````
### 🎯 Task
- [what this PR set out to do — the scope yardstick]

### 🧪 Findings Cross-Check
| # | Finding | Real? | Scope | Impact | Priority (given → actual) | Verdict |
|---|---|---|---|---|---|---|
| 1 | [short summary] | ✅ / 🟡 / ⛔ / 🔁 / ❓ | in / out | defect / strengthens / none | P1 → P2 | ✅ Address now |

### 🔍 Evidence
- **#1** `path/file.ts:42` — [what the code actually shows, and why that confirms or disproves the finding]

### ⚖️ Priority Corrections
- **#3** labelled P0, actually P2 — [why]

### ⚪ Skipped — P3 (not vetted)
- **#6**, **#7** — [one-line summary each]. Ask if you want these vetted too.

### ❌ Rejected
- **#5** — [reason a reviewer can read]

### ✅ Hand-off to /myfix
- Address now: #1, #4
- Defer as follow-up: #2 → [suggested ticket title]
- Rejected: #3, #5

[N findings · N vetted (P0–P2) · N to fix · N deferred · N rejected · N skipped (P3)]
````

If a finding cannot be settled by reading the code, say so plainly and name the command, test, or reproduction that would settle it — do not guess a verdict.

## Goal

Make sure only real, in-scope, task-strengthening changes make it into the PR — so defects get fixed, good-but-unrelated suggestions become their own tickets instead of scope creep, and wrong or already-handled findings die with a documented reason instead of wasting a fix cycle.
