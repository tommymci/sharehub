---
title: "masterconcept.ai — AI Crawler Access Fix (GEO Remediation)"
date: 2026-06-26
---

# masterconcept.ai — AI Crawler Access Fix (GEO Remediation)

**Audience:** Cloudflare administrator for `masterconcept.ai`
**Goal:** let AI *citation* engines (ChatGPT, Claude, Perplexity, etc.) read the site so it can be cited in AI answers (GEO), while still blocking bulk *training* scrapers.

> 📎 **Companion to the diagnostic audit** — *why* this is needed (the crawler-access evidence and on-page SEO reconciliation) is documented separately in **[Master Concept — GEO / AI-Crawler Access Audit](./masterconcept-geo-crawler-audit.html)**. This doc is the **how-to-fix**.

---

## TL;DR

1. Cloudflare Free's managed "Block AI bots" is **all-or-nothing** — it blocks the *citation* bots you want, not just trainers.
2. **Fix = Cloudflare → AI Crawl Control:** block the training crawlers, allow the citation + search crawlers.
3. Verify by requesting the site as each bot: expect **search + citation = 200, training = 403**.

---

## Background — why AI engines can't see the site

Cloudflare Free's managed "Block AI training bots" toggle is **all-or-nothing**. It blocks **every** AI crawler by User-Agent — including the *citation* bots (`ChatGPT-User`, `PerplexityBot`, `Claude-User`) that we actually want for GEO. The result: generative engines get a 403 block page instead of the real content, so the site cannot be cited in AI answers.

**The fix is to replace that blunt toggle with per-crawler control: block trainers, allow citation engines.**

---

## The Fix — Cloudflare AI Crawl Control

AI Crawl Control gives per-crawler allow/block control. When you set a crawler to **Block**, Cloudflare deploys a managed WAF custom rule that hard-blocks it (HTTP 403) at the edge. On the Free plan, detection is by User-Agent string. (Upgrading to a paid plan adds Bot-Management identity verification, which is harder to spoof — optional.)

### Step 1 — Open AI Crawl Control
Cloudflare dashboard → select `masterconcept.ai` → left menu **AI Crawl Control** → **Security** tab.
Tick **"Show inactive crawlers"** so the full catalog is visible.

> Cloudflare occasionally renames this area (it began as "AI Audit"). If you don't see "AI Crawl Control", look under **Security → Bots / Control AI crawlers**.

### Step 2 — Set each crawler per the tables below
The **"Block Crawler"** toggle: **ON = blocked**, **OFF = allowed**.

#### 🔴 BLOCK — training / bulk scrapers (toggle ON)

| Crawler | Vendor |
|---|---|
| GPTBot | OpenAI |
| ClaudeBot | Anthropic |
| CCBot | Common Crawl |
| Bytespider | ByteDance |
| Amazonbot | Amazon |
| meta-externalagent | Meta |
| FacebookBot | Meta |
| Google-CloudVertexBot | Google |
| Applebot-Extended | Apple |
| PetalBot | Huawei |
| TikTok Spider | ByteDance |
| Anchor Browser | Anchor |
| Novellum AI Crawl | Novellum |
| ProRataInc | ProRata.ai |
| Timpibot | Timpi |

> If the catalog also lists `Google-Extended`, you can block it too — though it's primarily a *robots.txt* training-opt-out token (Google fetches with `Googlebot`), so it may not appear as a separate crawler here.

#### 🟢 ALLOW — citation / answer engines (toggle OFF) — these drive GEO referral traffic

| Crawler | Vendor | Category |
|---|---|---|
| ChatGPT-User | OpenAI | AI Assistant |
| OAI-SearchBot | OpenAI | AI Search |
| Claude-User | Anthropic | ⚠️ labeled "AI Crawler" but is user-triggered citation — **allow** |
| Claude-SearchBot | Anthropic | AI Search |
| PerplexityBot | Perplexity | AI Search |
| Perplexity-User | Perplexity | AI Assistant |
| DuckAssistBot | DuckDuckGo | AI Assistant |
| Applebot | Apple | AI Search |
| MistralAI-User | Mistral | AI Assistant |
| Manus Bot | Manus | AI Assistant |
| meta-externalfetcher | Meta | ⚠️ user-triggered — **allow** (vs `meta-externalagent` = block) |

