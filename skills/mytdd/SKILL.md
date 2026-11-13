---
name: mytdd
description: Write a throwaway discriminating test from a ticket, run it against the unchanged codebase to establish a red baseline, then run the identical test after implementation and compare. Use when the user runs /mytdd, or asks to "write the test first", "prove this is broken before I fix it", "red-green this ticket", "baseline test", "discriminating test", or "run the before/after test". The test is generated BEFORE implementation and is frozen the moment the baseline is taken — hashed at baseline and re-verified afterwards, never weakened, rewritten or deleted to accommodate the code. It states the outcomes any valid fix must satisfy without pinning the mechanism, and must contain at least one discriminating trap: a case that a plausible but wrong fix would fail, which is what separates this from ordinary tests. The test lives in a staging folder under the project's .claude directory, is copied into the real test tree only for the duration of a run, and is removed again on any exit, so it never enters git and never ships. Resolves the staging folder from CLAUDE.md, then from an existing folder under .claude, and only asks when neither exists. Detects which test runner the ticket's surface uses and runs just that one file. Isolated by design — it writes and runs one test, and hands off to nothing. Do NOT use to write permanent suite tests, to run the full test suite, or to write manual verification steps for a human (use /myrepro).
---

# 🧪 /mytdd — Before/After Discriminating Test

Write the test before the code, prove it fails for the right reason, then prove the implementation turned it green without anyone touching the test.

```
ticket  →  write test       (outcomes, not mechanisms + a discriminating trap)
        →  BEFORE run       copy → run → delete
        →  classify         fail = good · pass = BROKEN TEST · error = fix tooling
        →  freeze           SHA256 + baseline saved
        ─────────────────── implementation happens elsewhere ───────────────────
        →  AFTER run        verify hash, same file, same command
        →  compare          newly passing · still failing · regressions
```

The test is a throwaway proof, not suite code. It never enters git and it is never shipped.

## Rules

- **The test is written before the implementation, and frozen once the baseline is taken.** SHA256 the file at baseline; re-verify it on the after-run. A changed hash halts the run and says so. "Do not modify the test" is checked, not trusted.
- **Never weaken, loosen, rewrite, skip or delete a test to make the implementation pass.** Not a relaxed assertion, not a widened matcher, not a removed case, not an `.only` narrowing the run. If the implementation cannot pass the test, that is the finding.
- **State outcomes, never mechanisms.** The test says *what must be true*, never *how*. Pin an approach and you fail a correct fix that chose a different one. Where several implementations are legitimate, name them in the header and assert the shared outcome.
- **Every test must contain at least one discriminating trap.** A case that a **plausible but wrong** fix fails. Without it you have an ordinary test that any half-fix sails through, and the point is lost. Name the trap explicitly in the header, with the wrong approach it catches.
- **A new-behaviour test that PASSES at baseline is broken, not good news.** It is not exercising the new behaviour. Say so, regenerate it, re-baseline. Do not proceed.
- **Fix a test that cannot run; never a test that fails.** Before the baseline locks, a syntax error, bad import or missing fixture is a tooling bug and must be fixed. A test that *runs and fails* is correct and is left alone. After the baseline locks, nothing changes without the user's explicit approval, recorded in the baseline file.
- **Clean up on any exit.** The copy in the real test tree is removed whether the run passed, failed, crashed or was interrupted. A `trap` or equivalent, never a bare trailing `rm`. Check for orphans from earlier runs before starting.
- **Never run the whole suite.** Run only the generated file. The baseline is about this ticket.
- **Never commit, stage or push.** Read-only on git except for status checks.
- **Print everything in the chat/terminal.**
- **Always end with the exact run command, ready to paste.** Both runs print it, with the real paths and flags filled in — never placeholders — so it never has to be asked for and the user can re-run the test without the skill. Print the paste-safe form `cp … && <runner> …; rm -f …`, **not** the `trap … EXIT` form: a trap pasted into an interactive shell fires when that shell exits, not when the run ends, so the copy would sit in the test tree until the terminal closes.

## Steps

**First, decide which run this is.** Look for `<staging>/<TICKET>.baseline.json`. If it exists, this is the **AFTER** run — go straight to step 7 and change nothing else: do not regenerate the test, do not re-resolve the staging folder or the runner. If it does not exist, this is the **BEFORE** run and the steps below run in order. If the user asks to re-baseline a ticket that already has one, print what the existing baseline holds and require explicit confirmation — overwriting it destroys the only record of the broken state, and that state cannot be recreated once the code is fixed.

1. **Resolve the staging folder**, in this order — ask only as a last resort:
   1. A folder recorded in the project's `CLAUDE.md` → use it.
   2. An existing folder under `.claude/` already holding tests (commonly `.claude/discriminatingtest/`) → use it and say which.
   3. Neither → ask, proposing `.claude/discriminatingtest/`, then create it and record the choice in `CLAUDE.md` along with the runner command. Ask once per project, never again.

   Prefer the `.claude/` of the **workspace root** over a single repo's, so one folder serves every sub-project. Confirm it sits outside any git repository; if it does not, say so — a staging folder inside a repo will be committed.

