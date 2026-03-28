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
    ├── workflows/         # Deliverable workflow files
    │   ├── suite-bottom-up.md
    │   ├── market-map-org.md
    │   ├── market-map-product.md
    │   ├── investor-network.md
    │   └── peer-benchmark.md
    └── references/
        ├── REFERENCE.md  # Endpoint catalog + pipeline operations
        └── EXAMPLES.md   # Example prompts and request shapes
```

## Notes

- `SKILL.md` is the main agent-readable skill definition.
- `workflows/` contains deliverable workflows (market maps, investor network, peer benchmark, orchestrator). Files prefixed with `suite-` are orchestrators that delegate to other workflows.
- `references/` contains reference material (endpoint catalog, examples).
- `alphalens-api/README.md` explains local copying and publishing usage.
