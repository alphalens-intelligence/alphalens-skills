---
name: alphalens-api
description: Use the AlphaLens API for organization and product discovery and pipeline workflows. Use when the user wants to query AlphaLens data, find similar companies or products, build target lists, enrich pipeline items, or integrate against the AlphaLens API. Requires an AlphaLens subscription with API access.
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

**Note:** This skill requires an active [AlphaLens subscription](https://alphalens.ai) with API access. Contact sales@alphalens.ai for enterprise pricing.

## Authentication

- Expect `ALPHALENS_API_KEY` to be available.
- Send `API-Key: <value>` on requests.

## Base URLs

- Production: `https://api-production.alphalens.ai`
- Public OpenAPI: `https://api-production.alphalens.ai/openapi.json`
- Public docs: `https://api-production.alphalens.ai/docs`

## Working Rules

- Treat the live OpenAPI and endpoint responses as the source of truth.
- If the user already gives a known company, resolve the company first, then use the similar organizations endpoint.
- If the user already gives a known domain, prefer by-domain resolution over free-text search.
- Product-level search usually performs better than organization-level search for precise market/category matching. Prefer product search when the user is asking about products, use cases, features, or competitive product landscapes.
- Resolve company names or domains before ID-anchored similarity searches.
- Be careful with policy-, quota-, and credit-gated endpoints. Avoid broad fan-out without a clear reason.
- **Always set `is_headquarters=true` on organization and product search endpoints.** This filters to headquarters locations only and returns much better results. Only omit this filter if the user explicitly asks for all locations including branch offices.

## Choose The Right Search Path

- Known company or domain:
  resolve the organization, then call `GET /api/v1/search/organizations/{organization_id}/similar`.
- Known product or product-led market thesis:
  prefer the product search endpoints, especially `GET /api/v1/search/products/search` or `GET /api/v1/search/products/{product_id}/similar`.
- Free-text market discovery:
  use `GET /api/v1/search/organizations/search` or `GET /api/v1/search/products/search`.
- Do not turn "Find companies similar to Ramp" into a free-text search if you can resolve Ramp directly.

## Comprehensive Fan-Out Strategy

For thorough market research, use this 4-ring fan-out strategy. **Fire ALL API calls in parallel** — never sequentially. Use `limit=50` and paginate with `offset` when needed.

### The 4 Rings

| Ring | Method | What it surfaces |
|------|--------|------------------|
| 1 — Product similarity | `/search/products/{id}/similar` | Niche product lines inside large orgs; companies that org-level misses |
| 2 — Org similarity ring 1 | `/search/organizations/{id}/similar?limit=50` | Direct org-level competitors; the obvious players |
| 3 — Known player sweep | `by-domain` for domains you know | Established names that may be too large/small for similarity (incumbents, emerging startups) |
| 4 — Org similarity ring 2 | `/search/organizations/{ring2_id}/similar` | Niche players only reachable by pivoting through ring-1 results |

### How to Execute

Fire all four rings simultaneously in a single parallel block:

```bash
API="https://api-production.alphalens.ai"
KEY="${ALPHALENS_API_KEY}"

# Ring 1: product-level similarity (fire for each qualifying product)
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/{pid1}/similar?limit=50" > /tmp/prod1.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/{pid2}/similar?limit=50" > /tmp/prod2.json &

# Ring 2: org-level similarity from anchor (use offset=0 AND offset=50)
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{anchor_id}/similar?limit=50&offset=0"  > /tmp/org0.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{anchor_id}/similar?limit=50&offset=50" > /tmp/org50.json &

# Ring 3: domain lookups for known players (your own knowledge, 20-40 domains)
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/known1.com" > /tmp/d1.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/known2.com" > /tmp/d2.json &

# Ring 4: second-ring org similarity (top 5-8 orgs from Ring 2)
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{ring2_top1_id}/similar?limit=50" > /tmp/r2_1.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{ring2_top2_id}/similar?limit=50" > /tmp/r2_2.json &

wait   # collect all results
```

### Key Rules

- **Never make AlphaLens API calls sequentially** — parallel calls turn a 60-second loop into a 2-3 second sweep.
- **Use `limit=50`** — the default of 24 misses too much.
- **Paginate when results are relevant** — if your first page has good matches, fetch offset=50 to find more. Don't paginate if the first page already饱和了你的需求.
- **Supplement with your own knowledge** — add companies you know should be there and verify via `by-domain`.

## Default Workflow

### Research companies similar to a known company

1. If the user gives a domain, call `GET /api/v1/entities/organizations/by-domain/{domain}`.
2. Use the resolved `organization_id` with `GET /api/v1/search/organizations/{organization_id}/similar`.
3. Use detail endpoints only for shortlisted results.

### Enrich organization data

Once you have an `organization_id`, you can fetch additional data:

- `GET /api/v1/entities/organizations/{organization_id}/products` - List the company's products
- `GET /api/v1/entities/organizations/{organization_id}/growth-metrics` - Get headcount, web traffic, LinkedIn followers, job openings
- `GET /api/v1/entities/organizations/{organization_id}/funding` - Get funding rounds and investors
- `GET /api/v1/entities/organizations/{organization_id}/people` - Get founders and leadership
- `GET /api/v1/entities/organizations/{organization_id}/addresses` - Get locations (HQ and branches)

### Research products from a prompt

1. Prefer product search if the user's intent is product-led, even if the end goal is a company list.
2. Use the product search endpoints:
   - `GET /api/v1/search/products/search`
   - `GET /api/v1/search/products/search-customers`
   - `GET /api/v1/search/products/{product_id}/similar`
3. Fetch product details only when needed.

### Build or enrich a pipeline

1. Use `GET /api/v1/pipelines/{pipeline_id}/items` to inspect existing pipeline items.
2. Add organizations to a pipeline with `POST /api/v1/pipelines/{pipeline_id}/organizations`.
3. Poll pipeline item status with `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status`.
4. Read final values with `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values`.

## Endpoint Priorities

- Use direct resolution plus similarity for known companies.
- Use product search first when the question is really about products, categories, features, or use cases.
- Use public search endpoints for discovery.
- Use entity endpoints for detail fetches.
- Use pipeline endpoints for enrichment workflows.

## References

- For endpoint mapping and workflow details, see [references/REFERENCE.md](references/REFERENCE.md).
- For example prompts and request shapes, see [references/EXAMPLES.md](references/EXAMPLES.md).