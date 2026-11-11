---
name: myprobe
description: Write a live role-differential authorization probe from a ticket, run it against the running application before implementation to prove the gate is missing, then run the identical script afterwards to prove it is closed. Use when the user runs /myprobe, or whenever a ticket's acceptance criteria take the form "X may, Y may not" — role gates, permission checks, plan or module entitlement, tenant isolation, guest versus authenticated. Also use when the user asks to "prove the gate works", "does the viewer actually get 403", "test this against the running app", "check it as both roles", "live probe", or pastes a ticket about authorization, privilege escalation, forced browsing, or broken access control. Tests the composition an in-process suite cannot reach — the real proxy, session cookies, CSRF, middleware ordering as deployed, cached config and routes — which is where a policy that exists but is never invoked actually hides. Runs at least three arms (denied, allowed, and a control that is not the superuser), because the superuser passes every conceivable gate and so a two-arm run cannot tell a correct gate from one that is far too broad. Never creates the role under test — fixtures come from the project's own seeders, and a role the seeder does not define is a stop, not a firstOrCreate. Proves which account each arm is authenticated as before probing, so a failed login cannot masquerade as a correct denial. Asserts the distinction rather than the value: denied is exactly the project's denied code, allowed is anything but it. The script is hash-locked at baseline, re-run byte-identical afterwards, lives in the same staging folder as /mytdd, and never enters git. Resolves every project-specific fact — base URL, auth mechanism, seeded accounts, denied status code — from the repository rather than assuming a framework, and records them once in CLAUDE.md. Do NOT use for business-logic correctness, for permanent suite tests, for a human walkthrough (use /myrepro), or when the surface is not request/response over a network boundary (use /mytdd alone).
---

# 🔐 /myprobe — Live Role-Differential Authorization Probe

A controlled experiment: two or more runs that differ in exactly one variable — identity. The **difference** is the evidence, never either status code on its own.

```
ticket  →  classify      probe-shaped?  "X may, Y may not" over a network boundary
        →  resolve       stack · auth · seeded accounts · denied code  →  CLAUDE.md
        →  arms          denied · allowed · a control that is not the superuser
        →  write         asserts, exits 0/1/2, self-contained
        →  BEFORE run    red = the denied arm SUCCEEDS
        →  freeze        SHA256 + baseline saved
        ─────────────────── implementation happens elsewhere ───────────────────
        →  AFTER run     verify hash, same bytes, same command
        →  report        before/after table, ready for the ticket comment
```

Broken access control is unusual among bug classes: **from the privileged seat, the broken state and the fixed state look identical.** No amount of single-perspective testing surfaces it. That is what the differential is for.

## Rules

- **Three arms minimum.** Denied, allowed, and a control that is **not** the superuser. An admin passes every conceivable gate, so a viewer-versus-admin pair cannot distinguish a correct gate from one that is far too broad. The third arm is what catches over-denial.
- **Never create the role under test.** Prefer an account the seeder already provides: zero writes, no cleanup, and it is structurally impossible to invent a role. Seeding is the fallback. A role the project does not define is a **stop** that names the missing fixture — never a `firstOrCreate`. A fabricated role with no permissions is refused by any deny-by-default gate, so a green run would prove nothing.
- **Prove identity before probing.** One whoami request per arm, asserting both the account and the role. Without it a failed login returns unauthenticated responses that read as correct denials — and that failure is silent, because login errors are easy to discard.
- **Assert the distinction, not the value.** Denied is exactly the project's denied code. Allowed is **anything but** that code. Pinning "allowed" to `200` manufactures false alarms — an authorized user hitting a deliberately invalid resource id gets a downstream error, not `200` — and a column of expectations that are wrong on purpose teaches the reader to ignore the column.
- **Check the side effect, not only the code.** Read the resource back over HTTP as the allowed arm. A denial returned *after* the write already landed is the bug wearing the fix's clothes.
- **Frozen at baseline.** SHA256 the script when the baseline is taken; re-verify before the after-run. **Same bytes, no flags** — a flag means the two runs execute different code and the comparison stops being one. **Self-contained**: no sourcing a shared helper, or the lock covers half the behaviour.
- **Never weaken, loosen or drop an arm to make the implementation pass.** Not a relaxed expected code, not a removed request, not a dropped control arm, not an arm quietly re-pointed at a different account. If the implementation cannot satisfy the probe, that is the finding. Before the baseline locks, a script that *cannot run* (exit `2`) is a tooling bug and must be fixed; a script that runs and reports a wrong arm is correct and is left alone. After it locks, nothing changes without the user's explicit approval, recorded in the baseline file.
- **Red means the denied arm SUCCEEDS.** A denied arm that is *already* refused at baseline is **not** a green light: either the bug is not real, or you are not authenticated as that role at all. Investigate, do not proceed. The identity check is what tells those two apart.
- **Exit non-zero, and distinguish three outcomes.** `0` every arm as expected · `1` an arm is wrong — a real result, read the table · `2` **could not run**: stack down, login failed, or a route that 404s for everyone. Without `2`, a failed login reports a pass it never established.
- **Clean up on any exit.** Cookie jars from `mktemp`, never a predictable path; fixtures removed via `trap … EXIT INT TERM`, never a trailing `rm` that a crash or Ctrl-C skips.
- **Unresolvable infrastructure is a stop, not a question.** A question invites an answer from memory, and a remembered fixture is the exact failure this skill exists to prevent. Name the missing thing instead.
- **Never commit, stage or push.** Read-only on git except inspection.
- **Print everything in the chat/terminal.**

