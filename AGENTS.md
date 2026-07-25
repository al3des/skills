# Repository guidance

Reusable skills live in `skills/<skill-name>/SKILL.md` with valid YAML frontmatter containing `name` and `description`.

Keep each skill self-contained. Put supporting material under that skill's `references/`, `scripts/`, or `assets/` directory and use relative links from `SKILL.md`.

Before publishing changes, verify discovery with:

```bash
npx skills add . --list
```