2. **Read the ticket and identify the surface.** Which code the change lands in decides the runner, the destination directory, the filename convention and the command. Detect rather than assume — a workspace can hold several:

   | surface | runner | runs | filename |
   |---|---|---|---|
   | PHP / Laravel backend | Pest / PHPUnit | often inside a container | `ENG1234Test.php` — class-discovery naming, no hyphen |
   | JS/TS frontend | Vitest | host or container | `ENG1234.test.tsx` |
   | React Native / mobile | Jest | host | `ENG1234.test.tsx` |

   Record the resolved command in `CLAUDE.md`, including the flags the project needs — a memory limit, a config path, a container name. Those are learned once and should not be rediscovered.

   **If the ticket is a role or permission gate** — acceptance criteria of the form *“X may, Y may not”* — this test proves the mechanism in-process but cannot prove the gate is wired into the stack as deployed, where a policy that exists but is never invoked would hide. Say so and name `/myprobe`, which baselines the same ticket against the running application. Both run at this same moment, before implementation; neither replaces the other.

3. **Write the test.** Header first, in this shape — it is the artefact, not decoration:

   ```
   TICKET-ID — one line, what must become true (discriminator).

   THROWAWAY red→green proof, not shipped. Copied in to run, then deleted.

   The outcomes any valid fix MUST satisfy, without pinning the mechanism
   (<name the legitimate alternatives here>):

     1. <outcome, observable, mechanism-free>
     2. <outcome>
     3. <THE TRAP — the case a naive fix fails, and why it is tempting>
     4. <the thing that must keep working>

   BEFORE: <what the code does today, and which outcomes fail>  → RED
   AFTER:  <what a correct fix changes, and why they go green>  → GREEN

   NOT covered here (verify in the permanent suite): <honest scope boundary>

   Do not weaken these assertions.
   ```

   Then: one test case per numbered outcome, named after it, so a failure names the outcome it broke. Note any project-specific gotcha inline where it bites — a base `TestCase` that already handles migrations, a mock the harness needs. Cover the requirement, its edge cases, and the existing behaviour that must survive.

4. **Take the BEFORE baseline.** Copy in, run that one file, remove the copy:

   ```bash
   cp "<staging>/<file>" "<dest>/<file>"
   trap 'rm -f "<dest>/<file>"' EXIT INT TERM
   <runner command> "<dest-relative path>"
   ```

   Classify **every** case:

   | result | meaning | action |
   |---|---|---|
   | ❌ fails | discriminates correctly | keep |
   | ⚠️ **passes** | **broken** — not exercising the new behaviour | regenerate, re-baseline |
   | ⛔ errors | tooling, not behaviour | fix the test, re-baseline |
   | ✅ passes *(regression guard)* | existing behaviour confirmed | must still pass after |

   Compare the result against the header's `BEFORE:` prediction. A mismatch means the ticket was misread — stop and say so rather than carrying a wrong assumption into the implementation.

5. **Freeze.** Write a baseline file beside the test — `<TICKET>.baseline.json`: the per-case results, the test's SHA256, the timestamp, and the commit SHA the baseline was taken against. The after-run may happen days later or in another session; without this there is nothing to compare to.

6. **Stop.** Implementation is not this skill's job. Say the baseline is locked and what must turn green.

7. **The AFTER run.** Re-verify the SHA256 **first** — if it differs, halt and report the test was modified, with the diff. Then the identical copy-run-delete cycle, same command, same file.

8. **Report.**

## Output format

````
### 🧪 TICKET — discriminating test
- Surface: [backend / frontend / mobile] · Runner: [Pest / Vitest / Jest]
- Test: `<staging>/<file>`
- Command: `<the exact command>`

### 🎯 Outcomes under test
| # | Outcome | Trap? |
|---|---|---|
| 1 | ... | |
| 3 | ... | ⚠️ catches [the plausible wrong fix] |

### 🔴 BEFORE — baseline
| # | Case | Result | Reading |
|---|---|---|---|
| 1 | ... | ❌ fail | correct — discriminates |
| 4 | ... | ✅ pass | regression guard |
- Predicted: [the header's BEFORE] · Actual: [match / MISMATCH — explain]
- SHA256: `abc123…` · commit `a3f1c2` · baseline saved to `<TICKET>.baseline.json`
- **Locked.** Implement now; do not touch the test.

### 🟢 AFTER — comparison      *(second invocation)*
- Integrity: ✅ hash unchanged  /  ⛔ MODIFIED — halted
| # | Case | Before | After | |
|---|---|---|---|---|
| 1 | ... | ❌ | ✅ | turned green |
| 3 | ... | ❌ | ❌ | **still failing** — the trap caught it |
| 4 | ... | ✅ | ❌ | **REGRESSION** |

### 📋 Verdict
- Turned green: n/n · Still failing: n · Regressions: n
- [**Implementation proven** — every outcome green, no regressions]
- [**Not proven** — what is still failing and what that says about the fix]

### ▶️ Run it yourself
```bash
cp "<staging>/<file>" "<dest>/<file>" && <runner command> "<dest-relative path>"; rm -f "<dest>/<file>"
```

Test removed from the test tree. Nothing committed.
````

## Goal

Make "it works now" something the code earned against a test it could not influence. The test is written before the implementation exists, carries a trap a plausible-but-wrong fix fails, is hash-locked at baseline, and is deleted after — so what survives is a comparison, not a test that quietly grew agreeable.
