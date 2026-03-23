---
name: alphalens-api
description: Use the AlphaLens API for organization and product discovery, search planning, collections, and pipeline workflows. Use when the user wants to query AlphaLens data, find similar companies or products, build target lists, enrich pipeline items, or integrate against the AlphaLens API.
---

# AlphaLens API

## Authentication

- Prefer API key auth.
- Expect `ALPHALENS_API_KEY` to be available.
- Send `API-Key: <value>` on requests.
- Bearer auth also exists, but use API keys unless the user explicitly asks for bearer tokens.

## Base URLs

- Production: `https://api-production.alphalens.ai`
- Public OpenAPI: `https://api-production.alphalens.ai/openapi.json`
- Public docs: `https://api-production.alphalens.ai/docs`

## Working Rules

- Treat the live OpenAPI and endpoint responses as the source of truth.
- Prefer `POST /api/v1/agent/search-params` for natural-language research prompts.
- Resolve company names or domains before ID-anchored similarity searches.
- Before mutating collections or pipelines, inspect current state first.
- Be careful with policy-, quota-, and credit-gated endpoints. Avoid broad fan-out without a clear reason.

## Default Workflow

### Research companies from a prompt

1. Call `POST /api/v1/agent/search-params` with `{"params": {"prompt": "..."}}`.
2. Use the returned plan to choose one of:
   - `GET /api/v1/search/organizations/search`
   - `GET /api/v1/search/organizations/search-customers`
   - `GET /api/v1/search/organizations/{organization_id}/similar`
3. Fetch entity details only when needed.

### Research products from a prompt

1. Call `POST /api/v1/agent/search-params`.
2. Use the returned plan to choose one of:
   - `GET /api/v1/search/products/search`
   - `GET /api/v1/search/products/search-customers`
   - `GET /api/v1/search/products/{product_id}/similar`
3. Fetch product details only when needed.

### Build or enrich a pipeline

1. Inspect existing collections or pipelines before creating or updating anything.
2. Create or reuse the target collection.
3. Inspect or update collection columns.
4. Create custom questions if enrichment columns are needed.
5. Add organizations or documents to the pipeline.
6. Poll pipeline item status before reading final values.

## Endpoint Priorities

- Use `agent/search-params` first for natural-language intent.
- Use public search endpoints for discovery.
- Use entity endpoints for detail fetches.
- Use collection, custom-question, and pipeline endpoints for enrichment workflows.

## References

- For endpoint mapping and workflow details, read [reference.md](reference.md).
- For example prompts and request shapes, read [examples.md](examples.md).
