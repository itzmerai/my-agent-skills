# /myfrontend-design

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myfrontend-design` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Build distinctive, production-grade frontend interfaces — components, pages, landing pages, dashboards — with a bold, intentional aesthetic direction that avoids generic "AI slop" design.

> **Standalone skill.** Not part of the PR-workflow pipeline. Like every skill in this suite, it never runs git for you.

## Install (Claude Code)

```bash
git clone --branch skill/myfrontend-design https://github.com/itzmerai/my-agent-skills.git myfrontend-design-skill
ln -s "$PWD/myfrontend-design-skill/skills/myfrontend-design" ~/.claude/skills/myfrontend-design
```

Restart Claude Code, then type `/` and you should see `/myfrontend-design`.

## Usage

Describe the interface you want built:

```
/myfrontend-design
A landing page for a specialty coffee subscription — warm, editorial, unexpected.
```

You get working code (HTML/CSS/JS, React, Vue, etc.) built around a committed aesthetic direction, with attention to typography, color, motion, spatial composition, and atmosphere — not cookie-cutter defaults.

## Credits

Adapted from the `frontend-design` skill via [claude-code-templates](https://github.com/davila7/claude-code-templates) (aitmpl.com), MIT licensed. Original terms apply.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
