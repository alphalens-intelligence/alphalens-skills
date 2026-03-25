# AlphaLens API Reference

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
- For searches driven by open-ended free text, use the search planning endpoint first.
- If the user already provides a known company or domain, do not start with free-text planning. Resolve the company and use the similar organizations endpoint.
- If the question is product-led, prefer product endpoints. Product search usually yields better precision than organization search for detailed categories, features, and use cases.
- Similarity endpoints are ID-anchored; resolve the source entity before calling them.
- Searches, reads, and enrichment can be credit- and policy-gated. Avoid repeated or unnecessary requests.

## Which Endpoint Does What

- `POST /api/v1/agent/search-params`
  turns an open-ended prompt into a structured search plan
- `GET /api/v1/entities/organizations/search-by-name/{organization_name}`
  resolves a company name into candidate organizations
- `GET /api/v1/entities/organizations/by-domain/{domain}`
  resolves a known company domain into a specific organization
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

## Search Strategy

### Use natural language planning when the market is unknown

Good examples:

- "Find vertical SaaS companies serving private equity firms"
- "Find fintech infrastructure products for expense management teams"
- "Find AI procurement startups in Europe founded after 2021"

Recommended flow:

1. Call `POST /api/v1/agent/search-params`
2. Read `entity_type`, `search_method`, and normalized filters
3. Call the matching search endpoint

### Use direct similarity when the reference company is known

Good examples:

- "Find companies similar to Ramp"
- "Show competitors to Brex"
- "Find companies like Rippling"

Recommended flow:

1. Resolve the company by domain or name
2. Call `GET /api/v1/search/organizations/{organization_id}/similar`

Do not treat these as generic free-text market searches unless resolution fails.

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

## Core Read Workflows

### 1. Turn a natural-language prompt into a search plan

Use:

`POST /api/v1/agent/search-params`

Request body:

```json
{
  "params": {
    "prompt": "Find vertical SaaS companies serving private equity firms"
  }
}
```

The response tells you:

- `entity_type`: `organization` or `product`
- `search_method`: `description`, `customer_base`, or `similar`
- `search_query`
- normalized filters

Map the result into a concrete search endpoint instead of guessing.

### 2. Resolve organizations before ID-anchored searches

Useful endpoints:

- `GET /api/v1/entities/organizations/search-by-name/{organization_name}`
- `GET /api/v1/entities/organizations/by-domain/{domain}`
- `GET /api/v1/entities/organizations/{organization_id}`

Use these when a user gives you a company name or domain and you need an `organization_id`.

This is the preferred path for lookalike requests about a named company.

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

## Collections And Pipeline Workflows

Use these for list building, enrichment, and async-ish pipeline processing.

### Safe mutation order

1. Inspect existing collections first:
   - `GET /api/v1/collections`
2. Create a collection only if needed:
   - `POST /api/v1/collections`
3. Inspect available columns:
   - `GET /api/v1/collections/columns/{collection_id}`
4. Update configured columns if needed:
   - `PATCH /api/v1/collections/columns/{collection_id}`
5. Create custom question columns when enrichment logic is required:
   - `POST /api/v1/custom-questions/`
6. Add organizations to a pipeline:
   - `POST /api/v1/pipelines/{pipeline_id}/organizations`
7. Read pipeline progress and outputs:
   - `GET /api/v1/pipelines/{pipeline_id}/items`
   - `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status`
   - `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values`

### Collection creation notes

The API exposes `POST /api/v1/collections` for creating collections. Inspect the live OpenAPI for the current request schema before generating bodies.

### Custom question notes

The API exposes `POST /api/v1/custom-questions/` for computed columns. These can be plain LLM questions, tool-enabled questions, or formulas. Inspect the live schema before creating them.

### Pipeline item readiness

When using pipelines:

- Do not assume item values are ready immediately after adding an organization or document.
- Poll the item status endpoint until `is_ready` is true before treating values as final.

## Heuristics

- If the user asks "find companies like X", resolve `X` to an organization, then use the similar organizations endpoint.
- If the user asks for a competitive landscape around a known company, use direct similarity before trying natural-language planning.
- If the user asks "find products for this market", use search planning first, then the product search endpoints.
- If the user's wording is product-led, prefer product search over organization search.
- If the user asks for a target list with enrichment, inspect collections and pipelines before creating new ones.
- If the user gives a domain, prefer by-domain resolution over fuzzy name matching.