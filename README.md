# /myseo-optimizer

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myseo-optimizer` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

On-page and technical SEO guidance: keyword strategy, meta tags, schema markup, Core Web Vitals, content structure, internal linking, and a pre-publish SEO checklist.

> **Standalone skill.** Not part of the PR-workflow pipeline. Like every skill in this suite, it never runs git for you.

## Install (Claude Code)

```bash
git clone --branch skill/myseo-optimizer https://github.com/itzmerai/my-agent-skills.git myseo-optimizer-skill
ln -s "$PWD/myseo-optimizer-skill/skills/myseo-optimizer" ~/.claude/skills/myseo-optimizer
```

Restart Claude Code, then type `/` and you should see `/myseo-optimizer`.

## Usage

Point it at content or a page to optimize:

```
/myseo-optimizer
Optimize this blog post's title, meta description, and headings for "react hooks tutorial".
```

You get concrete recommendations — title/meta tags, header hierarchy, schema markup, image and Core Web Vitals fixes — grounded in on-page SEO best practices, plus a checklist to run before publishing.

## Credits

Adapted from the `SEO Optimizer` skill via [claude-code-templates](https://github.com/davila7/claude-code-templates) (aitmpl.com), MIT licensed. Original terms apply.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
