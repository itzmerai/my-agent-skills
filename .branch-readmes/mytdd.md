# /mytdd

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `mytdd` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Write the test before the code, prove it fails for the right reason, then prove the implementation turned it green without anyone touching the test.

> **Throwaway by design.** The test is copied into the real test tree only for the duration of a run and removed on any exit. It never enters git and it is never shipped. Nothing is committed or pushed.

## Install (Claude Code)

```bash
git clone --branch skill/mytdd https://github.com/itzmerai/my-agent-skills.git mytdd-skill
ln -s "$PWD/mytdd-skill/skills/mytdd" ~/.claude/skills/mytdd
```

Restart Claude Code, then type `/` and you should see `/mytdd`.

## The cycle

```
ticket  →  write test       (outcomes, not mechanisms + a discriminating trap)
        →  BEFORE run       copy → run → delete
        →  classify         fail = good · pass = BROKEN TEST · error = fix tooling
        →  freeze           SHA256 + baseline saved
        ─────────────────── implementation happens elsewhere ───────────────────
        →  AFTER run        verify hash, same file, same command
        →  compare          newly passing · still failing · regressions
```

Run it twice: once before you build, once after.

## What makes it discriminating

**Outcomes, never mechanisms.** The test says what must be true, never how. Pin an approach and you fail a correct fix that chose a different one. Where several implementations are legitimate, the header names them and the test asserts the shared outcome.

**Every test carries a trap** — a case a *plausible but wrong* fix fails. A gate on `hasPermission('create_invoices')` looks right until the test proves the driver holds that permission for the mobile app and must still be refused on web. Without a trap you have an ordinary test that any half-fix sails through, and the point is lost.

**A test that passes at baseline is broken, not good news.** If it goes green before the code exists, it isn't exercising the new behaviour. Regenerate it.

**The baseline is checked against a prediction.** The header states what should fail before and why. If the real baseline disagrees, the ticket was misread — that surfaces before implementation, not after.

## Why the test can't change

Hashed at baseline, re-verified on the after-run. A changed hash halts the run and reports it. *"Do not modify the test"* is checked, not trusted.

Before the baseline locks, a test that **cannot run** — syntax error, bad import, missing fixture — is a tooling bug and gets fixed. A test that **runs and fails** is correct and is left alone. After it locks, nothing changes without your explicit approval.

## Storage

The test lives in a staging folder under the project's `.claude/`, resolved in this order:

1. A folder recorded in `CLAUDE.md`
2. An existing folder under `.claude/` already holding tests
3. Neither → asks once, proposing `.claude/discriminatingtest/`, then records the choice

It prefers the workspace root's `.claude/` so one folder serves every sub-project, and warns if that folder sits inside a git repository — where it would be committed.

## Runners

Detected from the surface the ticket touches, since one workspace often has several:

| surface | runner | filename |
|---|---|---|
| PHP / Laravel backend | Pest / PHPUnit, often in a container | `ENG1234Test.php` — class-discovery naming |
| JS/TS frontend | Vitest | `ENG1234.test.tsx` |
| React Native / mobile | Jest | `ENG1234.test.tsx` |

The resolved command and its project-specific flags — a memory limit, a container name — are recorded in `CLAUDE.md` so they're learned once.

## Cleanup

```bash
cp "<staging>/<file>" "<dest>/<file>"
trap 'rm -f "<dest>/<file>"' EXIT INT TERM
<runner> "<path>"
```

The trap matters: a bare trailing `rm` never fires when the runner segfaults or you hit Ctrl-C, leaving an untracked file in a tracked test directory for the next `git add .` to sweep up. It also checks for orphans from earlier runs before starting.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
