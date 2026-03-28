# Product Research Workflow

Use this workflow for product-centric market analysis — when the user wants to understand a company's product lines and find competitors at the product level.

---

## Step 1: Resolve anchor and fetch products

```bash
# Resolve org by domain
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/{domain}"

# Fetch products for that org
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/{org_id}/products?limit=20"
```

---

## Step 2: Score and select real products

Most companies return 5-10 products, but many are delivery channels (REST API, Chrome Extension, CRM Integrations). Apply this scoring:

```
+40  is_primary_product: true
+10  per product_reference URL that is NOT the company homepage
 +5  per distinct entry in product_target_audiences

-50  product_name contains: api, rest, extension, bulk, integration, crm, webhook, export, sdk
-30  product_categories contains 'services'
-20  ALL product_references point only to company homepage
```

**Rules:**
- Keep only products with score ≥ 20
- Maximum 3 products
- If two products share >70% overlapping audiences, merge them

---

## Step 3: Fan-out for each qualifying product — ALL IN PARALLEL

```bash
# Product similarity for each qualifying product
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/{pid1}/similar?limit=50" > /tmp/prod1.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/products/{pid2}/similar?limit=50" > /tmp/prod2.json &

# Org similarity ring 1
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{org_id}/similar?limit=50&offset=0" > /tmp/org0.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{org_id}/similar?limit=50&offset=50" > /tmp/org50.json &

# Known player domain lookups
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/known1.com" > /tmp/d1.json &

# Org similarity ring 2 (top results from ring 1)
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{top1_id}/similar?limit=50" > /tmp/r2_1.json &

wait
```

---

## Step 4: Generate HTML

Read `scripts/market-map-template.html` and populate with one tab per qualifying product.

---

## Why Product-Level?

Product similarity finds competitors that org-level misses:
- A large company may have one product that competes with your anchor
- Their org description is too broad to show similarity
- But their specific product description matches closely

---

## Common Pitfalls

- **Self result #0**: product similarity always returns the source product first — skip it
- **Low cosine distance = more similar**: sort ascending
- **Companies with multiple products**: a competitor may appear on multiple tabs — don't deduplicate
- **Products only for indexed orgs**: if AlphaLens hasn't indexed the company, product lookup won't work