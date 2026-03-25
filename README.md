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
    ├── SKILL.md           # Main agent-readable skill definition
    ├── references/
    │   ├── REFERENCE.md  # Endpoint mapping and workflow guidance
    │   └── EXAMPLES.md   # Example prompts and request shapes
    └── README.md         # Local installation and publishing notes
```

## Notes

- `SKILL.md` is the main agent-readable skill definition.
- `references/` contains supplementary documentation loaded on demand.
- `alphalens-api/README.md` explains local copying and publishing usage.
