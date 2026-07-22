# /myhumanizer

Part of the [**my-agent-skills**](https://github.com/itzmerai/my-agent-skills) suite — this branch contains **only** the `myhumanizer` skill. See [`main`](https://github.com/itzmerai/my-agent-skills/tree/main) for the full collection.

Remove tell-tale signs of AI-generated writing — inflated symbolism, promotional language, em-dash overuse, rule-of-three, AI vocabulary, negative parallelisms — and add natural, human voice. Based on Wikipedia's "Signs of AI writing" guide.

> **Standalone skill.** Not part of the PR-workflow pipeline. Like every skill in this suite, it never runs git for you.

## Install (Claude Code)

```bash
git clone --branch skill/myhumanizer https://github.com/itzmerai/my-agent-skills.git myhumanizer-skill
ln -s "$PWD/myhumanizer-skill/skills/myhumanizer" ~/.claude/skills/myhumanizer
```

Restart Claude Code, then type `/` and you should see `/myhumanizer`.

## Usage

Give it text to humanize:

```
/myhumanizer
<paste the AI-sounding text here>
```

You get the rewritten text with AI patterns removed and real voice added, optionally followed by a short summary of what changed and why.

## Credits

Original skill by [@blader](https://github.com/blader/humanizer) (humanizer). Retained per its original terms.

## License

[MIT](./LICENSE) © 2026 Ryan Amasora
