# AlphaLens Skills

Installable agent skills for AlphaLens.

## Available Skills

- `alphalens-api`: Skill for using the AlphaLens API for search planning, organization and product discovery, collections, and pipeline workflows.

## Install

Install this repository with:

```bash
npx skills add alphalens-intelligence/alphalens-skills
```

If your runtime prefers a full GitHub URL, use:

```bash
npx skills add https://github.com/alphalens-intelligence/alphalens-skills
```

## Repository Layout

```text
alphalens-skills/
└── alphalens-api/
    ├── SKILL.md
    ├── reference.md
    ├── examples.md
    └── README.md
```

## Notes

- `SKILL.md` is the main agent-readable skill definition.
- `reference.md` contains endpoint mapping and workflow guidance.
- `examples.md` contains example prompts and request shapes.
- `alphalens-api/README.md` explains local copying and publishing usage.
