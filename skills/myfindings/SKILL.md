---
name: myfindings
description: Parse PR review findings, categorize them by severity (P0–P3), summarize counts, list each finding, flag which fixes are required (P0–P2 only), note logic/behavior impact, and ask the user to confirm the critical findings are addressed before proceeding. Use when the user runs /myfindings, or asks to "validate PR findings", "triage review comments", "categorize findings by severity", "which findings must I fix", or "quality gate this PR". Validation checkpoint only — reports and asks, does not fix code or run git.
---

# /myfindings — PR Findings Validator & Quality Gate

Take a set of PR review findings/comments, sort them into severity buckets, summarize them, and act as a quality gate: make clear that **P0, P1, and P2 must be addressed** before proceeding, note the impact of fixing them, then ask the user to confirm. **Report and ask only — do not modify code, apply fixes, or run any git commands.**

## Rules

- **No fixing, no code changes, no git.** This skill triages and gates. It never edits files, implements fixes, or runs state-modifying commands. If the user then wants fixes applied, that's a separate follow-up.
- **Work from real findings.** Use the findings the user provides (pasted review comments, a review tool's output, or findings already in the conversation). If none are available, ask the user to paste the PR review findings and stop.
- **Don't invent findings.** Only categorize what's actually there. If severity isn't given, infer it from the content using the levels below and say it was inferred.
- **P0–P2 are required; P3 is optional.** Always make this distinction explicit in the output.
- **Print everything in the chat/terminal**, grouped and easy to scan. Don't write to a file unless asked.

## Severity levels

- 🔴 **P0 (Critical)** — Blocking: broken functionality, crashes, security flaws, data loss.
- 🟠 **P1 (High)** — Major issues affecting core behavior or reliability.
- 🟡 **P2 (Medium)** — Important improvements, edge cases, or maintainability concerns.
- ⚪ **P3 (Low)** — Minor: style, formatting, naming, non-blocking suggestions.

## Steps

1. **Gather the findings.** Read the findings the user provided or that exist in the conversation. If there are none, ask for them and stop.
2. **Categorize each finding** into P0 / P1 / P2 / P3 using the severity levels above. If a finding already carries a severity label, respect it; otherwise infer and note that it was inferred.
3. **Count** the findings per severity level.
4. **List** each finding under its bucket with a short, clear one-line explanation.
5. **Highlight required fixes** — state explicitly that P0, P1, and P2 must be addressed before proceeding, and that P3 is optional/non-blocking.
6. **Add an impact note** — indicate whether addressing these findings may change business logic, affect existing behavior, or strengthen the implementation. Be specific to the findings when possible.
7. **Ask the final validation question** — confirm the user has addressed all P0–P2 findings and is aware the changes may affect or improve current logic.

## Output format

Print in this order, plainly formatted for quick scanning:

```
Findings Summary:
🔴 P0: <n>
🟠 P1: <n>
🟡 P2: <n>
⚪ P3: <n>

Details:

🔴 P0:
- <finding> — <one-line explanation>

🟠 P1:
- <finding> — <one-line explanation>

🟡 P2:
- <finding> — <one-line explanation>

⚪ P3:
- <finding> — <one-line explanation>

⚠️ Required Fixes: P0, P1, and P2 must be addressed before proceeding. P3 is optional (non-blocking).

🧾 Impact Note: <whether fixing these may change business logic / affect existing behavior / strengthen the implementation>

❓ Confirmation: Have you addressed all P0, P1, and P2 findings, and are you aware that these changes may affect or improve the current logic?
```

- Omit a severity bucket's detail block if it has zero findings, but always keep it in the Summary count (showing `0`).
- Keep each finding to one line. Don't restate the full review comment — distill it.
