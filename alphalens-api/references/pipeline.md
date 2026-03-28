# Pipeline Workflow

Use pipelines for list building, enrichment, and async processing.

---

## Inspect Pipeline Items

Before adding anything, inspect existing items:

```bash
GET /api/v1/pipelines/{pipeline_id}/items
```

---

## Add Organizations to Pipeline

```bash
POST /api/v1/pipelines/{pipeline_id}/organizations
Content-Type: application/json

{
  "organization_id": 123
}
```

---

## Poll for Readiness

Do not assume values are ready immediately after adding an organization:

```bash
GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/status
```

Poll until `is_ready: true`.

---

## Read Final Values

```bash
GET /api/v1/pipelines/{pipeline_id}/items/{pipeline_item_id}/values
```

---

## Submit Documents to Pipeline

```bash
POST /api/v1/pipelines/{pipeline_id}/documents
# Or for binary data:
POST /api/v1/pipelines/{pipeline_id}/documents-binary-data
```

---

## Key Rules

- **Always inspect before mutating** — know what's already in the pipeline
- **Poll readiness** — values are computed asynchronously
- **Credit-gated** — the pipeline endpoints may require credits