## Steps

**First, decide which run this is.** Look for `<staging>/<TICKET>.probe.baseline.json`. If it exists, this is the **AFTER** run — go straight to step 8 and change nothing else: do not regenerate the script, do not re-resolve the environment or the arms. If it does not exist, this is the **BEFORE** run and the steps below run in order. If the user asks to re-baseline a ticket that already has one, print what the existing baseline holds and require explicit confirmation — overwriting it destroys the only record of the broken state, and that state cannot be recreated once the code is fixed.

1. **Resolve the ticket, then classify.** Take it from the session first, then from the argument — Linear, Jira, a GitHub issue or pasted text all work. Ask: do the acceptance criteria say *"X may, Y may not"* across a request/response boundary? If the surface is a CLI, a library, or pure business logic, it is **not probe-shaped** — say so and stop; `/mytdd` covers it alone.

2. **Resolve the environment.** Detect, never assume a framework. Record the answers once under a `## /myprobe` heading in `CLAUDE.md`, the way `/mytdd` records its runner command, so later runs ask nothing.

   | Concern | Resolution order | If unresolved |
   |---|---|---|
   | Staging folder | `/mytdd`'s ladder, unchanged: `CLAUDE.md` → an existing folder under `.claude/` → ask once, proposing `.claude/discriminatingtest/`. Prefer the workspace root's `.claude/`; warn if it sits inside a git repo | ask |
   | Reach the app | compose port map → `APP_URL` / `VITE_*` / `NEXT_PUBLIC_*` → a dev-server command in `package.json`, `Procfile` or `Makefile` → README | stop |
   | Authenticate | **a login helper the test suite already has** (`actingAs`, `sign_in`, `login_as`, a Playwright/Cypress `login()` fixture) → the login route handler → README | ask |
   | Obtain each role | an account the seeder already provides → a documented seed command | stop |
   | Run code in app context | `docker compose exec` → `kubectl exec` → `rails runner` / `manage.py shell -c` / `node -e` → local | stop |
   | Denied status code | grep the project's **existing authorization tests** for what they assert — some APIs return `404` deliberately so they do not leak existence | `403`, and say so |
   | Unauthenticated code | the same tests | `401` for APIs, `302` for session apps |
   | Routes and verbs | the routes file; confirm each one exists for *someone* before probing, or the run tests nothing | stop |

   Reuse the suite's own login routine wherever one exists. It already encodes the real flow, redirects and token handling; re-deriving it by hand is how a `419` or a silent redirect becomes a false pass.

3. **Resolve the arms.** Denied and allowed come from the acceptance criteria. The **control** arm is the one fact neither the ticket nor the code can supply — the ticket is usually silent on whether a neighbouring role keeps access, and reading the code to set the expectation assumes the conclusion under test. If the criteria enumerate the roles, use them and ask nothing. Only when they do not, ask **once**: candidates pre-filled from the seeder, your reading marked as the default, one batched prompt.

