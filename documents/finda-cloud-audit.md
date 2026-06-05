---
title: "finda.cloud — Tech Stack & SEO Audit"
date: 2026-06-05
---

# finda.cloud — Tech Stack & SEO Audit

**Site:** https://finda.cloud/ · **Company:** Finda Cloud Technology Limited — APAC cloud / SaaS value-added distributor · **Audited:** 2026-06-05 (external black-box analysis, no server access)

## Contents
{:#contents}

1. [Executive summary](#summary)
2. [Technology stack](#stack)
3. [What's working well](#working)
4. [Issues found](#issues)
5. [Recommendations](#recommendations)
6. [Target region & language](#region)
7. [Questions for the website manager](#questions)

---

## 1. Executive summary
{:#summary}

finda.cloud is a **client-rendered React single-page app on Firebase Hosting** with an **excellent
`<head>` meta layer** but **almost no content that search engines can read**. The page body is an empty
`<div id="root">` filled in by JavaScript, so non-JS crawlers and social scrapers see a blank page.

The site genuinely has a **news & events section** (stored in Firestore, e.g. `/news/1`, `/events/101`),
but that content is **invisible to search** for three independent reasons: it's not in the sitemap, it
uses numeric-ID URLs instead of keyword slugs, and it's rendered only in the browser.

The metadata, social cards, HTTPS, CDN and analytics are all done well. **The gap is rendering and
content discoverability, not tags.** Fixing how pages are rendered and how news/events are exposed
(sitemap + slugs + homepage links) addresses most of the issues at once.

---

## 2. Technology stack
{:#stack}

### Frontend

| Layer | Detail |
|---|---|
| Framework | **React 18** (`createRoot`) — SPA, rendered fully in the browser |
| Build tool | **Vite** (hashed `/assets/index-*.js` + `.css`, `modulepreload`) |
| Routing | **React Router** (client-side) |
| UI | **lucide-react** icons, **Framer Motion** animation |
| Bundle | Single, not code-split, ~1.04 MB uncompressed (served Brotli-compressed) |

### Hosting & infrastructure

| Layer | Detail |
|---|---|
| Host | **Firebase Hosting** (Google) — `hosting-site=findacloud-website`, IP `199.36.158.100` |
| CDN | **Fastly** edge (Tokyo PoP), HTTP/2 **and** HTTP/3, Brotli |
| TLS | Google Trust Services managed cert (confirm SAN lists `finda.cloud`) |
| DNS | **SiteGround** nameservers; registrar **GoDaddy**; reg. 2023, expires 2027 |
| Email | **Google Workspace** (MX); SPF includes email-marketing senders |

### Data & integrations

- **Cloud Firestore** — backs the dynamic **news/events** content, fetched client-side
- **EmailJS** — contact-form submission; **WhatsApp** click-to-chat link
- **Intercom "Fin AI"** support chat widget
- **Google Analytics** + **Microsoft Clarity** (behaviour analytics)
- **Google Search Console** verified; **Zoom** domain verification (distributor relationship)

---

## 3. What's working well
{:#working}

- Keyword-rich, well-sized `<title>` and `<meta description>`, region-targeted (HK / SG / APAC).
- Complete **Open Graph** + **Twitter Card** tags; valid OG image (1220×640).
- **Canonical** tag, `lang="en"`, responsive viewport.
- HTTPS + HSTS, HTTP→HTTPS redirect, HTTP/3, Brotli, fast TTFB (~0.2 s).
- Valid `sitemap.xml` for the main section routes; Google Search Console connected.

---

## 4. Issues found
{:#issues}

| Severity | Issue | Why it matters |
|---|---|---|
| 🔴 Critical | **Content rendered only in JS** — body is `<div id="root"></div>`, no `<h1>`/`<nav>`/`<noscript>` | Non-JS crawlers and social/LinkedIn/AI scrapers see an empty page; Google renders JS only on a slower second pass |
| 🔴 Critical | **News/events not in the sitemap** — only 9 static routes listed, no `/news/{id}` or `/events/{id}` | Real article content is undiscoverable; Google has no path to it |
| 🔴 Critical | **Numeric-ID URLs, no slugs** — articles live at `/news/1`, `/events/101` | URLs carry zero keyword signal; weaker relevance, click-through and shareability |
| 🔴 Critical | **Soft-404s on every route** — unknown paths return HTTP `200` + the app shell | Google flags soft-404s; crawl budget wasted; dead URLs look "alive" |
| 🟠 High | **`robots.txt` missing** — returns the app shell, not a real file | No crawl directives; sitemap isn't advertised |
| 🟠 High | **No structured data (JSON-LD)** | No `Organization` / `Article` rich results; weaker entity recognition |
| 🟡 Medium | **No `hreflang`, English-only** | No Traditional Chinese for the primary HK/Macau audience |
| 🟡 Medium | **Missing security headers** (only HSTS present) | No CSP, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy` |
| 🟢 Low | **1 MB single bundle, no code-splitting** | Hurts Core Web Vitals (LCP/TBT) on mobile/APAC networks |

---

## 5. Recommendations
{:#recommendations}

Ordered by impact. The first three solve most of the problem together.

| # | Action | Why / how |
|---|---|---|
| 1 | **Render real HTML** (SSR / SSG / prerender) | Highest ROI — gives crawlers and social scrapers actual content. Add a prerender step to the Vite build (`vite-plugin-prerender`, `react-snap`) or move to Next.js/Astro |
| 2 | **Make news/events indexable** | (a) **Generate sitemap entries dynamically** from the Firestore collection (with `lastmod`); (b) switch detail URLs to **keyword slugs** (`/news/{slug}`), self-canonical per article; (c) ensure each is prerendered |
| 3 | **Surface news/events on the homepage** | Add a "Latest news / events" block on `/` linking to each article. Improves internal linking, crawl discovery and freshness signals — and is good for visitors |
| 4 | **Return real `404`s** for unknown routes | Stop serving `200` + app shell for everything; 301 any stray old URLs to a relevant page |
| 5 | **Add a `robots.txt`** referencing the sitemap | Currently missing; advertise the (extended) sitemap and submit in Search Console |
| 6 | **Add JSON-LD** | `Organization` + `LocalBusiness` (HK office) site-wide, `Article`/`NewsArticle` on each news/event page |
| 7 | **Add `hreflang` + Traditional Chinese** | Match how the HK/Macau market searches |
| 8 | **Add security headers** | CSP, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`, `X-Frame-Options` |
| 9 | **Code-split the bundle** | Improve Core Web Vitals; drop the obsolete `<meta keywords>` |

**On a blog:** the existing news/events section can double as the keyword engine once it's indexable
(points 1–3). A dedicated blog/insights hub is worthwhile later for informational terms (Tencent Cloud,
Zoom, Freshworks, SaaS distribution in HK/SG), ideally bilingual — but fixing news/events visibility
comes first and reuses content you already have.

---

## 6. Target region & language
{:#region}

Evidence points to **Hong Kong + Macau + Singapore, broader APAC**: meta copy "HK, SG & APAC"; Tencent
Cloud **港澳 (HK/Macau)** authorized distributor; Zoom **Hong Kong** distributor; office in **Kwun Tong,
Kowloon, Hong Kong**.

Implications:

- Add **`hreflang`** for `en` and `zh-Hant` (consider `zh-HK`) and publish **Traditional Chinese**
  versions of key pages and articles.
- Add **`LocalBusiness` / `Organization` JSON-LD** with the HK address; set up / verify Google Business
  Profile for local-pack visibility.
- Confirm **international targeting** in Search Console and that the sitemap covers both language variants.

---

## 7. Questions for the website manager
{:#questions}

These can't be determined from outside — please confirm:

1. **Rendering plan** — is the SPA staying client-rendered, or is SSR/prerendering on the roadmap?
2. **Sitemap generation** — `sitemap.xml` covers only static routes; can it loop the Firestore
   news/events collection so every article is included automatically?
3. **News/event URLs** — can detail pages move from numeric IDs (`/news/1`) to **keyword slugs**? Is
   there a slug field in Firestore, or would we add one? Should old numeric URLs 301 to new slugs?
4. **Homepage news block** — can we add a "latest news/events" section on the homepage linking to each
   article?
5. **404 handling** — is the catch-all `200` + app shell intentional, or should unknown routes 404?
6. **Languages** — is a Traditional Chinese (zh-Hant) version planned, or is English-only deliberate?
7. **Search Console / GA4 access** — to confirm current indexing, soft-404 reports and top queries.
8. **Analytics IDs** — confirm the GA4 / Microsoft Clarity property IDs so we can validate tracking.
9. **TLS cert** — confirm the production certificate's SAN list includes `finda.cloud`.
10. **www / domain** — should `www.finda.cloud` resolve and redirect to the apex? (It currently doesn't.)

---

*Black-box audit — based on externally observable signals (HTTP headers, HTML/JS bundle, DNS/WHOIS,
public search). Server-side configuration should be confirmed with the team.*
