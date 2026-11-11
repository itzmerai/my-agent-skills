# /myprobe

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myprobe` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

A controlled experiment against the running app: two or more runs that differ in exactly one variable — identity. The **difference** is the evidence, never either status code on its own.

> **Throwaway by design.** The probe lives in a staging folder outside the repo, is hash-locked at baseline, and never enters git. Nothing is committed or pushed.

## Install (Claude Code)

```bash
git clone --branch skill/myprobe https://github.com/itzmerai/my-agent-skills.git myprobe-skill
ln -s "$PWD/myprobe-skill/skills/myprobe" ~/.claude/skills/myprobe
```

Restart Claude Code, then type `/` and you should see `/myprobe`.

## The cycle

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

Run it twice: once before you build, once after.

## Why a live probe, and not just a feature test

Broken access control is unusual among bug classes: **from the privileged seat, the broken state and the fixed state look identical.** An admin exercising the app sees everything work either way, so no amount of single-perspective testing surfaces it.

And an in-process test, however good, skips the proxy, real session cookies, CSRF round-trips, middleware ordering as deployed, and cached config and routes. Authorization bugs live in exactly those seams — a policy that exists but is never invoked, middleware registered on the wrong route group. This catches *"the fix is in the code but not in the running system."*

## What makes it discriminating

**Three arms, never two.** Denied, allowed, and a control that is **not** the superuser. An admin passes every conceivable gate, so a viewer-versus-admin pair cannot tell a correct gate from one that is far too broad. The third arm is what catches over-denial.

**The role under test is never created.** Fixtures come from the project's own seeders, cited `file:line`. A role the project doesn't define is a *stop* that names the missing fixture — never a `firstOrCreate`. A fabricated role with no permissions is refused by any deny-by-default gate, so a green run on one proves nothing at all.

**Identity is proven before probing.** One whoami request per arm. Without it a failed login returns unauthenticated responses that read as correct denials — silently, because login errors are easy to discard.

**The distinction is asserted, not the value.** Denied is exactly the project's denied code; allowed is *anything but* it. Pinning "allowed" to `200` manufactures false alarms — an authorized user hitting a deliberately invalid id gets a downstream error — and expectations that are wrong on purpose teach you to ignore the column.

**The side effect is checked.** The resource is read back as the allowed arm. A denial returned *after* the write already landed is the bug wearing the fix's clothes.

## Red is inverted

In an ordinary before/after test, red means the test fails. Here **red means the denied arm succeeds**. So the rule inverts too:

> A denied arm that is *already* refused at baseline is **not** a green light — either the bug isn't real, or that arm isn't authenticated at all.

The identity check is what tells those two apart.

## Exit codes

| | |
|---|---|
| `0` | every arm as expected |
| `1` | an arm is wrong — a real result, read the table |
| `2` | **could not run** — stack down, login failed, route 404s for everyone |

`2` earns its place: without it, a failed login reports a pass it never established.

## Universal, not framework-shaped

Every project-specific fact is resolved from the repository, then recorded once in `CLAUDE.md` so later runs ask nothing:

| Concern | Resolved from |
|---|---|
| Reach the app | compose port map → `APP_URL` / `VITE_*` / `NEXT_PUBLIC_*` → dev-server command → README |
| Authenticate | **a login helper the test suite already has** → the login route handler → README |
| Obtain each role | an account the seeder already provides → a documented seed command |
| Denied status code | the project's existing authorization tests — some APIs return `404` deliberately, so as not to leak existence |
| Routes and verbs | the routes file, confirming each exists for *someone* first |

Reuse the suite's own login routine wherever one exists — it already encodes the real flow, and re-deriving it by hand is how a `419` or a silent redirect becomes a false pass.

## How many questions it asks

**One, and often zero.** The control arm is the only fact neither the ticket nor the code can supply: tickets are usually silent on whether a neighbouring role keeps access, and reading the code to set that expectation assumes the conclusion under test. When the acceptance criteria enumerate the roles, it asks nothing.

Everything else that can't be resolved is a **stop, not a question** — a question invites an answer from memory, and a remembered fixture is the exact failure this skill exists to prevent.

## Storage

```
<staging>/ENG1234.probe.sh               # runs in place, stays runnable by hand
<staging>/ENG1234.probe.baseline.json    # per-arm results, SHA256, exit code, commit SHA
```

The staging folder is resolved with the same ladder `/mytdd` uses — `CLAUDE.md`, then an existing folder under `.claude/`, then asks once — preferring the workspace root so one folder serves every sub-project.

Never a session scratchpad: it dies with the session, and the gap between the two runs is exactly where a session ends.

## Why the script can't change

Hashed at baseline, re-verified on the after-run. **Same bytes, no flags** — a flag means the two runs execute different code and the comparison stops being one. **Self-contained**, with no shared helper sourced in, or the lock covers only half the behaviour.

## Pairs with

`/mytdd` is the in-process sibling: it proves the *mechanism* changed, this proves it changed **in the running system**, and a permanent suite test stops it changing back. `/myrepro` is the same differential written for a human to walk through in a browser.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
