# AlphaLens API Skill Package

This folder contains a publishable skill package for the AlphaLens API.

**Note:** This skill requires an active [AlphaLens subscription](https://alphalens.ai) with API access. Contact sales@alphalens.ai for pricing.

## Contents

- `SKILL.md`: Router — auth, search path selection, workflow routing (~150 lines)
- `references/REFERENCE.md`: Endpoint mapping and workflow rules
- `references/EXAMPLES.md`: Example prompts and request shapes
- `workflows/org-market-map.md`: Org-level market map (Steps 1-6, design rules, PDF export)
- `workflows/product-market-map.md`: Product-centric market map (Step 0 scoring + Steps 1-4)
- `workflows/investor-network.md`: D3 force-directed investor network graph
- `workflows/peer-benchmark.md`: Chart.js peer benchmark with headcount/funding charts
- `workflows/bottom-up-suite.md`: Bottom-up suite orchestration + shared nav bar CSS
- `workflows/favicon-proxy.md`: Node.js favicon proxy server + bad favicon detection
- `workflows/pipeline-enrichment.md`: Pipeline and collection CRUD workflow

## Architecture

`SKILL.md` acts as a thin router. It loads on every query (~150 lines) and contains auth, base URLs, search path selection, and workflow routing logic. The heavier workflow files are only read when the task requires them:

- A simple "find companies similar to X" query loads only `SKILL.md`
- An org-level market map loads `SKILL.md` + `favicon-proxy.md` + `org-market-map.md`
- A bottom-up mapping loads files sequentially as each phase begins

## Local Use

To make this skill available locally in a supported agent runtime, copy this folder to one of:

- `~/.cursor/skills/alphalens-api/`
- `.cursor/skills/alphalens-api/`

The final path must contain `SKILL.md` directly inside the `alphalens-api` directory.

## Publish For Installer Use

If you want to install this with a repo-based installer such as:

```bash
npx skills add your-org/your-skills-repo
```

publish a repository that includes this `alphalens-api/` directory with `SKILL.md` inside it.

The safest layout for a dedicated skills repo is:

```text
your-skills-repo/
└── alphalens-api/
    ├── SKILL.md
    ├── references/
    │   ├── REFERENCE.md
    │   └── EXAMPLES.md
    └── workflows/
        ├── org-market-map.md
        ├── product-market-map.md
        ├── investor-network.md
        ├── peer-benchmark.md
        ├── bottom-up-suite.md
        ├── favicon-proxy.md
        └── pipeline-enrichment.md
```

## Claude Code

For Claude Code, follow that tool's skill/plugin install flow against the published repo. The exact commands can differ by runtime version, but the skill package content in this folder is the part you need to publish.
