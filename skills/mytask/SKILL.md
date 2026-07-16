---
name: mytask
description: Analyze a task, ticket, or bug report to classify it (bug / feature / invalid), verify it against the actual codebase before any work starts, assess impact, and recommend a Git branching strategy. Use when the user runs /mytask, or asks to "verify this ticket", "is this a real bug", "should we build this", "does this feature already exist", or wants a branch name for a task. Recommendation only — never runs state-modifying git.
---

# 🧠 /mytask — Issue & Feature Verification + Branch Strategy

Analyze a given task, ticket, or report and determine whether it is a **real bug**, a **new feature request**, or **invalid / already implemented**. Validate it against the actual codebase *before* any development begins, then recommend a clean Git branching strategy.

## Rules

- **NO git state-modifying commands.** Never run `git checkout -b`, `git branch`, `git switch -c`, `git commit`, `git push`, etc. You only *recommend* the branch name and print the command for the user to run themselves.
- **Read-only inspection is expected.** Use `git status`, `git log`, `git branch`, code search (Grep/Glob/Read), logs, and running the app where relevant to verify claims against reality — do not classify from the ticket text alone.
- **Verify before you conclude.** A bug is only "Confirmed" if you found evidence in the code/behavior. A feature is only "Missing" if you searched and could not find it. Say what you checked.
- **Print the final analysis in the chat/terminal** using the exact output format below.

## Steps

1. **Read the task.** Capture the ticket ID (if any), the described behavior, and any reproduction steps or acceptance criteria. If critical detail is missing, note the assumption you're making rather than guessing silently.

2. **Classify** the task as one of:
   - 🐞 Bug / Issue
   - ✨ Feature Request
   - ❌ Invalid / Already Resolved

3. **Verify** against the actual codebase:

   **🐞 If it's a Bug:**
   - Reproduce it if steps are provided (run the app / flow, hit the API, inspect the UI).
   - Confirm it actually exists; note whether it's consistent or intermittent.
   - Check logs/errors, UI/UX behavior, and API responses.
   - Identify **Expected vs Actual** behavior.
   - → Conclude **Confirmed Bug** or **Not Reproducible**, with a clear explanation of findings.

   **✨ If it's a Feature:**
   - Search the codebase (UI, backend/API, related modules) to see if it already exists fully, partially, or not at all.
   - → Conclude **New Feature**, **Already Implemented**, or **Needs Enhancement**, with an explanation of current system behavior.

4. **Determine the project scope.** Identify whether the task touches:
   - 📱 **App only** — mobile/app project changes only
   - 🖥️ **Backend / Dashboard only** — server, API, admin/dashboard changes only
   - 🔀 **Both** — spans app *and* backend/dashboard

   Base this on the actual affected areas you find during verification, not just the ticket wording.

5. **Assess impact:**
   - Affected areas (UI, backend, database, integrations, etc.)
   - Priority (Low / Medium / High)
   - Risk level

6. **Recommend a branch** off `develop` (unless the user specifies another base), following the naming convention below.

## Branch naming convention

- **Bug:** `fix/<ticket-id>-<short-description>`
- **Feature:** `feat/<ticket-id>-<short-description>`
- **Improvement / Enhancement:** `chore/<ticket-id>-<short-description>`

Use a lowercase, hyphenated short description. If there is no ticket ID, omit that segment (e.g. `fix/login-error`).

Examples:
- `fix/ENG-1529-login-error`
- `feat/ENG-1602-add-user-profile`
- `chore/ENG-1701-refactor-api-handler`

## Output format

Print exactly this structure:

```
### ✅ Task Analysis
- Type: [Bug / Feature / Invalid]
- Status: [Confirmed / Not Reproducible / Already Implemented]
- Project Scope: [App only / Backend-Dashboard only / Both]

### 🔍 Findings
[Detailed explanation — what you checked and what you found]

### ⚠️ Impact
- Area:
- Priority:
- Risk:

### 🌿 Branch Recommendation
- Base Branch: develop
- Branch Name: <suggested-branch-name>
```

Then, since git branch creation is the user's to run, print the ready-to-copy command (do **not** execute it):

```
git checkout develop && git pull && git checkout -b <suggested-branch-name>
```

## Goal

Ensure that no unnecessary work is done, bugs are real and reproducible, features are truly missing, branch naming is clean and consistent, and development starts with clarity and confidence.
