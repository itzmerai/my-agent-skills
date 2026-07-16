---
name: myfix
description: Address and implement PR review findings in the code — work through P0–P2 findings (P3 optional), locate the affected code, apply the fix, and flag any that change business logic or existing behavior. Use when the user runs /myfix, or asks to "fix the findings", "implement the review feedback", "address the P0/P1/P2 issues", "resolve these review comments in code", or after /myfindings has triaged findings. Edits code but never runs git commit/push — ends by handing off to /mypr for the commit + PR text.
---

# /myfix — Address & implement PR findings

Take triaged PR review findings and actually fix them in the code. Prioritize the required severities (P0, P1, P2), locate the real code behind each finding, implement the fix, and clearly report what changed — especially anything that alters business logic or existing behavior. **This skill edits code. It never runs git state-modifying commands (`git commit`, `git add`, `git push`, etc.) — hand those off to /mypr.**

Pairs with `/myfindings`: that skill triages and gates; this one implements.

## Rules

- **NO git state-modifying commands.** Never run `git commit`, `git add`, `git push`, `git reset`, etc. Read-only inspection (`git status`, `git diff`, `git log`) is allowed to understand the change. When done, point the user to `/mypr` for the commit message + push command + PR brief.
- **Fix the required set by default.** Address P0, P1, and P2. Treat P3 as optional — only do P3 if the user asks. If severities aren't provided, run the same triage as `/myfindings` (or ask the user to run it first) before fixing.
- **Work from real code.** For each finding, find the actual file/line it refers to and read enough surrounding context to fix it correctly. Don't guess-patch. If a finding is ambiguous or you can't locate it, flag it and skip rather than fix the wrong thing.
- **Match the surrounding code.** Follow the existing style, naming, and idioms of the file you're editing.
- **Surface logic/behavior impact.** If a fix changes business logic, alters existing behavior, or affects other call sites, call it out explicitly per finding — don't bury it.
- **Verify when cheap.** If the project has an obvious build/test/lint/typecheck command, run it after fixing to confirm nothing broke. If not, say what you did and didn't verify. Never claim something passes that you didn't run.

## Steps

1. **Get the findings.** Use the findings the user provides, the output of a prior `/myfindings` run, or findings already in the conversation. If there are none, ask for them (or suggest `/myfindings` first) and stop.
2. **Confirm scope.** State which findings you'll fix (P0–P2 by default) and which you'll skip (P3 unless asked). If any finding is ambiguous, note it now.
3. **Fix each finding, most critical first (P0 → P1 → P2):**
   - Locate the affected code and read the relevant context.
   - Implement the fix in the smallest correct way.
   - Note if it changes logic / behavior / other call sites.
4. **Verify** with the project's build/test/lint command if one exists. Report the result honestly.
5. **Report** what was fixed, what was skipped and why, and any behavior changes (format below).
6. **Hand off** — tell the user to run `/mypr` to get the commit message, push command, and PR brief. Do not commit or push yourself.

## Output format

After making the edits, print a summary in this order:

```
Fixed:
🔴 P0:
- <finding> → <what you changed, files touched> [⚠️ logic/behavior change: <detail>]
🟠 P1:
- <finding> → <what you changed, files touched>
🟡 P2:
- <finding> → <what you changed, files touched>

Skipped:
⚪ P3 (optional): <count> — <list briefly, or "not requested">
❓ Ambiguous / not located: <finding> — <why skipped, what you'd need>

Verification: <command run and result, or "not run — no build/test command found">

⚠️ Behavior changes: <anything that alters existing logic or affects other call sites, or "none">

Next: run /mypr to generate the commit message, push command, and PR brief. (This skill does not run git.)
```

- Omit a severity block if it had no findings to fix.
- Be concrete in "what you changed" — reference `file:line` where useful.
- If verification fails, report the failure and what's still broken rather than papering over it.
