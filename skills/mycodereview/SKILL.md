---
name: mycodereview
description: Review a GitHub pull request; for your working diff use /code-review. Use when the user runs /mycodereview, or asks to "review this PR", "review PR 123", "review this pull request", or pastes a GitHub PR URL and wants it reviewed. Single-pass narrative review via gh — no finder angles, no subagents, no workflow fan-out.
argument-hint: "[pr number] [additional instructions]"
---

# /mycodereview — GitHub Pull Request Review

Restored from the built-in `/review` skill as it shipped in Claude Code 2.1.222,
before 2.1.223 removed it and repointed `/review` at `code-review`.

## Argument handling

The first whitespace-separated token is the review target. Strip any backticks
and a leading `#` from it — `#1234`, `` `1234` ``, `1234`, and a full GitHub PR
URL are all valid. Everything after that first token is treated as additional
instructions from the user.

**If no target was given**, do this and stop:

> Run `gh pr list` to show the open pull requests, then ask the user which one
> to review (`/mycodereview <number>`).

## Review prompt

Review target: GitHub pull request `<target>`.

Gather this target's diff with (instead of any local `git diff`):

1. `gh pr view <target> --json title,body,author,baseRefName,headRefName,state,additions,deletions,changedFiles,labels` for context
2. `gh pr diff <target>` for the unified diff

The PR's diff is the only review scope — local working-tree changes are out of
scope. When you need surrounding code, Read the files in this checkout if it
matches the PR's branch, otherwise fetch file contents via `gh`.

If the user supplied additional instructions after the target, honor them here.

Analyze the changes and provide a thorough code review that includes:

- An overview of what the PR does
- Analysis of code quality and style
- Specific suggestions for improvements
- Any potential issues or risks

Keep your review concise but thorough. Focus on:

- Code correctness
- Following project conventions
- Performance implications
- Test coverage
- Security considerations

Format your review with clear sections and bullet points.
