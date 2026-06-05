---
title: "finda.cloud — Tech Stack & SEO Audit"
date: 2026-06-05
---

# finda.cloud — Tech Stack & SEO Audit

**Site:** https://finda.cloud/
**Company:** Finda Cloud Technology Limited — APAC cloud / SaaS value-added distributor
**Audited:** 2026-06-05 (external black-box analysis; no server access)

---

## Executive summary

The marketing site is a **client-rendered React single-page app on Firebase Hosting** with an
**excellent `<head>` meta layer** but **almost no crawlable content**. Search engines that don't
execute JavaScript see a near-empty page. On top of that, three issues actively work against ranking:

1. **Soft-404s everywhere** — every URL (including random paths and old WordPress URLs) returns
   HTTP `200` with the empty app shell instead of a real `404`.
2. **`robots.txt` and `sitemap.xml` are served unreliably** — most requests return the app shell, not
   the real files. When the sitemap does appear it lists only ~4 low-value URLs (including `/404` and a
   Google-verification token).
3. **The old WordPress blog/news is gone** — partnership announcements that Google indexed (Tencent
   Cloud, Zoom) now resolve to the empty SPA. That's lost content, lost keywords, and lost link equity.

The meta tags, social cards, HTTPS, CDN and analytics are all done well. The gap is **rendering +
content**, not metadata. This is a content + technical-SEO problem more than a "tags" problem.

---

## 1. Technology stack

### Frontend
| Layer | Detail |
|---|---|
| Framework | **React 18** (`createRoot`, `react-dom`) — SPA, rendered fully in the browser |
| Build tool | **Vite** (hashed `/assets/index-*.js` + `.css`, `modulepreload` injection) |
| Routing | **React Router** (client-side) |
| UI libs | **lucide-react** icons (93 refs), **Framer Motion** animation |
| Bundle | Single, **not code-split**, ~1.04 MB uncompressed (served Brotli-compressed) |

### Hosting / infra
| Layer | Detail |
|---|---|
| Host | **Firebase Hosting** (Google) — TXT `hosting-site=findacloud-website`, anchor IP `199.36.158.100` |
| CDN | **Fastly** edge (`x-served-by: cache-nrt-…`, Tokyo PoP) |
| Protocols | HTTP/2 **and** HTTP/3 (`alt-svc: h3`), Brotli compression |
| TLS | Google Trust Services managed cert (Firebase). *Note:* served leaf CN shows another tenant domain — normal for Firebase shared multi-SAN certs, but worth confirming the cert lists `finda.cloud`. |
| DNS | **SiteGround** name servers (legacy WordPress host still controls DNS); registrar **GoDaddy**; WHOIS privacy proxy; current registration 2023-02-25, expires 2027 |
| Email | **Google Workspace** (MX `aspmx.l.google.com`); SPF includes `sendersrv.com` + `dnssmarthost.net` (email-marketing / transactional sending) |

### Third-party services
- **Intercom "Fin AI"** support chat widget (`app_id: iwd6wqhz`)
- **Google Analytics** + **Microsoft Clarity** (behaviour analytics / heatmaps)
- **Google Search Console** verified (TXT `google-site-verification=…`)
- **Zoom** domain verification TXT (`ZOOM_verify_…`) — distributor relationship

---

## 2. Technical SEO

### ✅ Strong
- Keyword-rich, well-sized `<title>` and `<meta description>`, region-targeted (HK / SG / APAC).
- Complete **Open Graph** + **Twitter Card** tags; OG image valid (`200`, 1220×640, with `og:image:alt`).
- **Canonical** tag present, `lang="en"`, responsive viewport.
- HTTPS everywhere, HSTS, HTTP→HTTPS 301, fast TTFB (~0.21 s cold), HTTP/3, Brotli.
- Google Search Console connected.

### 🔴 Critical
| Issue | Evidence | Impact |
|---|---|---|
| **Content rendered only in JS** | Raw HTML body is `<div id="root"></div>` — no `<h1>`, `<nav>`, `<main>`, `<article>`, no `<noscript>` fallback | Non-JS crawlers (and social / LinkedIn / many AI crawlers) see an empty page; Google renders JS only on a deferred second pass |
| **Soft-404s on every route** | Random paths, `/wp-login.php`, `/wp-json/`, old blog URLs all return **HTTP 200** + app shell | Google flags soft-404s; crawl budget wasted; dead URLs look "alive" |
| **`robots.txt` / `sitemap.xml` unreliable** | 3/3 repeat hits returned the SPA shell, not the files | Crawlers frequently get no valid robots directives and no URL list |
| **Thin / broken sitemap** | When present, ~4 URLs only — includes `/404` and a `google…` verification token | Almost nothing for Google to discover; junk entries |
| **Legacy blog lost** | Indexed WordPress posts (`/category/news/`, partnership posts) now return the empty SPA | Lost indexed content, keywords, and inbound link equity |
| **No structured data** | No JSON-LD anywhere | No `Organization` / `Product` rich results; weaker entity recognition |

