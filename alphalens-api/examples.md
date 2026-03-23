# AlphaLens API Examples

## Example Prompts

- `Research the competitive landscape around zoominfo.com using AlphaLens.`
- `Find vertical SaaS companies serving private equity firms.`
- `Find products similar to Gong for mid-market sales teams.`
- `Build a target list of AI procurement startups and enrich it in a pipeline.`
- `Resolve Ramp by domain, find similar companies, and summarize the overlap.`

## Natural-Language Search Examples

Use natural-language planning for open-ended market discovery prompts such as:

- `Find fintech infrastructure companies in the US founded after 2019.`
- `Find AI procurement startups in Europe.`
- `Find products for finance teams that automate AP workflows.`

Start with:

```http
POST /api/v1/agent/search-params
API-Key: <ALPHALENS_API_KEY>
Content-Type: application/json
```

```json
{
  "params": {
    "prompt": "Find fintech infrastructure companies in the US founded after 2019"
  }
}
```

Then map the returned plan onto the matching organization or product search endpoint.

## Known-Company Similarity Example

If the user already gives a known company, direct similarity is better than natural-language planning.

Example prompt:

- `Find companies similar to Ramp.`

Recommended flow:

1. Resolve the company directly:

```http
GET /api/v1/entities/organizations/by-domain/ramp.com
API-Key: <ALPHALENS_API_KEY>
```

2. Use the returned `organization_id`:

```http
GET /api/v1/search/organizations/123/similar
API-Key: <ALPHALENS_API_KEY>
```

Do not start with `POST /api/v1/agent/search-params` for this kind of request unless you cannot resolve the company.

## Product-Led Search Example

Product-level search usually performs better for precise category, feature, and use-case mapping.

Example prompt:

- `Find products similar to Gong for mid-market sales teams.`

Recommended path:

```http
GET /api/v1/search/products/search?description=sales%20conversation%20intelligence%20for%20mid-market%20sales%20teams
API-Key: <ALPHALENS_API_KEY>
```

If you already have a known `product_id`, use:

```http
GET /api/v1/search/products/123/similar
API-Key: <ALPHALENS_API_KEY>
```

## Example Search Planning Request

```http
POST /api/v1/agent/search-params
API-Key: <ALPHALENS_API_KEY>
Content-Type: application/json
```

```json
{
  "params": {
    "prompt": "Find fintech infrastructure companies in the US founded after 2019"
  }
}
```

## Example Organization Search Request

```http
GET /api/v1/search/organizations/search?description=fintech%20infrastructure&year_founded_min=2020&country_keys=US
API-Key: <ALPHALENS_API_KEY>
```

## Example Similar Organization Flow

1. Resolve the source company:

```http
GET /api/v1/entities/organizations/by-domain/ramp.com
API-Key: <ALPHALENS_API_KEY>
```

2. Use the returned `organization_id`:

```http
GET /api/v1/search/organizations/123/similar
API-Key: <ALPHALENS_API_KEY>
```

## Example Pipeline Flow

1. Inspect current collections:

```http
GET /api/v1/collections
API-Key: <ALPHALENS_API_KEY>
```

2. Add an organization to a pipeline:

```http
POST /api/v1/pipelines/42/organizations
API-Key: <ALPHALENS_API_KEY>
Content-Type: application/json
```

```json
{
  "organization_id": 123
}
```

3. Poll readiness:

```http
GET /api/v1/pipelines/42/items/999/status
API-Key: <ALPHALENS_API_KEY>
```

4. Read final values:

```http
GET /api/v1/pipelines/42/items/999/values
API-Key: <ALPHALENS_API_KEY>
```
