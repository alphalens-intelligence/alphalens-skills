# Market Map Workflow

Use this workflow when the user wants a competitive landscape or market map for a company.

**Produces:** A single-page HTML grid showing similar organizations clustered by category.

**Prerequisite:** Read `scripts/favicon-proxy.md` and start the proxy server before rendering.

---

## Step 1: Resolve the anchor company

```bash
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/{domain}"
```

Get the `organization_id`, `active_domain`, and `logo_url` from the response.

---

## Step 2: Fan-out search — ALL CALLS IN PARALLEL

Never make AlphaLens API calls sequentially. Fire all calls simultaneously:

```bash
API="https://api-production.alphalens.ai"
KEY="${ALPHALENS_API_KEY}"

# Primary: org similarity from anchor
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{anchor_id}/similar?limit=50" > /tmp/r_anchor.json &

# Secondary: domain lookups for known competitors (your knowledge)
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/competitor1.com" > /tmp/r_c1.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/entities/organizations/by-domain/competitor2.com" > /tmp/r_c2.json &

# Additional rings: top results from anchor similarity
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{top1_id}/similar?limit=50" > /tmp/r2_1.json &
curl -s -H "API-Key: $KEY" "$API/api/v1/search/organizations/{top2_id}/similar?limit=50" > /tmp/r2_2.json &

wait
```

---

## Step 3: Supplement with your own knowledge

After the API fan-out, add companies you know should be there:

1. For each known company, verify via `GET /api/v1/entities/organizations/by-domain/{domain}`
2. If not found in AlphaLens, the lookup triggers indexing — mark as `.pending`
3. If `venture_organization_index_status` is `indexing_failed`, the company cannot be mapped

---

## Step 4: Cluster the results

Group companies by shared characteristics:
- Business model
- Buyer persona
- Use case
- Go-to-market

Aim for 5-10 clusters. Never use "Other" — name clusters by buyer outcome (e.g., "AI Due Diligence Platforms" not "AI Tools").

Do not show approximate company counts in cluster subtitles.

---

## Step 5: Render the HTML market map

Read `scripts/market-map-template.html` and populate it with clustered results.

**Design rules:**
- **Light mode only** — `background: #f7f8fc`, dark text
- **No emojis** in tab labels, cluster titles, section headers, or button text
- CSS 3-column grid. Each cluster: distinct coloured border, **white background (`#fff`)**. Good border palette: red/amber/yellow/blue/green/purple/teal/grey/pink.
- Each company: `<a class="company" href="https://app.alphalens.ai/discover/organization/{active_domain}" target="_blank">` wrapping a 44x44px white logo tile + name below
- **Anchor company**: gold border + glow (`border: 2px solid #d69e2e; box-shadow: 0 0 10px rgba(214,158,46,0.3)`)
- **Indexed companies**: full opacity, solid border
- **Indexing/pending companies**: `opacity: 0.65`, dashed border, italic name
- **indexing_failed companies**: `opacity: 0.5`, dashed grey border, `?` suffix on name
- Legend at bottom. No company counts in subtitles
- "Click any logo to open its AlphaLens profile" in the subtitle

---

## Step 6: PDF export

Use `html2pdf.js` (CDN) with these settings for a clean A2 landscape export:

```js
html2pdf().set({
  margin: 12,
  filename: 'market-map.pdf',
  image: { type: 'jpeg', quality: 0.98 },
  html2canvas: {
    scale: 2, useCORS: false, allowTaint: false,
    backgroundColor: '#f7f8fc',
    windowWidth: Math.max(document.documentElement.scrollWidth, 1560),
    scrollX: 0, scrollY: 0
  },
  jsPDF: { unit: 'mm', format: 'a2', orientation: 'landscape' },
  pagebreak: { mode: 'avoid-all' }
}).from(document.body).save();
```

Set `visibility: hidden` (not `display: none`) on the export button before rendering. A2 landscape gives enough room for a 3-column layout without clipping.

---

## Common Pitfalls

- **Domain mismatch**: AlphaLens `active_domain` may differ from public domain. Use AlphaLens domain for profile links; real domain for favicons.
- **Duplicate entries**: checking `robin.ai` when `robinai.com` already exists. Check by name if domain lookup returns unexpected ID.
- **Pagination**: missing players is usually offset=0 limitation. Use `limit=50`.