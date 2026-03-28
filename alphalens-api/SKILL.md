---
name: alphalens-api
description: >-
  Use this skill whenever the user wants to discover companies, search for products,
  build market maps, manage collections, or run pipeline workflows using AlphaLens.
  Triggers include: any mention of 'AlphaLens', 'market map', 'competitive landscape',
  'company discovery', 'product search', 'similar companies', 'find companies like',
  'company enrichment', 'target list', or requests to find, enrich, or visualize
  company/product data. Also use when the user asks to build a market landscape,
  research competitors, or automate prospecting workflows.
metadata:
  {
    "openclaw":
      {
        "requires": { "env": ["ALPHALENS_API_KEY"] },
        "primaryEnv": "ALPHALENS_API_KEY",
        "homepage": "https://alphalens.ai"
      }
  }
---

# AlphaLens API

**Note:** This skill requires an active [AlphaLens subscription](https://alphalens.ai) with API access.

## Quick Reference

| Task | Guide |
|------|-------|
| Find similar companies | Use `by-domain` + `/similar` — see below |
| Search products | Use `/search/products` — see below |
| Market / competitive map | Read [references/market-map.md](references/market-map.md) |
| Product research | Read [references/product-research.md](references/product-research.md) |
| Pipeline workflows | Read [references/pipeline.md](references/pipeline.md) |
| All endpoints | See [references/REFERENCE.md](references/REFERENCE.md) |

## Authentication

```bash
API="https://api-production.alphalens.ai"
KEY="${ALPHALENS_API_KEY}"
```

Send `API-Key: $KEY` on all requests.

## Core Patterns

### Find companies similar to a known company

```bash
# 1. Resolve by domain
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/ramp.com"

# 2. Use organization_id for similarity
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{id}/similar?limit=50&is_headquarters=true"
```

### Search products

```bash
# Free-text product search
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/search?description=AI%20procurement&is_headquarters=true"

# Similar products
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/{product_id}/similar?limit=50&is_headquarters=true"
```

### Enrich organization data

```bash
# Products, funding, growth metrics, people, addresses
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/{id}/products"
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/{id}/funding"
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/{id}/growth-metrics"
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/{id}/people"
```

## Critical Rules

- **Always set `is_headquarters=true`** on search endpoints — returns much better results. Only omit if the user asks for all locations.
- **Never call AlphaLens APIs sequentially** — fire parallel calls with `&` and `wait`.
- **Use `limit=50`** — the default of 24 misses too much.
- **Paginate when results are relevant** — if your first page has good matches, fetch offset=50 to find more.
- **Resolve by domain first** — never guess an organization_id.
- **Poll pipeline readiness** — values are computed asynchronously. Check `is_ready` before reading values.
- **Credit-gated endpoints** — some endpoints require credits. Avoid unnecessary fan-out.

## References

- [references/REFERENCE.md](references/REFERENCE.md) — full endpoint reference
- [references/EXAMPLES.md](references/EXAMPLES.md) — example prompts and request shapes