# Pipeline Enrichment Workflow

Use pipelines for list building, enrichment, and async processing. Read this when the user wants to build a target list, enrich companies, or manage pipeline items.

---

## Safe mutation order

Always inspect before mutating. Follow this order:

1. Inspect existing pipeline items first:
   `GET /api/v1/pipelines/{pipeline_id}/items`
2. Add organizations:
   `POST /api/v1/pipelines/{pipeline_id}/organizations`

---

## Read pipeline items

1. List pipeline items:
   `GET /api/v1/pipelines/{pipeline_id}/items`

2. Poll readiness:
   `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status`

3. Read values when ready:
   `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values`

---

## Add organizations to pipeline

```bash
POST /api/v1/pipelines/{pipeline_id}/organizations
Content-Type: application/json

{
  "organization_id": 123
}
```

---

## Pipeline item readiness

- Do not assume item values are ready immediately after adding an organization.
- Poll the item status endpoint until `is_ready` is true before treating values as final.

---

## Submit documents to pipeline

```bash
POST /api/v1/pipelines/{pipeline_id}/documents
# Or for binary data:
POST /api/v1/pipelines/{pipeline_id}/documents-binary-data
```

---

## Key Rules

- **Always inspect before mutating** — know what is already in the pipeline
- **Poll readiness** — values are computed asynchronously; do not read values until `is_ready: true`
- **Credit-gated** — pipeline endpoints may require AlphaLens credits
- **Use `is_headquarters=true`** when adding organizations by search to avoid duplicates

Collections and custom questions are managed in the [AlphaLens web interface](https://app.alphalens.ai), not via the public API.
