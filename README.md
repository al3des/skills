# al3des skills

Reusable agent skills published for installation through the open [skills.sh](https://skills.sh) ecosystem.

[![skills.sh](https://skills.sh/b/al3des/skills)](https://skills.sh/al3des/skills)

## Install

List the available skills:

```bash
npx skills add https://github.com/al3des/skills --list
```

Install from the repository URL:

```bash
npx skills add https://github.com/al3des/skills
```

Install one skill explicitly:

```bash
npx skills add https://github.com/al3des/skills --skill model-routing
```

The GitHub shorthand works too:

```bash
npx skills add al3des/skills
```

## Skills

### `model-routing`

Routes orchestrated Pi work across OpenAI Codex GPT models and thinking levels, using the lowest sufficient model and effort while controlling duplicated context, validation, and token use.

This skill is Pi- and OpenAI-Codex-specific. Its bundled research notes capture the source-grounded assumptions behind the routing policy.

## Repository layout

Each reusable skill lives under `skills/<name>/SKILL.md`, which is discovered directly by the Vercel skills CLI.

## License

MIT
