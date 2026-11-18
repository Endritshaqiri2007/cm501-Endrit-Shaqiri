# Web Performance Report: eKosova

**Website:** <https://ekosova.rks-gov.net/> (public homepage, no login) · **Tool:** Chrome DevTools (Network, Lighthouse, Coverage) · **Setup:** Incognito window, "Disable cache" on · **Date:** 18 November 2026

## A1 – Totals

| Metric | No throttling | Slow 4G |
|---|---|---|
| Requests | 25 | 25 |
| Transferred / resources | 598 kB / 1.6 MB | 598 kB / 1.6 MB |
| DOMContentLoaded | 411 ms | 4.49 s |
| **Load** | **433 ms** | **5.06 s** (11.7× slower) |

On a fast connection the page feels instant. On Slow 4G the same 598 kB takes about five seconds, so a few heavy files make a big difference on mobile networks (Fig. 1–2).

## A2 – The 3 largest resources

| # | File | Type | Transferred |
|---|---|---|---|
| 1 | `SamsungSans-Light.woff2` | font | 191 kB |
| 2 | `language.sq-al.js` | script | 181 kB (932 kB unzipped) |
| 3 | `SamsungSans-Bold.woff2` | font | 80 kB |

These three files are about 452 kB, roughly **75% of the whole page**. Two are fonts; the third is a full translation file loaded on the homepage, which only shows a few labels. On Slow 4G the font and the language file each took over 3.5 s (Fig. 3).

## A3 – Images

Images are not the main problem: 6 requests, 32.7 kB in total (about 5%) (Fig. 4). Four are **SVG**, a good modern choice for icons and logos. Two are **PNG**, an older format where WebP would be smaller. `rks-logo.png` is **178 × 197 px but shown at about 50 × 55 px**, so it is oversized (Fig. 5). Lighthouse estimates 21 KiB of savings from better image delivery. Also, `faviconS.ico` is 26.3 kB – more than both PNGs together, for a tiny tab icon.

## A4 – Third parties

Almost everything comes from `ekosova.rks-gov.net`. The only third-party file is `jquery.min.js` (28.1 kB) from `cdnjs.cloudflare.com` – 1 of 25 requests, confirmed with the filter `-domain:ekosova.rks-gov.net` (Fig. 6–7). There is no analytics, no advertising and no external font service. This is a clear strength.

## A5 – Render-blocking and unused code

**Render-blocking:** Lighthouse flags one file, the main stylesheet `styleComplete.mobile.css` (14.0 KiB, 190 ms, est. saving 140 ms). Nothing is painted until it arrives (Fig. 8).

**Unused code:** the Coverage tab reports **1.0 MB of 1.2 MB used (83%), 203 kB unused** (Fig. 9).

| File | Total | Unused |
|---|---|---|
| `styleComplete.css` | 73.5 kB | **86.7%** |
| `site.js` | 64.6 kB | 82.6% |
| `jquery.min.js` | 87.5 kB | 52.0% |
| `suggested-services.css`, `favorite-services.css` | 9.2 kB | **100%** |

The 83% figure is misleading. `language.sq-al.js` (932 kB) counts as fully "used" because it is one big object that runs once on load. Without it, about **three quarters** of the remaining CSS and JS is never used. The render-blocking stylesheet is itself 87% unused, so the browser waits for CSS it mostly does not need.

## A6 – Lighthouse: top 3 opportunities

Mobile, Performance only: **score 86/100**. FCP 1.0 s, TBT 50 ms, CLS 0 and Speed Index 1.6 s are all green; **LCP 4.2 s is red** (Fig. 10–12).

1. **Modern HTTP (est. 670 ms).** Every file is served over **HTTP/1.1**, so the browser queues the 24 first-party downloads. HTTP/2 or HTTP/3 would send them in parallel over one connection (Fig. 13).
2. **Render-blocking requests (est. 140 ms).** The main stylesheet blocks the first paint (see A5).
3. **LCP request discovery.** The LCP element is `div.loader`, a **full-screen loading overlay**, not real content. It also lacks `fetchpriority="high"` (Fig. 14). With HTTP/1.1 queuing, this pushes LCP to 4.2 s.

Smaller items: reduce unused CSS (11 KiB), minify CSS and JS (3 KiB each), 3 long main-thread tasks.

## A7 – Main document and 3 headers

The first request, `https://ekosova.rks-gov.net/`, returns **200 OK** with 5.4 kB transferred (Fig. 15).

- **`content-encoding: br`** – the HTML is compressed with Brotli, a modern, efficient method. That is why the page is only 5.4 kB on the wire.
- **`content-type: text/html; charset=utf-8`** – tells the browser this is HTML in UTF-8, so Albanian letters such as *ë* and *ç* display correctly.
- **`strict-transport-security: max-age=31622400; includeSubDomains; preload`** – forces HTTPS for about a year. It improves security and saves returning visitors an HTTP → HTTPS redirect.

**Missing:** there is **no `cache-control` header**. Browsers must guess how long to cache the page. A clear policy (short for the HTML, long for the versioned CSS/JS files) would speed up repeat visits.

## A8 – Conclusion

eKosova is **fast on a good connection but too slow on mobile**. The load event goes from 0.4 s to 5.1 s on Slow 4G, and LCP is 4.2 s against a 2.5 s target. For a government portal that many citizens use on their phones, the mobile result matters most.

The biggest problems are the old **HTTP/1.1** protocol, **heavy fonts and a full translation file** (about three quarters of the page weight), and a **loading overlay as the LCP element** behind a mostly unused, render-blocking stylesheet.

**Fix first, in this order:**

1. **Enable HTTP/2 or HTTP/3** on the server – a configuration change with the largest estimated saving.
2. **Slim the fonts and the language file** – subset the fonts or use fewer weights, and load only the translations the homepage needs.
3. **Remove the loader overlay and trim the CSS** – let real content be the first large paint, inline the critical CSS, and drop the two unused stylesheets.

Smaller wins: add `cache-control`, shrink the favicon, and resize `rks-logo.png` to WebP or SVG. On the positive side, the site already uses Brotli, SVG icons, HSTS and almost no third-party code.

## Appendix – Screenshots

![Fig. 1 – A1 totals, no throttling](screenshots/a1-normal.png)

![Fig. 2 – A1 totals, Slow 4G](screenshots/a1-slow4g.png)

![Fig. 3 – A2 sorted by size](screenshots/a2-largest.png)

![Fig. 4 – A3 image requests](screenshots/a3-images.png)

![Fig. 5 – A3 rks-logo.png, 178 × 197 px](screenshots/a3-rks-logo.png)

![Fig. 6 – A4 Domain column](screenshots/a4-domains.png)

![Fig. 7 – A4 only third-party request](screenshots/a4-third-party.png)

![Fig. 8 – A5 render-blocking requests](screenshots/a5-render-blocking.png)

![Fig. 9 – A5 Coverage](screenshots/a5-coverage.png)

![Fig. 10 – A6 Lighthouse score and metrics](screenshots/a6-lighthouse-score.png)

![Fig. 11 – A6 Insights](screenshots/a6-insights.png)

![Fig. 12 – A6 Diagnostics](screenshots/a6-diagnostics.png)

![Fig. 13 – A6 Modern HTTP: all files on HTTP/1.1](screenshots/a6-modern-http.png)

![Fig. 14 – A6 LCP request discovery](screenshots/a6-lcp-discovery.png)

![Fig. 15 – A7 response headers](screenshots/a7-headers.png)