### 🟡 Secondary
- **No `hreflang`** despite multi-region targeting; site is **English-only** (no Traditional Chinese for the primary HK/Macau market).
- **Missing security headers** — only HSTS present; no CSP, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`.
- **Obsolete `<meta keywords>`** — ignored by all major engines (harmless, dated).
- **1 MB single bundle, no code-splitting** — hurts Core Web Vitals (LCP/TBT), a mobile ranking factor on APAC networks.

---

## 3. Content SEO

The site is **content-thin**: roughly four real routes and no crawlable body text. There is currently
**no live blog or news section** — the WordPress content that previously carried keyword-rich
announcements (Tencent Cloud authorized distributor, Zoom distributor, Freshworks) was dropped in the
migration to the React SPA and now soft-404s.

Consequences:
- The pages targeting valuable commercial terms ("Tencent Cloud reseller Hong Kong", "Zoom
  distributor HK", "SaaS distributor APAC") have **little indexable text** to rank on.
- **No localized Chinese content** for HK/Macau, even though the brand's partnerships and press
  coverage are bilingual and the local audience searches in Traditional Chinese.
- Brand/entity signals are weak without `Organization` schema, an "About/Contact" with `LocalBusiness`
  data, or a content hub that earns links.

---

## 4. "Do we need a blog for more keyword coverage?" — Yes

**Recommended.** You effectively *had* one and lost it in the migration. A blog/insights hub is the
cheapest, highest-leverage way to rebuild keyword coverage and topical authority:

- Target the commercial + informational terms around each partnership: Tencent Cloud, Zoom, Freshworks,
  XaaS/SaaS distribution, cloud migration in HK/SG.
- Re-publish or 301-redirect the **old indexed news/partnership posts** so you recover that equity
  instead of soft-404ing it.
- Publish in **English and Traditional Chinese** to match how the HK/Macau market searches.
- Each post is an indexable page with real text — directly addressing the "empty SPA" problem for at
  least part of the site.

Cadence matters less than consistency; even 1–2 quality posts/month rebuilds a content footprint.

---

## 5. Target region

Evidence points to **Hong Kong + Macau + Singapore, broader APAC**:
- Meta copy: "HK, SG & APAC"; Tencent Cloud **港澳 (HK/Macau)** authorized distributor; Zoom **Hong
  Kong** distributor; office in **Kwun Tong, Kowloon, Hong Kong**.

Implications:
- Add **`hreflang`** for `en` and `zh-Hant` (and consider `zh-HK`).
- Publish **Traditional Chinese** versions of key pages and blog posts.
- Add **`LocalBusiness` / `Organization` JSON-LD** with the HK address; set up / verify Google Business
  Profile for local-pack visibility.
- Confirm Search Console **international targeting** and that the sitemap covers both language variants.

---

## 6. Prioritized recommendations

1. **Fix rendering** (highest ROI) — add SSR / SSG / prerendering so crawlers get real HTML.
   Options: migrate to Next.js/Astro, or add a prerender step to the existing Vite build
   (`vite-plugin-prerender`, `react-snap`) or a Firebase + prerender.io / Rendertron path.
2. **Return real `404`s** for unknown routes; stop serving `200` + app shell for everything.
3. **Ship reliable, static `robots.txt` and a real `sitemap.xml`** (clean URLs only — no `/404`, no
   verification tokens); submit in Search Console.
4. **Recover the old blog/news** — 301-redirect or re-publish the indexed WordPress posts; stand up a
   bilingual blog going forward.
5. **Add JSON-LD** — `Organization` + `LocalBusiness` (HK office), and `Product`/`Service` for offerings.
6. **Add `hreflang` + Traditional Chinese content** for HK/Macau.
7. **Add security headers** (CSP, `X-Content-Type-Options: nosniff`, `Referrer-Policy`,
   `Permissions-Policy`, `X-Frame-Options`).
8. **Code-split the bundle** to improve Core Web Vitals; drop the obsolete keywords meta.

---

## 7. Open questions for the website manager

These can't be determined from outside the site — please confirm:

1. **Rendering plan:** Is the SPA intended to stay client-rendered, or is SSR/prerendering on the
   roadmap? (Determines how we fix the "empty HTML" problem.)
2. **Old WordPress content:** What happened to the previous blog/news? Is the content archived anywhere
   so we can 301-redirect or re-publish it? Are the old URLs expected to stay live?
3. **robots.txt / sitemap.xml:** Are these meant to be deployed as static files on Firebase? They're
   currently returning the app shell on most requests — is that a known deploy/cache config issue?
4. **404 handling:** Is the catch-all `200` + app shell intentional, or should unknown routes return a
   real 404?
5. **Languages:** Is a Traditional Chinese (zh-Hant) version planned for the HK/Macau audience, or is
   English-only deliberate?
6. **Blog ownership:** Who would own a relaunched blog (in-house, agency)? What cadence is realistic?
7. **Search Console:** Can we get access to Search Console + GA4 to confirm current indexing, soft-404
   reports, and which queries already drive traffic?
8. **Analytics IDs:** Please confirm the GA4 / Microsoft Clarity property IDs so we can validate
   tracking is firing correctly.
9. **TLS cert:** Please confirm the production certificate's SAN list includes `finda.cloud`
   (the externally observed leaf CN belonged to another Firebase tenant).
10. **www / domain:** Should `www.finda.cloud` resolve and redirect to the apex? (It currently doesn't
    respond.)

---

*Black-box audit — findings are based on externally observable signals (HTTP headers, HTML/JS bundle,
DNS/WHOIS, public search results). Server-side configuration should be confirmed with the team.*
