# My Agent Skills

A collection of [agent skills](https://agentskills.io/home) — packaged instructions agent loads on demand to handle specific kinds of tasks.

## Structure

Skills are grouped into category folders, each containing one or more skills with a `SKILL.md` and YAML frontmatter:

```
skills/
├── category-name/
│   ├── skill-name/
│   │   ├── SKILL.md      # required — frontmatter (name, description) + instructions
│   │   ├── scripts/       # optional — helper scripts the skill can run
│   │   ├── references/    # optional — docs loaded only when needed
│   │   └── assets/         # optional — templates, static files the skill outputs
│   └── another-skill/
│       └── SKILL.md
└── another-category/
    └── skill-name/
        └── SKILL.md
```

Categories are just plain directories — freeform kebab-case names (e.g. `software-engineer`, `cloud`, `analytics`) used to group related skills. They aren't part of the spec; agents don't read them. They exist purely to keep the repo navigable as the number of skills grows.

### SKILL.md format

```markdown
---
name: skill-name
description: One-line, specific description of WHEN to use this skill. This is what triggers it, so be concrete about the situations and keywords that should invoke it.
---

Instructions for how agents should carry out the task once triggered.
```

- `name`: kebab-case, matches the skill's own folder name (not the category folder above it).
- `description`: the single most important field — it's how the agent decides relevance. Be specific about triggers, not just what the skill does.
- Keep the body focused: step-by-step guidance, not a full essay. Push large reference material into `references/` and load it only when needed.

## Discovery

There's no manifest or index file to maintain. At startup, an agent scans the skill folders and loads only each `SKILL.md`'s `name` + `description` (a few dozen tokens each) — that's the whole index. The full `SKILL.md` body is only loaded once a task matches a skill's description. Category folders are purely for human navigation; they play no role in discovery.

## Adding a new skill

1. Copy `example-skill/` to `<category>/<skill-name>/` (create the category folder if it doesn't exist yet).
2. Rewrite `SKILL.md` — name, description, instructions.
3. Delete any unused `scripts/` / `references/` / `assets/` subfolders.
