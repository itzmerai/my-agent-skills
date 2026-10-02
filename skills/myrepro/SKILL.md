---
name: myrepro
description: Turn a ticket into concrete before-and-after verification steps. Use when the user runs /myrepro, whenever the user shares a ticket or issue and wants to know how to reproduce, test, or verify it, or asks to "repro this", "how do I test this", "write test steps", or "verify before and after". First classifies the ticket as a bug or a feature from its content. For a bug: writes exact reproduction steps (preconditions, environment, inputs, actions) that trigger the defect on the current code, states the observed broken behavior versus the expected behavior, then writes verification steps confirming the fix resolves it and that nearby behavior still works. For a feature: writes steps showing the baseline behavior before the change (feature absent or old behavior), then acceptance steps demonstrating the new behavior after implementation, mapped to each acceptance criterion in the ticket, including edge cases and negative paths. Whenever the behavior is reachable in a browser -- a layout bug, a role or permission bug, a wrong number on a screen, anything with a visible symptom -- the plan ends with a full walkthrough that starts at the login screen: which account and role to sign in with, where those credentials come from in the repo, every navigation step by its real on-screen label, the exact input, and what to expect at each step. Role-based tickets get one pass for the affected role and a second for a control role that should behave differently. Where possible, runs the repro against the code to confirm the before state actually reproduces, and suggests an automated test that captures it. Every step states what to expect BEFORE (on today's code) and what to expect AFTER (once the work is done). If the ticket is ambiguous or missing details needed to reproduce, lists the gaps instead of guessing. Do NOT use for auditing file changes after work (use /myscope) or for triaging PR review findings (use /myverdict).
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
- **If a browser can reach it, start the walkthrough at login.** Whatever the ticket is about -- a misaligned card, a permission check, a total that comes out wrong -- if a tester could get there in a browser, the steps run from the login screen to logout: the URL to open, the account and role to sign in as, every click and navigation by its real on-screen label, the exact input, and what each step should show. Never drop the reader in at "once you are on the settings page". The reader has not seen this app before.
- **Never invent credentials.** Take test accounts from the project's own seeders, factories, fixtures, `.env.example`, or setup docs, and cite the file and line each came from. If the repo has none, write `<ask ticket author>` and list it as a gap -- a plausible-looking `admin@example.com` that does not exist wastes the tester's first ten minutes.
- **Always end with the step-by-step repro.** It is the last section, after the gaps, and every step carries both an `Expected` and an `Actual (today)`. A plan that stops at the matrix leaves the reader without a walkthrough to actually follow.
- **Print the plan in the chat/terminal** using the exact output format below.

## Steps

1. **Classify the ticket** as **Bug** or **Feature** from its content. If `/mytask` already classified it, reuse that verdict and say so rather than re-litigating it.

2. **Extract what must be proven.** For a bug, the defect statement — what is broken, where, under what conditions. For a feature, every acceptance criterion in the ticket, listed and numbered so steps can map to them.

3. **Locate the code path.** Find the routes, components, handlers, models, or jobs involved, so the steps reference real names. Note the entry point a tester would start from.

4. **Pick the test surface, and the accounts.** Decide how a tester actually reaches this: **browser/UI**, **API client**, **CLI**, or **not exercisable here**. If a browser can reach it at all the surface is UI -- that includes role and permission bugs, data bugs with a visible symptom, and anything a user would notice. Then find the logins: search the project's seeders, factories, fixtures, `.env.example`, and setup docs, and record the file and line for each. For a role-based ticket pick **two** -- the role the ticket is about, and a control role that should behave differently. That contrast is what proves the bug is about permissions rather than about the page.

5. **Write the BEFORE state.**
   - **Bug** — preconditions, environment, account/role, data fixtures, then exact actions that trigger the defect, and the observed broken behavior versus what the ticket says should happen.
   - **Feature** — the baseline: what happens today, whether that is "the feature does not exist", "the old behavior does X", or "the button is absent".

6. **Try to run it.** Reproduce the bug or capture the baseline using the project's own tooling — dev server, CLI, an existing test, a targeted query. Record the outcome as *confirmed*, *not reproducible*, or *not runnable here*, with the command used.

7. **Write the AFTER state**, mapped one-to-one to the defect or to each acceptance criterion. Cover:
   - The happy path for each criterion
   - **Edge cases** — empty, zero, maximum, boundary, duplicate, concurrent, offline
   - **Negative paths** — invalid input, missing permission, expired session, wrong state — and what the user should see instead of a crash

8. **Add regression checks.** Name the nearby behavior most likely to break: anything sharing the touched utility, route, component, table, or migration. Each still gets a before/after pair — for these, the two are usually *identical*, and that is exactly what makes them regression checks.

9. **Suggest an automated test.** Detect the project's framework and existing test layout, then name the file path to add, the case name, and the assertion that would have caught this. Prefer the cheapest level that captures it — unit over integration, integration over end-to-end.

10. **List the gaps.** Anything ambiguous, missing, or unverifiable, phrased as a question the ticket's author can answer.

11. **Write the full step-by-step repro** — a walkthrough someone with no prior context can follow start to finish, with an `Expected` and an `Actual (today)` on every step. Shape it to the surface from step 4:

    - **Browser / UI — the default whenever a browser can reach it.** Begin at a cold start and end at a clean one:
      1. the command that brings the app up, and the base URL
      2. open the login page at its real path
      3. sign in with a named account and password, and say which role that is
      4. every navigation step by its real on-screen label — sidebar item, tab, breadcrumb, button
      5. the exact input to type or file to upload
      6. the element to look at, and what it should say
      7. log out

      Name things as they appear on screen, not as they appear in the code: *"Click **Settings** in the left sidebar → open the **Members** tab → expect a **Role** column in the table; actual: the column is absent."*

    - **Role-based** — run the whole walkthrough once per role, in separate tables: the affected role, then a control role that should behave differently. The control pass is what proves the defect is about permissions and not about the page. Include what each role should *not* be able to reach, and what they see instead — a 403, a redirect, or a hidden control.

    - **Backend, config, or CLI** — the exact commands, using the project's real tooling and paths (`docker compose exec app …`, `php artisan …`, `npm run …`).

    - **Security or infrastructure that cannot be exercised here** — still write the steps and the expected result, put "unverifiable from this repo" in `Actual`, and name the environment that would settle it. Never skip the section because it cannot be run locally.

    End it with a single copy-paste **run command** block in the project's own convention — the one command that executes the check.

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

### ▶️ Step-by-step repro — full walkthrough
**Surface:** [Browser (UI) / API client / CLI / not exercisable here]
**Base URL:** [http://localhost:8000]
**Bring it up:** `[the command that starts the app]`

**Accounts**
| Role | Login | Password | Where it came from |
|---|---|---|---|
| [affected role] | [email or username] | [password] | `database/seeders/UserSeeder.php:21` |
| [control role] | ... | ... | ... |

**Pass 1 — [role], the affected one**
| # | Step | Expected | Actual (today) |
|---|---|---|---|
| 1 | Open `http://localhost:8000/login` | login form renders | ✅ as expected |
| 2 | Log in as `[email]` / `[password]` | lands on `/dashboard` as **[role]** | ✅ |
| 3 | [navigate — the real sidebar item, tab, or button by its on-screen label] | [what that screen shows] | ✅ |
| 4 | [the action that triggers it — exact input, exact click] | ✅ [the required behavior] | ❌ [what happens right now] |
| 5 | [any follow-on check — reload, revisit, confirm persistence] | ... | ... |
| 6 | Log out | back at `/login` | ✅ |

**Pass 2 — [control role], should behave differently**  *(role-based tickets only)*
| # | Step | Expected | Actual (today) |
|---|---|---|---|
| 1 | Log in as `[control email]` / `[password]` | lands on `/dashboard` as **[control role]** | ✅ |
| 2 | [try to reach the same screen] | [403 / redirect / control hidden] | ... |

Run command:
```bash
[copy-paste block using this project's own paths and tooling]
```
````

The walkthrough above is the **browser** shape. When the surface is an API client or CLI, keep the same `# / Step / Expected / Actual (today)` table and drop what does not apply — no Accounts table, no Pass 2 — with each step an exact command and its expected output or status code. When the surface is **not exercisable here**, still write the steps and expected results, put "unverifiable from this repo" in every `Actual`, and name the environment that would settle it.

If the repo has no seeded test accounts, put `<ask ticket author>` in the Accounts table and list it in the gaps — do not fill in a plausible-looking login that does not exist. If the app has no authentication at all, say so in one line and start the walkthrough at the first URL instead.

If the ticket is a **Feature**, the `Expect BEFORE` column is the baseline ("no such button", "returns the old shape") rather than a failure — keep the column, never drop it. If the BEFORE state turns out to be **not reproducible**, say so at the top and stop for a decision rather than writing an AFTER column for a defect that may not exist.

## Goal

Make "it works now" provable instead of asserted. Write the plan after `/mytask` so the before-state is captured while the code is still broken — that evidence cannot be recovered once the fix lands — then run the AFTER column once `/myscope` confirms the diff is clean, and hand the automated test to whoever implements it.
