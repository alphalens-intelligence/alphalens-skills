# Favicon Proxy Server & Bad Favicon Detection

This is shared infrastructure required by all market map workflows (org-level and product-centric) and the investor network graph. Set it up before rendering any HTML output.

## Why this is needed

Google's favicon CDN (`t0.gstatic.com`) blocks cross-origin canvas reads. Without a same-origin proxy:
- PDF export via `html2pdf.js` renders blank squares instead of logos
- Canvas fingerprinting for bad favicon detection fails silently

## Step 1 — Start the local favicon proxy server

```js
// server.js — serves market-map.html + proxies favicons
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

In the HTML, rewrite all favicon `<img>` src attributes to `/favicon/{domain}` on `DOMContentLoaded` — never embed the Google CDN URL directly.

## Step 2 — Detect and replace bad favicons (canvas fingerprinting)

Google returns a generic globe icon when a domain has no favicon. Detect and replace it with a coloured letter avatar:

```js
// On DOMContentLoaded, after rewriting img srcs to /favicon/<domain>:
const badHash = await loadHash('/favicon/xyznotarealdomain99999abc.invalid');

document.querySelectorAll('.logo-wrap img').forEach(img => {
  const check = () => { if (hashImg(img) === badHash) replaceWithInitial(img); };
  img.complete ? check() : img.addEventListener('load', check);
});

function hashImg(imgEl) {
  const c = document.createElement('canvas'); c.width = c.height = 16;
  const ctx = c.getContext('2d'); ctx.drawImage(imgEl, 0, 0, 16, 16);
  return ctx.getImageData(0,0,16,16).data.reduce((a,v) => a+v, 0);
}
function loadHash(url) {
  return new Promise(r => { const i = new Image();
    i.onload = () => r(hashImg(i)); i.onerror = () => r(null); i.src = url; });
}
function replaceWithInitial(img) {
  const name = img.closest('a.company')?.querySelector('.company-name')?.textContent?.trim() || '?';
  const hue = [...name].reduce((a,c) => a + c.charCodeAt(0), 0) % 360;
  img.closest('.logo-wrap').innerHTML =
    `<div style="width:32px;height:32px;border-radius:8px;background:hsl(${hue},52%,56%);
     display:flex;align-items:center;justify-content:center;color:#fff;font-weight:700;
     font-size:14px;font-family:-apple-system,sans-serif;">${name[0].toUpperCase()}</div>`;
}
```

This works because images served through the local proxy are same-origin — canvas reads are allowed.
