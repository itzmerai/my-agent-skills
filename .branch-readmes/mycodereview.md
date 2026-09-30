# /mycodereview

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `mycodereview` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Review a GitHub pull request in a **single pass** — gather the diff with `gh`, then produce a readable narrative review. No finder angles, no subagent fan-out, no workflow orchestration.

> **Review only.** This skill reads the PR via `gh pr view` / `gh pr diff` and never runs a state-modifying git command. It reports; you decide.

## Why this exists

Claude Code shipped a built-in `/review` for pull requests up to **2.1.222**. In **2.1.223** that skill was removed and `/review` was repointed at `code-review`, which runs a multi-angle, multi-agent review — thorough, but far slower and more token-hungry than a quick PR read-through.

This skill restores the original single-pass behavior. The review instructions are reproduced from that built-in skill; the argument handling additionally accepts a full PR URL.

## Install (Claude Code)

```bash
git clone --branch skill/mycodereview https://github.com/itzmerai/my-agent-skills.git mycodereview-skill
ln -s "$PWD/mycodereview-skill/skills/mycodereview" ~/.claude/skills/mycodereview
```

Restart Claude Code, then type `/` and you should see `/mycodereview`.

## Usage

Pass a PR number or URL, optionally followed by extra instructions:

```
/mycodereview 383
/mycodereview https://github.com/owner/repo/pull/383
/mycodereview 383 focus on the auth changes
```

Run it with no argument and it lists the open PRs (`gh pr list`) and asks which one to review.

You get:
- **Overview** — what the PR does.
- **Code quality and style** — how the change is written.
- **Suggestions** — specific, actionable improvements.
- **Issues and risks** — what could go wrong.

Focused on correctness, project conventions, performance, test coverage, and security.

## Requirements

The [`gh` CLI](https://cli.github.com/), authenticated (`gh auth login`). The PR's diff is the only review scope — local working-tree changes are ignored. For your uncommitted work, use Claude Code's built-in `/code-review` instead.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