4. **Write the probe** to `<staging>/<TICKET>.probe.sh`. It runs in place — no copy-in step — and must stay runnable by hand. Header first; it is the artefact, not decoration:

   ```
   TICKET-ID — one line: who must be refused, on what, and who must still get through.

   THROWAWAY live proof, not shipped. Runs in place against the dev stack.

   ARMS        denied   <role>  <account>  <seeder file:line>
               allowed  <role>  <account>  <seeder file:line>
               control  <role>  <account>  <seeder file:line>  — must <stay/become> <code>

   CODES       denied  = <code>   (learned from <test file:line>)
               allowed = anything but <code>
               unauth  = <code>   → exit 2, never read as a denial

   REQUESTS    <METHOD> <path>    <the criterion it comes from>

   SIDE EFFECT <what is read back as the allowed arm, and what must be unchanged>

   BEFORE: <which arms are wrongly allowed today>   → RED
   AFTER:  <what a correct fix changes>             → GREEN

   NOT covered here (verify in the permanent suite): <honest scope boundary>

   Do not weaken these assertions. Do not edit between runs — the hash is checked.
   ```

   Then: assert each arm, print one line per request naming the arm, the request, the code and the verdict, and exit `0`/`1`/`2`. Default to `bash` + `curl` so it depends on nothing; use the project's e2e runner only when the auth flow genuinely needs a browser.

5. **Take the BEFORE baseline.** Run it, and classify — this table is the inverse of `/mytdd`'s:

   | arm result | meaning | action |
   |---|---|---|
   | denied arm **allowed** | discriminates correctly — this is red | keep |
   | denied arm **already denied** | **not a win** — the bug is unreal, or that arm is not authenticated | check identity, re-baseline |
   | allowed arm denied | the gate is already too broad, or that login failed | stop |
   | exit `2` | tooling, not behaviour | fix the script, re-baseline |

   Compare against the header's `BEFORE:` prediction. A mismatch means the ticket was misread — stop and say so rather than carrying the wrong assumption into the fix.

6. **Freeze.** Write `<staging>/<TICKET>.probe.baseline.json`: per-arm per-request results, the script's SHA256, the exit code, the timestamp, and the commit SHA it ran against. The after-run may happen days later in another session; without this there is nothing to compare to. Never the session scratchpad — it dies with the session, and the gap between the two runs is exactly where a session ends.

7. **Stop.** Implementation is not this skill's job. Say the baseline is locked and which arms must flip.

8. **The AFTER run.** Re-verify the SHA256 **first** — if it differs, halt and report the script was modified, with the diff. Then the identical command on the identical bytes, and report.

## Output format

````
### 🔐 TICKET — live authorization probe
- Probe: `<staging>/<TICKET>.probe.sh`  ·  run with `bash <path>`
- Base URL: `<url>` · Auth: [session+CSRF / bearer / helper reused from `file:line`]
- Denied code: `<code>` (from `file:line`) · Allowed: anything but `<code>`

### 👥 Arms
| Arm | Role | Account | From |
|---|---|---|---|
| denied | viewer | ... | `database/seeders/...:41` |
| allowed | admin | ... | `...:28` |
| control | driver | ... | `...:35` — must keep access |

### 🔴 BEFORE — baseline
| Arm | Request | Code | Reading |
|---|---|---|---|
| viewer | `GET /billing/plans` | 200 | **wrongly allowed** — red, correct |
| admin | `GET /billing/plans` | 200 | authorized |
- Side effect: [what was read back, and its state]
- Predicted: [header's BEFORE] · Actual: [match / MISMATCH — explain]
- Exit `1` · SHA256 `abc123…` · commit `a3f1c2` · baseline saved
- **Locked.** Implement now; do not edit the script.

### 🟢 AFTER — comparison      *(second invocation)*
- Integrity: ✅ hash unchanged  /  ⛔ MODIFIED — halted
| Arm | Request | Before | After | |
|---|---|---|---|---|
| viewer | `GET /billing/plans` | 200 | 403 | gate closed |
| driver | `GET /billing/plans` | 200 | 403 | **over-denied** — control arm lost access |
| admin | `GET /billing/plans` | 200 | 200 | unaffected |

### 📋 Verdict
- Arms correct: n/n · Exit `0`/`1`/`2`
- [**Gate proven** — denied arms refused, allowed arms unaffected, side effect clean]
- [**Not proven** — which arm is wrong and what that says about the fix]

Fixtures and cookie jars removed. Nothing committed.
````

## Goal

Make "the gate works now" a comparison the code could not influence: arms drawn from the project's own seeders, identity proven before the first probe, the distinction asserted rather than a status code eyeballed, and the same bytes run on both sides of the fix — so what survives is evidence a reviewer can read, not a screenshot of the privileged seat where everything always looked fine.
