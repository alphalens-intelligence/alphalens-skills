# AlphaLens API Reference

**Note:** This skill requires an active [AlphaLens subscription](https://alphalens.ai) with API access.

## Authentication

Default to API key authentication:

```http
API-Key: <ALPHALENS_API_KEY>
```

Bearer auth exists in the API, but API key auth is the preferred default for agent workflows.

## Base URLs

- Production: `https://api-production.alphalens.ai`
- Public docs: `https://api-production.alphalens.ai/docs`
- Public OpenAPI: `https://api-production.alphalens.ai/openapi.json`

## General Guidance

- Start with the public OpenAPI when you need contract details.
- Prefer narrow reads before writes.
- If the user already provides a known company or domain, resolve the company and use the similar organizations endpoint.
- If the question is product-led, prefer product endpoints. Product search usually yields better precision than organization search for detailed categories, features, and use cases.
- Similarity endpoints are ID-anchored; resolve the source entity before calling them.
- Searches, reads, and enrichment can be credit- and policy-gated. Avoid repeated or unnecessary requests.

## Which Endpoint Does What

### Organization Resolution
- `GET /api/v1/entities/organizations/by-domain/{domain}`
  resolves a known company domain into a specific organization (returns `organization_id`)
- `GET /api/v1/entities/organizations/{organization_id}`
  fetches full organization details

### Organization Enrichment (requires organization_id)
- `GET /api/v1/entities/organizations/{organization_id}/products`
  lists all products for the organization
- `GET /api/v1/entities/organizations/{organization_id}/funding`
  returns funding rounds and investor details
- `GET /api/v1/entities/organizations/{organization_id}/growth-metrics`
  returns headcount, web traffic, LinkedIn followers, job openings time series
- `GET /api/v1/entities/organizations/{organization_id}/people`
  returns founders and leadership
- `GET /api/v1/entities/organizations/{organization_id}/addresses`
  returns headquarters and branch office locations

### Search
- `GET /api/v1/search/organizations/{organization_id}/similar`
  finds organizations similar to a known reference organization
- `GET /api/v1/search/organizations/search`
  performs free-text organization discovery by description
- `GET /api/v1/search/organizations/search-customers`
  performs organization discovery by customer-base description
- `GET /api/v1/search/products/search`
  performs free-text product discovery by product description
- `GET /api/v1/search/products/search-customers`
  performs product discovery by customer-base or target-user description
- `GET /api/v1/search/products/{product_id}/similar`
  finds products similar to a known reference product

### Pipeline
- `GET /api/v1/pipelines/{pipeline_id}/organizations`
  adds an organization to a pipeline
- `GET /api/v1/pipelines/{pipeline_id}/items`
  lists pipeline items with values
- `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status`
  checks if pipeline item values are ready
- `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values`
  reads final pipeline item values

## Search Strategy

### Use direct similarity when the reference company is known

Good examples:

- "Find companies similar to Ramp"
- "Show competitors to Brex"
- "Find companies like Rippling"

Recommended flow:

1. Resolve the company by domain
2. Call `GET /api/v1/search/organizations/{organization_id}/similar`

### Prefer product search for precise market mapping

Product search is usually the better choice when the user is really asking about:

- product categories
- feature sets
- workflows
- target users
- competitive products

Examples:

- "Find products similar to Gong for mid-market sales teams"
- "Find AP automation products for finance teams"
- "Find developer tools for LLM observability"

If the end goal is still a company list, you can search products first and then roll results up to their organizations.

## Comprehensive Fan-Out Strategy

For thorough market research, use this 4-ring fan-out. **Fire all API calls in parallel** — never sequentially.

### The 4 Rings

| Ring | Method | What it surfaces |
|------|--------|------------------|
| 1 — Product similarity | `/search/products/{id}/similar` | Niche product lines inside large orgs; companies that org-level misses |
| 2 — Org similarity ring 1 | `/search/organizations/{id}/similar?limit=50` | Direct org-level competitors; the obvious players |
| 3 — Known player sweep | `by-domain` for known domains | Established names too large/small for similarity |
| 4 — Org similarity ring 2 | `/search/organizations/{ring2_id}/similar` | Niche players only reachable through ring-1 results |

### Execution Pattern

```bash
API="https://api-production.alphalens.ai"
KEY="${ALPHALENS_API_KEY}"

# Ring 1: product-level similarity
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/{pid}/similar?limit=50" > /tmp/prod.json &

# Ring 2: org similarity (paginate offset=0 AND offset=50)
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{id}/similar?limit=50&offset=0" > /tmp/org0.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{id}/similar?limit=50&offset=50" > /tmp/org50.json &

# Ring 3: known player domain lookups
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/known1.com" > /tmp/d1.json &

# Ring 4: second-ring org similarity
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{ring2_id}/similar?limit=50" > /tmp/r2.json &

wait
```

### Key Rules

- **Never call AlphaLens APIs sequentially** — use parallel curls with `&` and `wait`
- **Use `limit=50`** — default of 24 misses too much
- **Paginate when results are relevant** — if your first page has good matches, fetch offset=50 to find more
- **Supplement with your own knowledge** of the market

## Core Read Workflows

### 1. Resolve organizations before ID-anchored searches

Useful endpoints:

- `GET /api/v1/entities/organizations/by-domain/{domain}`
- `GET /api/v1/entities/organizations/{organization_id}`

Use these when a user gives you a domain and you need an `organization_id`.

### 2. Enrich organization data

Once you have an `organization_id`, fetch additional data:

- `GET /api/v1/entities/organizations/{organization_id}/products` - List the company's products
- `GET /api/v1/entities/organizations/{organization_id}/funding` - Funding rounds and investors
- `GET /api/v1/entities/organizations/{organization_id}/growth-metrics` - Headcount, web traffic, LinkedIn followers, job openings
- `GET /api/v1/entities/organizations/{organization_id}/people` - Founders and leadership
- `GET /api/v1/entities/organizations/{organization_id}/addresses` - HQ and branch locations

### 3. Resolve products or fetch product detail

Useful endpoints:

- `GET /api/v1/entities/products/{product_id}`
- `GET /api/v1/entities/products/by-domain/{domain}`

Use these when you need a product detail fetch or product list for a known company domain.

## Search Endpoints

All search endpoints support filters for location, company age/size, product categories, and funding. See `EXAMPLES.md` for the full filter reference.

### Organization discovery

- Description search:
  `GET /api/v1/search/organizations/search`
- Customer-base search:
  `GET /api/v1/search/organizations/search-customers`
- Similar organizations:
  `GET /api/v1/search/organizations/{organization_id}/similar`
- Location suggestions:
  `GET /api/v1/search/locations/suggest`
- Location metadata:
  `GET /api/v1/search/locations/metadata`

### Product discovery

- Description search:
  `GET /api/v1/search/products/search`
- Customer-base search:
  `GET /api/v1/search/products/search-customers`
- Similar products:
  `GET /api/v1/search/products/{product_id}/similar`

## Pipeline Workflows

Use pipelines for list building, enrichment, and async processing.

### Add organizations to a pipeline

1. Use `POST /api/v1/pipelines/{pipeline_id}/organizations` to add an organization:

```http
POST /api/v1/pipelines/{pipeline_id}/organizations
API-Key: <ALPHALENS_API_KEY>
Content-Type: application/json

{
  "organization_id": 123
}
```

### Read pipeline items

1. List pipeline items:
   `GET /api/v1/pipelines/{pipeline_id}/items`

2. Poll readiness:
   `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status`

3. Read values when ready:
   `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values`

### Pipeline item readiness

- Do not assume item values are ready immediately after adding an organization.
- Poll the item status endpoint until `is_ready` is true before treating values as final.

## Heuristics

- If the user asks "find companies like X", resolve `X` to an organization, then use the similar organizations endpoint.
- If the user asks for a competitive landscape around a known company, use direct similarity.
- If the user asks "find products for this market", use the product search endpoints.
- If the user's wording is product-led, prefer product search over organization search.
- If the user gives a domain, prefer by-domain resolution over fuzzy name matching.