#### 🟢 ALLOW — search engines (toggle OFF) — critical for SEO

`Googlebot` · `bingbot` · `Baiduspider` · `Terracotta Bot`

#### 🟢 ALLOW — first-party & archivers (toggle OFF)

`Cloudflare Crawler` (Cloudflare's own service bot — not a third-party scraper) · `archive.org_bot` · `Arquivo Web Crawler`

### ⚠️ The three pairs most often mis-toggled

| Allow ✅ | Block 🔴 | Why |
|---|---|---|
| **Claude-User** | ClaudeBot | user-triggered citation vs training scraper |
| **meta-externalfetcher** | meta-externalagent | user-triggered fetch vs training |
| **Applebot** | Applebot-Extended | search crawler vs AI-training token |

### Step 3 — Make sure the old managed toggle is off
If `masterconcept.ai` still has the legacy **"Block AI training bots"** managed setting (Overview → Control AI crawlers, or Security → Bots) set to "Block on all pages", switch it to **Do not block**. AI Crawl Control's per-crawler rules now handle blocking. Running both can double-block or conflict.

> Also remove any hand-written WAF custom rule that blocks AI user-agents, so AI Crawl Control is the single source of truth.

---

## Supplementary — robots.txt (defense-in-depth)

Cloudflare is the real gate (it enforces at the edge regardless of robots.txt), but a matching `robots.txt` documents intent for well-behaved bots and tools. In **AIOSEO → Tools → Robots.txt**, mirror the policy: `Allow` the citation + search agents, `Disallow` the trainers (`GPTBot`, `CCBot`, `ClaudeBot`, `Bytespider`, `Amazonbot`, `Google-Extended`, `Applebot-Extended`, …). Keep the existing AIOSEO sitemap reference.

---

## Verification

Request the site with each crawler's User-Agent and check the HTTP status. A blocked bot gets **403**; an allowed one gets **200**.

```bash
# Should be BLOCKED (403) — a training crawler
curl -s -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0 (compatible; GPTBot/1.1; +https://openai.com/gptbot)" https://masterconcept.ai/

# Should be ALLOWED (200) — a citation crawler
curl -s -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0 (compatible; ChatGPT-User/1.0; +https://openai.com/bot)" https://masterconcept.ai/

# Should be ALLOWED (200) — a search engine
curl -s -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://masterconcept.ai/
```

You can also watch live results in **AI Crawl Control → Security** (each crawler's Allowed / Unsuccessful request counts).

### Expected result after the fix

| Group | Crawlers | Expected |
|---|---|---|
| Browser / Search | Browser, Googlebot, Bingbot | **200** |
| SEO tools | SemrushBot, AhrefsBot | **200** (not in AI Crawl Control catalog) |
| AI **training** | GPTBot, ClaudeBot, CCBot, Bytespider, Amazonbot, meta-externalagent | **403** |
| AI **citation** | ChatGPT-User, OAI-SearchBot, PerplexityBot, Perplexity-User, Claude-User | **200** |

> Caveat: on Cloudflare **Free**, allow/block is by User-Agent string, which can be spoofed. That is acceptable for GEO (these are bots you *want*). For cryptographic bot verification, upgrade to a paid plan (Bot Management).

---

## Summary

| Item | Decision |
|---|---|
| Cloudflare | Use AI Crawl Control: block trainers, allow citation + search |
| Legacy "Block AI bots" toggle | Set to **Do not block** |
| Hand-written AI WAF rule | Remove (AI Crawl Control replaces it) |
| robots.txt (AIOSEO) | Mirror the policy (allow citation/search, disallow trainers) |
| Verify | curl as each bot → search + citation 200, training 403 |
