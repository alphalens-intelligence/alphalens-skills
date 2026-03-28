# Bottom-Up Market Intelligence Suite

When a user asks for a "bottom-up mapping", "bottom-up analysis", "deep dive", or "full mapping" for a company, **always produce all three outputs as a linked suite**. Never stop at just the market map.

**Step 0 — Start the favicon proxy**

Google's favicon CDN (`t0.gstatic.com`) blocks cross-origin canvas reads, breaking PDF export and favicon detection. Start the proxy first:

```js
// server.js
const http = require('http'), https = require('https');
const fs = require('fs'), path = require('path'), url = require('url');
http.createServer((req, res) => {
  const { pathname } = url.parse(req.url);
  if (pathname.startsWith('/favicon/')) {
    const domain = pathname.slice('/favicon/'.length);
    const src = `https://t0.gstatic.com/faviconV2?client=SOCIAL&type=FAVICON`
              + `&fallback_opts=TYPE,SIZE,URL&url=http://${domain}&size=128`;
    https.get(src, r => {
      res.writeHead(200, { 'Content-Type': r.headers['content-type'] || 'image/png',
                           'Cache-Control': 'public, max-age=86400' });
      r.pipe(res);
    }).on('error', () => { res.writeHead(404); res.end(); });
    return;
  }
  const file = path.join(__dirname, pathname === '/' ? 'market-map.html' : pathname);
  fs.readFile(file, (err, data) => {
    if (err) { res.writeHead(404); res.end(); return; }
    const mime = { '.html':'text/html', '.js':'text/javascript' };
    res.writeHead(200, { 'Content-Type': mime[path.extname(file)] || 'application/octet-stream' });
    res.end(data);
  });
}).listen(3456);
```

In `.claude/launch.json`, set `runtimeExecutable: "node"`, `runtimeArgs: ["server.js"]`, `port: 3456`, `autoPort: false`.

**Prerequisite:** You should only be here if the Mapping Workflow Selection section in SKILL.md routed you to the bottom-up path. If the user asked for a simple "market map", use `workflows/market-map-org.md` instead.

## Critical: Bottom-up mapping always uses product-centric maps

The market map component must be **product-centric** (one tab per qualifying product). This means:
- Always fetch the anchor company's products first
- Always run the Step 0 product scoring formula
- Always use product-level similarity (`/search/products/{id}/similar`) as the primary signal source
- Org-level similarity is a supplement, not the primary driver

If you skip product scoring and collapse to a single org-level map, you are **not following the bottom-up workflow** — you are running the simpler org-level workflow and mislabeling it.

---

## Execution order

1. Read `workflows/market-map-product.md` and build the tabbed market map
3. Read `workflows/investor-network.md` and build the investor network using the companies discovered in step 2
4. Read `workflows/peer-benchmark.md` and build the peer benchmark using the closest peers from step 2

Load each workflow file when you reach that phase — not all at once.

---

## Output files

The three outputs live as separate HTML files served by the same Node.js proxy, all sharing a consistent dark top navigation bar:

| File | What it shows |
|---|---|
| `{anchor}-market-map.html` | Product-centric tabbed market map (one cluster grid per product) |
| `{anchor}-investor-network.html` | D3 force-directed graph of investors + portfolio companies |
| `{anchor}-peer-benchmark.html` | Headcount growth, funding, capital efficiency + qualitative positioning for 5 closest peers |

---

## Consistent site-wide navigation (required on all three pages)

Every page gets the **same sticky dark top nav bar** — same HTML, just the `sn-active` class moves to the current page:

```html
<nav id="site-nav">
  <a class="sn-brand" href="{anchor}-market-map.html">
    <img src="/favicon/{anchor-domain}" width="20" height="20"> {Anchor Name}
  </a>
  <div class="sn-links">
    <a class="sn-link [sn-active?]" href="{anchor}-market-map.html">Market Map</a>
    <a class="sn-link [sn-active?]" href="{anchor}-investor-network.html">Investor Network</a>
    <a class="sn-link [sn-active?]" href="{anchor}-peer-benchmark.html">Peer Benchmark</a>
  </div>
  <div class="sn-action"><!-- page-specific export button --></div>
</nav>
```

```css
#site-nav { background:#1a202c; display:flex; align-items:center; padding:0 28px; height:44px; position:sticky; top:0; z-index:200; }
.sn-brand  { color:#fff; font-size:13px; font-weight:700; margin-right:20px; text-decoration:none; display:flex; align-items:center; gap:8px; }
.sn-links  { display:flex; align-items:center; gap:2px; flex:1; }
.sn-link   { font-size:12px; font-weight:500; padding:5px 12px; border-radius:6px; color:#a0aec0; text-decoration:none; display:flex; align-items:center; gap:5px; transition:all .15s; }
.sn-link:hover   { color:#fff; background:rgba(255,255,255,.08); }
.sn-link.sn-active { color:#1a202c; background:#fff; font-weight:600; }
.sn-action { display:flex; align-items:center; gap:8px; }
.sn-export { font-size:12px; font-weight:600; padding:5px 13px; border-radius:6px; border:1px solid rgba(255,255,255,.2); background:transparent; cursor:pointer; color:#e2e8f0; }
```
