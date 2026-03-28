# AlphaLens Skills

Installable agent skills for AlphaLens.

**Note:** These skills require an active [AlphaLens subscription](https://alphalens.ai) with API access. Contact sales@alphalens.ai for pricing.

## Available Skills

- `alphalens-api`: Skill for using the AlphaLens API for organization and product discovery and pipeline workflows.

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
    ├── workflows/         # Step-by-step workflow files
    │   ├── org-market-map.md
    │   ├── product-market-map.md
    │   ├── investor-network.md
    │   ├── peer-benchmark.md
    │   ├── bottom-up-suite.md
    │   ├── favicon-proxy.md
    │   └── pipeline-enrichment.md
    ├── references/
    │   ├── REFERENCE.md  # Endpoint mapping and workflow guidance
    │   └── EXAMPLES.md   # Example prompts and request shapes
    └── README.md         # Local installation and publishing notes
```

## Notes

- `SKILL.md` is the main agent-readable skill definition.
- `references/` contains supplementary documentation loaded on demand.
- `alphalens-api/README.md` explains local copying and publishing usage.
