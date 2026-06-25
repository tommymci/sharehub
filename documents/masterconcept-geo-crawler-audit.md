---
title: "Master Concept — GEO / AI-Crawler Access Audit"
date: 2026-06-25
---

# Master Concept — GEO / AI-Crawler Access Audit

_Site: masterconcept.ai · Inspected 2026-06-25 (frontend + AIOSEO schema layer + edge/CDN crawler test)_

## Executive summary
On-page **SEO is healthy** — Google/Bing and browsers see clean, complete structured data. The real
issue is at the **edge: Cloudflare is blocking AI crawlers**, so generative engines (ChatGPT, Claude,
Perplexity, Gemini) **cannot fetch the site at all**. Any GEO audit that crawls as an AI bot reads a
**block page**, which produces *false* "missing schema / 404 / duplicate metadata" findings.

---

## 1. Cloudflare crawler-access test — the key finding
Same URL, different `User-Agent`:

| Crawler | Category | HTTP | Bytes | Schema | Sees the page? |
|---|---|---|---|---|---|
| **Googlebot** | Search | `200` | 812 KB | ✓ | ✅ Full |
| **Bingbot** | Search | `200` | 812 KB | ✓ | ✅ Full |
| **SemrushBot** | SEO | `200` | 812 KB | ✓ | ✅ Full |
| **AhrefsBot** | SEO | `200` | 812 KB | ✓ | ✅ Full |
| **GPTBot** (OpenAI training) | AI | `520` | 16 B | ✗ | ❌ Blocked |
| **ChatGPT-User** (OpenAI cite) | AI | `403` | block page | ✗ | ❌ Blocked |
| **ClaudeBot** (Anthropic train) | AI | `520` | 16 B | ✗ | ❌ Blocked |
| **PerplexityBot** (Perplexity) | AI | `403` | block page | ✗ | ❌ Blocked |
| **Google-Extended** (Gemini) | AI | `520` | 16 B | ✗ | ❌ Blocked |

**Pattern:** search + SEO bots = full access; **every major AI crawler = blocked** (403/520).
CDN = **Cloudflare** (confirmed via `server: cloudflare` + `cf-ray`), with "Block AI bots" effectively on.

---

## 2. On-page SEO is actually fine (audit P0s do not reproduce)
Verified on the homepage, a real blog post, and `/solutions/geospatial/` (read as a browser / Googlebot):

| Reported "P0 / Critical" | Reality on masterconcept.ai | Verdict |
|---|---|---|
| Article schema **404** (`mainEntityOfPage`, `image`) | All schema URLs (`mainEntityOfPage`, `image`, `author`, `publisher`) → **200** | ❌ Not found |
| **Duplicate** title / desc / OG / **2× H1** | **1** title, **1** description, **1** OG set, **1** H1, **1** canonical per page | ❌ Not found¹ |
| Content-layer **schema missing** | Posts: Article + Breadcrumb + Organization + Person + WebPage + WebSite. Pages: WebPage + Breadcrumb + Org. Homepage: Org + WebSite + WebPage + Breadcrumb | ❌ Not found |
| robots / sitemap / hreflang | robots standard; AIOSEO sitemap fresh (lastmod today); hreflang present + reciprocal | ✅ Healthy |

SEO plugin: **AIOSEO v4.9.8** (only one — no plugin-vs-plugin conflict).
¹ *Only exception:* homepage has 2 `<h1>` ("Get The Heavy Lifting Solved" + "Become CIRRUS") — minor, make one an H2.

---

## 3. Why the vendor audit (GenOptima / 智推时代) reported those P0s
A GEO audit crawls **as an AI bot**. Those bots hit Cloudflare and get `403/520` → the tool parses a
**block/challenge page**, not the real page, and reports:
- "Article schema 404" = couldn't fetch the linked entity (blocked)
- "2 titles / 2 desc / 2 OG / 2 H1" = artifacts of parsing an error/fallback page
- "content schema missing" = JSON-LD never delivered (403)

**Tell-tale:** the report's quoted `robots.txt` does not match the site's real `robots.txt` → the data
is stale/synthetic, not a live rendered crawl. **The framework is reasonable; the findings are unverified.**

---

## 4. The real fix — allow AI *citation* bots at Cloudflare
Let answer-engines that **cite + drive referral traffic** through; keep blocking bulk **training** scrapers.

- **Allow (citation / answer engines):** `ChatGPT-User` · `OAI-SearchBot` · `PerplexityBot` · `Perplexity-User` · `Claude-User`
- **Keep blocking (bulk trainers):** `GPTBot` · `CCBot` · `ClaudeBot` · `Google-Extended` · `Bytespider` · `Amazonbot` · `meta-externalagent`

### ✅ Recommended for our plan — **works on Cloudflare FREE** (no paid add-on needed)
Our zone is on the **Free** plan, so do it by **User-Agent matching** (no Bot Management required):

1. **Turn OFF** the "Block AI bots" toggle (Cloudflare → **Security → Bots**). On Free it's all-or-nothing
   and is what's currently blocking the *citation* bots we want.
2. Add **one WAF Custom Rule** (Free includes 5) — Security → WAF → Custom rules — **Action: Block**:

```
(http.user_agent contains "GPTBot") or (http.user_agent contains "CCBot") or
(http.user_agent contains "ClaudeBot") or (http.user_agent contains "Google-Extended") or
(http.user_agent contains "Bytespider") or (http.user_agent contains "Amazonbot") or
(http.user_agent contains "meta-externalagent")
```

**Result:** trainers blocked; `ChatGPT-User` / `OAI-SearchBot` / `PerplexityBot` / `Perplexity-User` /
`Claude-User` pass through (HTTP 200) and can read + cite the site.

> **Trade-off (Free):** this matches **UA strings**, which can be spoofed — a scraper *could* fake a
> citation UA to get in. For GEO/citation that's an acceptable risk (those are the bots we *want*).
> Cryptographic verification needs **Bot Management** (Enterprise) — overkill here.

### Plan tiers (so support knows what's needed)
| Approach | Plan |
|---|---|
| "Block AI bots" toggle + WAF custom rule by **User-Agent** (the recommendation above) | **Free** ✅ |
| **Super Bot Fight Mode** — nicer UI with an "AI bots" allow/block category | Pro (~US$20/mo) — optional |
| `cf.bot_management.verified_bot` (verify bot authenticity, the *original* drafted rule) | Enterprise ❌ not needed |

Supplementary **robots.txt** (AIOSEO → Tools → Robots.txt) — allow the citation bots, disallow trainers.
*(Cloudflare is the real gate; robots.txt only helps bots that already get through.)*

---

## 5. Priorities
1. **🔴 Unblock AI citation bots at Cloudflare** — the #1 GEO lever (everything else is moot while they get 403).
2. **🟡 Homepage → single H1.**
3. **🟢 (optional) GEO enrichment** once crawlable: FAQPage schema on Q&A pages, Service schema on solution pages, confirm `max-snippet:-1` / `max-image-preview:large` in AIOSEO.

> Bottom line: the structured data is already good — it just isn't reaching the AI engines. Fix the
> Cloudflare access and the "GEO problem" largely solves itself; the vendor's schema P0s are block-page artifacts.
