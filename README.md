# ai-skills

Agent skills I create and use with Claude Code.

Each skill lives in its own top-level directory containing a `SKILL.md` (with
YAML frontmatter: `name`, `description`) plus any supporting `references/`,
`scripts/`, or `assets/` files it needs.

## Skills

| Skill | Description |
| --- | --- |
| [`repo-rundown`](repo-rundown/) | Orients you in an unfamiliar codebase — tech stack, architecture, data flow, repo layout for navigation, and gotchas — delivered one section at a time. |
| [`tactile-ux`](tactile-ux/) | Standards and a phased audit workflow for tactile, high-quality web UI — interaction feedback, async states, motion, forms, keyboard/touch, and WCAG 2.2 AA accessibility. |
| [`tech-tutor`](tech-tutor/) | Walks through a software engineering technology or concept conversationally, one small piece at a time, checking in before moving on. |

## Using a skill

Point your Claude Code skills directory at this repo, or symlink an individual
skill into `~/.claude/skills/`:

```bash
ln -s "$(pwd)/tech-tutor" ~/.claude/skills/tech-tutor
```
