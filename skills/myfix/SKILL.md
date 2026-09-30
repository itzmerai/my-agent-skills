---
name: myfix
description: Address and implement PR review findings in the code — work through P0–P2 findings (P3 optional), locate the affected code, apply the fix, and flag any that change business logic or existing behavior. Use when the user runs /myfix, or asks to "fix the findings", "implement the review feedback", "address the P0/P1/P2 issues", "resolve these review comments in code", or after /myfindings has triaged findings. Edits code but never runs git commit/push — ends by printing a one-line commit message and a push command for the fixes, ready to copy.
---

# /myfix — Address & implement PR findings

Take triaged PR review findings and actually fix them in the code. Prioritize the required severities (P0, P1, P2), locate the real code behind each finding, implement the fix, and clearly report what changed — especially anything that alters business logic or existing behavior. **This skill edits code. It never runs git state-modifying commands (`git commit`, `git add`, `git push`, etc.) — it prints them for you to run.**

Pairs with `/myfindings`: that skill triages and gates; this one implements.

## Rules

- **NO git state-modifying commands.** Never run `git commit`, `git add`, `git push`, `git reset`, etc. Read-only inspection (`git status`, `git diff`, `git log`) is allowed to understand the change. When done, **print** a one-line commit message and the push command for the user to run themselves.
- **Fix commits are not PRs.** By the time findings exist the pull request is already open, so these fixes need a commit and a push — not a PR brief. `/mypr` is for *opening* the PR; do not send the user back to it. The only exception: if no PR exists yet, say so and point to `/mypr`.
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
6. **Print the commit** — a one-line commit message describing the fixes (conventional-commit style, matching the repo's existing history), followed by the add/commit/push commands. The user runs them; you never do.

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

Commit the fixes — run these yourself:
```bash
git add -A
git commit -m "fix(<scope>): <what these fixes addressed, in one line>"
git push
```
```

- Omit a severity block if it had no findings to fix.
- Be concrete in "what you changed" — reference `file:line` where useful.
- If verification fails, report the failure and what's still broken rather than papering over it.
- Keep the commit message about the *fixes*, not the original feature — the feature's commit already exists. Match the repo's convention (check `git log --oneline -5`); default to conventional commits otherwise.
- If the branch has no open PR yet, skip the push block and point to `/mypr` instead.
