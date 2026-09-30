---
name: myfix
description: Address and implement PR review findings in the code — fix what /myverdict ruled "address now", locate the affected code, apply the fix, and flag any that change business logic or existing behavior. Findings ruled defer or reject are left alone unless the user explicitly names one to fix as well. Use when the user runs /myfix, or asks to "fix the findings", "implement the review feedback", "address the P0/P1/P2 issues", "resolve these review comments in code", or after /myfindings has triaged findings. Edits code but never runs git commit/push — ends by printing a one-line commit message and a push command for the fixes, ready to copy.
---

# /myfix — Address & implement PR findings

Take triaged PR review findings and actually fix them in the code. Prioritize the required severities (P0, P1, P2), locate the real code behind each finding, implement the fix, and clearly report what changed — especially anything that alters business logic or existing behavior. **This skill edits code. It never runs git state-modifying commands (`git commit`, `git add`, `git push`, etc.) — it prints them for you to run.**

Pairs with `/myverdict`: that skill rules on which findings are real, in scope, and worth doing; this one implements the survivors. Without a verdict, it falls back to severity (P0–P2).

## Rules

- **NO git state-modifying commands.** Never run `git commit`, `git add`, `git push`, `git reset`, etc. Read-only inspection (`git status`, `git diff`, `git log`) is allowed to understand the change. When done, **print** a one-line commit message and the push command for the user to run themselves.
- **Fix commits are not PRs.** By the time findings exist the pull request is already open, so these fixes need a commit and a push — not a PR brief. `/mypr` is for *opening* the PR; do not send the user back to it. The only exception: if no PR exists yet, say so and point to `/mypr`.
- **Fix what `/myverdict` ruled "address now" — nothing else.** The verdict decides, not the severity label. Findings ruled **defer as follow-up** or **reject** were examined and judged; quietly fixing them anyway reintroduces exactly the scope creep `/myverdict` exists to prevent.
- **The user can add to the set, explicitly.** If the user names a specific deferred or rejected finding ("also fix #4"), fix it — then record in the report that it was deferred or rejected, and why, so the change in the PR's scope is on the record. Never promote one on your own initiative.
- **No verdicts available?** Fall back to severity: fix P0, P1 and P2, skip P3, and say plainly that you did so because no `/myverdict` run was found. Suggest running it first for a scope check.
- **Work from real code.** For each finding, find the actual file/line it refers to and read enough surrounding context to fix it correctly. Don't guess-patch. If a finding is ambiguous or you can't locate it, flag it and skip rather than fix the wrong thing.
- **Match the surrounding code.** Follow the existing style, naming, and idioms of the file you're editing.
- **Surface logic/behavior impact.** If a fix changes business logic, alters existing behavior, or affects other call sites, call it out explicitly per finding — don't bury it.
- **Verify when cheap.** If the project has an obvious build/test/lint/typecheck command, run it after fixing to confirm nothing broke. If not, say what you did and didn't verify. Never claim something passes that you didn't run.

## Steps

1. **Get the verdicts.** Prefer the output of a prior `/myverdict` run — that is the authoritative list. Otherwise use `/myfindings` output or findings the user pasted, and note you are working without a scope check. If there is nothing at all, ask for it and stop.
2. **Confirm the set.** State exactly what you will fix (everything ruled *address now*, plus anything the user named), and what you are leaving alone (deferred, rejected, unvetted P3). If any finding is ambiguous, note it now.
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

Also fixed on request:
- <finding> (was: deferred / rejected) → <what you changed> [⚠️ <why it was set aside — scope, value — so the record is honest>]

Left alone:
⏭️ Deferred as follow-up: <list> — belongs in its own ticket
❌ Rejected: <list> — <reason from /myverdict>
⚪ P3 (not vetted): <count> — <list briefly>
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

- Omit any block that has nothing in it — including "Also fixed on request" when the user added nothing.
- Never move a finding from **Left alone** to **Fixed** without the user asking for it by name.
- Be concrete in "what you changed" — reference `file:line` where useful.
- If verification fails, report the failure and what's still broken rather than papering over it.
- Keep the commit message about the *fixes*, not the original feature — the feature's commit already exists. Match the repo's convention (check `git log --oneline -5`); default to conventional commits otherwise.
- If the branch has no open PR yet, skip the push block and point to `/mypr` instead.
