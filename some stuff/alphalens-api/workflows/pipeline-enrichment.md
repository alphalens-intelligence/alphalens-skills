# Pipeline Enrichment Workflow

Use pipelines for list building, enrichment, and async processing. Read this when the user wants to build a target list, enrich companies with custom questions, or manage collections.

---

## Safe mutation order

Always inspect before mutating. Follow this order:

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

---

## Read pipeline items

1. List pipeline items:
   `GET /api/v1/pipelines/{pipeline_id}/items`

2. Poll readiness:
   `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status`

3. Read values when ready:
   `GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values`

---

## Pipeline item readiness

- Do not assume item values are ready immediately after adding an organization.
- Poll the item status endpoint until `is_ready` is true before treating values as final.

---

## Collection creation notes

The API exposes `POST /api/v1/collections` for creating collections. Inspect the live OpenAPI for the current request schema before generating bodies.

---

## Custom question notes

The API exposes `POST /api/v1/custom-questions/` for computed columns. These can be plain LLM questions, tool-enabled questions, or formulas. Inspect the live schema before creating them.
