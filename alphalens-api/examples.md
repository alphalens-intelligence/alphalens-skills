# AlphaLens API Examples

## Example Prompts

- `Research the competitive landscape around zoominfo.com using AlphaLens.`
- `Find vertical SaaS companies serving private equity firms.`
- `Find products similar to Gong for mid-market sales teams.`
- `Build a target list of AI procurement startups and enrich it in a pipeline.`
- `Resolve Ramp by domain, find similar companies, and summarize the overlap.`

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
