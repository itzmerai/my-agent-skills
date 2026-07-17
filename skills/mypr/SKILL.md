---
name: mypr
description: Generate a one-liner commit message and a short PR brief for the current changes, printed in the terminal for copy-paste. Use when the user runs /mypr, or asks to "write a PR description", "make a PR brief", "give me a commit message + PR", or is wrapping up a task and wants something to paste into GitHub. Never runs git commit/push — output only.
---

# /mypr — Commit message + PR brief

Print a one-liner commit message and a filled-in PR brief for the current changes so the user can copy-paste them. **Output only — do not run any state-modifying git commands.**

## Rules

- **NO git state-modifying commands.** Never run `git commit`, `git add`, `git push`, etc. You are only producing text for the user to copy.
- **Read-only inspection is allowed and expected.** Use `git status`, `git diff`, `git diff --staged`, and `git log --oneline -10` to understand what actually changed. Base everything on the real diff, not guesses.
- **Print everything in the chat/terminal**, formatted so the whole thing is easy to copy. Do not write it to a file unless asked.

## Steps

1. Inspect the working tree (read-only):
   - `git status` to see staged/unstaged/untracked files.
   - `git diff` and `git diff --staged` to see the actual changes.
   - `git rev-parse --abbrev-ref HEAD` to get the current branch name for the push command.
   - Determine the **base branch** the PR should target (used in Step 4): check the remote's default branch with `git symbolic-ref refs/remotes/origin/HEAD` and whether an integration branch like `develop` exists via `git branch -r`.
   - If there are no changes, say so and stop.
2. Detect the project type (web vs mobile app vs both) from the files/stack (e.g. `package.json` + React/Next = web; `android/`, `ios/`, Flutter, React Native = app). This drives the build checklist line and the **Tested on** section — drop Android/iOS rows for a pure web project, keep them for mobile/cross-platform.
3. Produce a **one-liner commit message** followed by a **push command**, in this exact form so they're copy-ready:

   ```
   git commit -m "concise imperative summary of the change"
   git push -u origin <current-branch-name>
   ```

   - Commit message: keep it short, imperative mood, lowercase after any conventional-commit prefix if the repo uses one (check `git log` for the convention).
   - Push command: fill `<current-branch-name>` with the real branch from `git rev-parse --abbrev-ref HEAD` — do not leave the placeholder. If the branch is a default branch (`main`/`master`), still print the command but note that the user may want to push to a feature branch instead.
   - These are **text for the user to copy** — never run them.

4. Determine **where to base the PR** — the branch it should merge *into* (its target) — and print how to open it, right after the push command:
   - Choose the base branch:
     - Prefer `develop` if it exists on the remote (the integration branch `/mytask` branches off) — check `git branch -r` for `origin/develop`.
     - Otherwise use the remote's default branch (`main`/`master`) from `git symbolic-ref refs/remotes/origin/HEAD`.
     - If the branch you just pushed *is* that base, don't open a PR against itself — flag it and suggest a feature branch instead.
   - In the same fenced block as the commit/push, print a copy-ready open command with the real base and head filled in (base = target branch, head = the branch you pushed):

     ```
     gh pr create --base <base-branch> --head <current-branch-name> --web
     ```

   - Also print a no-`gh` fallback URL: `https://github.com/<owner>/<repo>/compare/<base-branch>...<current-branch-name>?expand=1` (fill `<owner>/<repo>` from `git remote get-url origin`). `--web` and the URL both open the PR page prefilled so the user can paste the brief below.
   - **Copy-only** — never run these; `gh pr create` opens a PR (a state-changing action), so it's the user's to run.

5. Produce the PR brief using the template below. Fill it in based on the real diff:
   - **Title**: brief but descriptive.
   - **Description**: simple, close, and brief — what it does, why, relevant context.
   - **Related Issue(s)**: leave the placeholder if none is known; do not invent issue numbers.
   - **Changes**: bullet list of the files modified or added — just the file paths, no explanation per file.
   - **Testing Instructions**: concrete steps and edge cases for this specific change.
   - **Tested on**: include Android/iOS rows only if it's a mobile/cross-platform project; otherwise replace with what's relevant (browser/OS) or drop.
   - **Checklist**: check the boxes that are genuinely satisfied by the current change; leave the rest unchecked. Adjust the first line to the project type (web vs Android/iOS).

## Output format

Print the commit message, push command, and PR-open command (with the base branch filled in) together in one fenced block, then the PR brief inside a fenced ```markdown block so the user can copy the raw markdown cleanly.

## PR brief template

```markdown
# Pull Request Title (Brief but descriptive, e.g., "Fix login button alignment on Android and iOS")

## Description
<!-- Make it simple, close and brief, Provide a clear summary of what this PR does, why it's needed, and any relevant context or motivation. -->

## Related Issue(s)
<!-- Link any related tickets, e.g., Fixes #123 or Related to #456 -->

## Changes
<!-- List the main changes in bullet points , list of file modified or added  just the files don't explain -->
- 
- 
- 

## Testing Instructions
<!-- Detailed steps to test the changes. Include specific scenarios or edge cases. -->
1. 
2. 

**Tested on:**
- Android: [Device/OS version]
- iOS: [Device/OS version]
- Other: [if applicable]

## Checklist  <!-- Always check what is already checked -->
- [ ] Code builds cleanly on (depends what type the project is it web or app so that you can add IOS and android)
- [ ] All tests pass locally
- [ ] No new lint errors or warnings
- [ ] Self-reviewed the code
- [ ] Documentation updated (if needed)

<!-- Additional notes (optional) -->
```
