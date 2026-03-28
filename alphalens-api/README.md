# AlphaLens API Skill Package

This folder contains a publishable skill package for the AlphaLens API.

**Note:** This skill requires an active [AlphaLens subscription](https://alphalens.ai) with API access. Contact sales@alphalens.ai for pricing.

## Contents

- `SKILL.md`: Main skill definition used by agent runtimes
- `workflows/`: Deliverable workflow files (market maps, investor network, peer benchmark, suite)
- `references/REFERENCE.md`: Endpoint catalog and pipeline operations
- `references/EXAMPLES.md`: Example prompts and request shapes

## Local Use

To make this skill available locally in a supported agent runtime, copy this folder to one of:

- `.claude/skills/alphalens-api/` (Claude Code)
- `.agents/skills/alphalens-api/` (OpenAI Codex)
- `~/.cursor/skills/alphalens-api/` (Cursor)
- `~/.openclaw/skills/alphalens-api/` (OpenClaw)

The final path must contain `SKILL.md` directly inside the `alphalens-api` directory.

## HTML Output and the Favicon Proxy

Some workflows (`market-map-org.md`, `market-map-product.md`, `investor-network.md`) generate HTML files with company logos. They start a local Node.js proxy server to avoid CORS issues when loading favicons and exporting to PDF. See the inline "Step 0 — Start the favicon proxy" in each entry-point workflow.

**Compatibility:** The proxy requires a persistent Node.js process and a browser environment. It works in:
- Claude Code desktop app (set up the proxy via `.claude/launch.json`)
- Local dev environments with a terminal

It will **not** work in web-based or ephemeral agent environments that cannot run a long-lived Node process and open a browser window.

**Fallback for restricted environments:** If the proxy is unavailable, logos fall back to coloured letter avatars and PDF export may produce blank logo placeholders. The workflow instructions include bad-favicon detection and fallback logic.

## Publish For Installer Use

If you want to install this with a repo-based installer:

```bash
npx skills add your-org/your-skills-repo
```

publish a repository that includes this `alphalens-api/` directory with `SKILL.md` inside it.

## Claude Code

For Claude Code, follow that tool's skill/plugin install flow against the published repo. The exact commands can differ by runtime version, but the skill package content in this folder is the part you need to publish.
