---
name: myreviewer
description: Review a plan against its originating task to judge whether the plan truly addresses the task and stays in scope. Use when the user runs /myreviewer, or asks to "review this plan", "does this plan match the task", "is the plan aligned", "check if the plan covers everything", or "is this plan good before I build it". Cross-checks every task requirement against the plan, flags gaps, scope creep, wrong assumptions, and misaligned steps, then gives a verdict (Aligned / Partially Aligned / Misaligned) with specific fixes. Review only — does not create the plan, implement it, or write code.
---

# 🔎 /myreviewer — Plan vs Task Alignment Review

Review a plan against its originating task and judge whether the plan **truly addresses the task** and stays in scope. This is the QA gate between "plan created" and "start building."

## Rules

- **Review only.** Do NOT create or rewrite the plan, implement it, or write production code. Planning stays with the user's planning agent; this skill only reviews. You may suggest specific fixes, but you do not apply them.
- **Judge against the task, not your own preferences.** The bar is "does the plan address *this task*," not "is this how I'd do it." Only flag deviations that actually break alignment, add unjustified scope, or rest on a wrong assumption.
- **Ground findings in evidence.** Quote or reference the specific task requirement and the specific plan step for every issue. Do not hand-wave.
- **Print the review in the chat/terminal** using the exact output format below.

## Steps

1. **Gather two inputs** — the **task** (ticket / requirement / goal) and the **plan** (the approach or steps to review). If either is missing, ask the user for it before reviewing. Do not proceed on guesses.

2. **Extract the task's real intent.** Break the task down into concrete requirements, acceptance criteria, and any clearly implied goals. This becomes the checklist you measure the plan against.

3. **Map plan → task.** For each task requirement, check whether the plan actually covers it, and classify:
   - ✅ **Covered** — a plan step properly handles this requirement.
   - ❌ **Gap** — the requirement is missed entirely by the plan.
   - ⚠️ **Scope creep** — a plan step that is not justified by the task.
   - 🔧 **Misalignment** — a step that addresses the task but the wrong way, or on a wrong/unstated assumption.

4. **Light feasibility check.** If the repository is available, quickly verify plan steps against reality (Grep/Glob/Read) and flag any step that conflicts with the actual codebase (references a file/module/API that doesn't exist, contradicts existing structure, etc.). Keep this light — deep feasibility analysis is not the goal.

5. **Give a verdict.** Assign an overall rating and a short, specific list of what to add, cut, or change before building.

## Rating scale

- **Aligned** — the plan covers all task requirements with no meaningful gaps or scope creep. Safe to build.
- **Partially Aligned** — the plan is mostly on track but has fixable gaps, scope creep, or misalignments. Address the fixes first.
- **Misaligned** — the plan does not address the core of the task, or is built on a wrong assumption. Needs replanning before building.

## Output format

Print exactly this structure:

```
### 🎯 Task Intent
- [real requirements extracted from the task]

### 🧭 Plan vs Task
| Requirement | Covered? | Notes |
|---|---|---|
| ... | ✅ / ❌ / ⚠️ / 🔧 | ... |

### 🚩 Issues
- Gaps:
- Scope creep:
- Misalignment:

### ✅ Verdict
- Rating: [Aligned / Partially Aligned / Misaligned]
- Fix before building:
  - ...
```

## Goal

Ensure a plan genuinely addresses its task — no missed requirements, no unjustified scope, no wrong assumptions — so development starts only when the plan is proven aligned. Planning is done elsewhere; this skill is the check before the build.
