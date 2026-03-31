# Master Concept Website Migration Plan

> **Status Legend:** `DECIDED` = confirmed approach | `PLANNED` = will implement | `NOTED` = recorded for future reference | `ACTIVE` = working on now

---

## Table of Contents

### Strategy & Decisions
1. [Domains & Strategy](#1-domains--strategy) `DECIDED`
2. [Context & Goals](#2-context--goals) `DECIDED`
3. [Architecture Overview](#3-architecture-overview) `DECIDED`
4. [Tech Stack](#4-tech-stack) `DECIDED`
5. [Hosting & Infrastructure](#5-hosting--infrastructure) `DECIDED`
6. [Domain Routing (Transition)](#6-domain-routing-transition) `NOTED`

### CMS & Content
7. [Payload CMS Features](#7-payload-cms-features) `NOTED`
8. [Payload Plugins](#8-payload-plugins) `NOTED`
9. [Payload CMS vs WordPress](#9-payload-cms-vs-wordpress) `NOTED`
10. [Block System & Templates](#10-block-system--templates) `NOTED`
11. [Internal Link Integrity](#11-internal-link-integrity) `NOTED`
12. [File & Media Management](#12-file--media-management) `NOTED`

### Development Workflow
13. [Local Development & Preview](#13-local-development--preview) `PLANNED`
14. [Git, CI/CD & Deployment](#14-git-cicd--deployment) `PLANNED`
15. [Vibe Coding Workflow](#15-vibe-coding-workflow) `NOTED`
16. [3-Layer Guardrails](#16-3-layer-guardrails) `PLANNED`

### Migration
17. [Migration Phases](#17-migration-phases) `PLANNED`
18. [SEO Safety](#18-seo-safety) `PLANNED`
19. [Converting Elementor Pages](#19-converting-elementor-pages) `NOTED`
20. [Cross-Linking Blog & Landing Pages](#20-cross-linking-blog--landing-pages) `PLANNED`

### Integrations & Future
21. [HubSpot Integration](#21-hubspot-integration) `NOTED`
22. [AI-Automated SEO](#22-ai-automated-seo) `NOTED`
23. [WebMCP Readiness](#23-webmcp-readiness) `NOTED`
24. [Future Enhancements](#24-future-enhancements) `NOTED`
25. [Reference URLs](#25-reference-urls)

---

## 1. Domains & Strategy
`DECIDED`

| Domain | Purpose |
|--------|---------|
| `masterconcept.ai` | **Current production** (WordPress) → future Next.js |
| Trial domain (TBD) | Options: `site.mcai.dev`, `masterconcept.io`, or other - flexible |

**Strategy:** Build and test the new framework on a separate trial domain first. No changes to production during trial phase. Migration to `masterconcept.ai` happens later.

---

## 2. Context & Goals
`DECIDED`

**Current:** WordPress + Elementor on SiteGround (GoGeek shared hosting)
- **Blog** - managed by marketing team
- **Landing pages** - managed by Tommy

**Problems:**
- Elementor is not vibe-codeable (content stored as serialized DB data)
- AI cannot easily read, edit, or generate Elementor pages
- Performance issues and plugin conflicts
- No path to WebMCP integration

**Goal:** Modern, AI-friendly stack where:
- Landing page design is vibe-coded (AI edits code directly)
- All content editable in admin panel (no code needed for content)
- Blog stays on WordPress during transition
- Same domain, no broken links, no SEO loss

---

## 3. Architecture Overview
`DECIDED`

### Front Door
> **GCP Cloud Run** (hosted on GCP, auto-deploy via GitHub Actions)

### Routing (Production - During Transition)

Routing approach TBD (see [Section 6](#6-domain-routing-transition)). Target:

| Path | Destination |
|------|-------------|
| `/blog/*` | WordPress on SiteGround |
| `/wp-admin/*` | WordPress on SiteGround |
| `/wp-content/*` | WordPress on SiteGround |
| `/admin/*` | Payload CMS Admin Panel |
| `/api/*` | Payload REST + GraphQL API |
| `/*` | Next.js 15 App (landing pages) |

### Frontend Stack

| Component | Role |
|-----------|------|
| **Next.js 15** (App Router, TypeScript) | Framework |
| **shadcn/ui** | Design system |
| **Tailwind CSS** | Styling |
| **Blocks** | Hero, CTA, Features, Pricing, Testimonial, BlogList, HubSpot Form, Download, etc. |

### API Consumers

| Consumer | How |
|----------|-----|
| **Claude Code** | Vibe coding + file upload via Local API |
| **Admin Users** | Edit content in browser at `/admin` |
| **WebMCP** (future) | AI agents via MCP protocol |

### Data Layer (All GCP)

| Service | Stores |
|---------|--------|
| **Cloud SQL PostgreSQL** (`mc-db`) | Pages, Posts, Categories, Tags, Users, Forms, SEO, Navigation, Settings |
| **GCP Cloud Storage** (`mc-media`) | Images (CDN), Ebooks/PDFs, Downloads, Auto-thumbnails |

### 3-Layer Guardrails

| Layer | Tool | Constrains |
|-------|------|------------|
| UI Design | shadcn/ui | Consistent components, prevents messy UI |
| Content Structure | Payload CMS | Schema-enforced content types |
| Code Patterns | CLAUDE.md / .cursorrules | Folder structure, naming, best practices |

---

## 4. Tech Stack
`DECIDED`

| Layer | Tool | Purpose |
|-------|------|---------|
| Framework | **Next.js 15** (App Router, TypeScript) | Frontend + backend in one app |
| CMS | **Payload CMS 3.x** | Content management, admin panel, API, auth |
| Design System | **shadcn/ui** | Consistent UI components for vibe coding |
| Styling | **Tailwind CSS** | Utility-first CSS via shadcn theming |
| Storage | **GCP Cloud Storage** (`mc-media`) | All media/files, served via CDN |
| Database | **Cloud SQL PostgreSQL** (`mc-db`) | GCP native, covered by GCP credits |
| Hosting | **GCP Cloud Run** (`mc-website`) | Hosts Next.js + Payload app |
| CI/CD | **GitHub Actions** | Auto-deploy on git push to main |
| Registry | **GCP Artifact Registry** (`mc-registry`) | Docker image storage |
| Forms | **HubSpot** (existing) | Lead capture, embedded via block component |
| AI Guardrails | **CLAUDE.md / .cursorrules** | Enforces code patterns for structured vibe coding |
| Blog (transition) | **WordPress on SiteGround** | Until blog migrated to Payload |

---

## 5. Hosting & Infrastructure
`DECIDED`

### GCP Resources (mc- prefix)

| Resource | Name | Purpose |
|----------|------|---------|
| Cloud Run Service | `mc-website` | Hosts Next.js + Payload app |
| Cloud SQL Instance | `mc-db` | PostgreSQL database |
| Cloud SQL Database | `mc-payload` | Payload data |
| Cloud Storage Bucket | `mc-media` | All uploaded files |
| Artifact Registry | `mc-registry` | Docker images |
| Service Account | `mc-deploy@PROJECT.iam.gserviceaccount.com` | Deploy access |

### Environment Variables

```env
# Database
DATABASE_URI=postgresql://USER:PASSWORD@HOST:5432/mc-payload

# Payload CMS
PAYLOAD_SECRET=your-random-secret-key-here
PAYLOAD_API_KEY=key-for-claude-code-access

# GCP Cloud Storage
GCS_BUCKET=mc-media
GCS_PROJECT_ID=your-gcp-project-id

# App
NEXT_PUBLIC_SITE_URL=https://site.mcai.dev
NODE_ENV=production
```

### Cost

**All GCP services covered by free GCP credits.**

| Service | Role | Cost (after credits) |
|---------|------|------|
| Cloud Run (`mc-website`) | Next.js + Payload | ~$5-10/month |
| Cloud SQL (`mc-db`) | Database (db-f1-micro) | ~$7-10/month |
| Cloud Storage (`mc-media`) | Media/files | ~$1-2/month |
| SiteGround GoGeek | WordPress blog (during transition) | ~$35/month (existing) |
| **After full migration** | All GCP only | **~$15-25/month total** |

### Why Not Firebase?

Firebase App Hosting is just a wrapper around GCP Cloud Run. Since you already have GCP with credits, use Cloud Run directly - full control, no extra abstraction.

---

## 6. Domain Routing (Transition)
`NOTED` - approach not decided yet, will finalize when ready for production migration

### The Problem
`masterconcept.ai` DNS points to SiteGround (WordPress). Need both WordPress (blog) and Cloud Run (new pages) on the same domain during transition.

### Options Discussed

**Option A: Cloudflare Worker** (you have paid Cloudflare)
- Worker routes by URL path: `/blog/*` → WordPress, `/*` → Cloud Run
- No DNS changes, no email impact
- Needs domain admin to add Worker Route + enable proxy (orange cloud)

**Option B: Cloud Run as reverse proxy**
- DNS points to Cloud Run instead of SiteGround
- Next.js `rewrites` in config proxy blog paths to SiteGround
- Risk: if Cloud Run goes down, WordPress also unreachable

**Option C: WordPress as front door**
- WordPress stays as default, specific paths proxy to Cloud Run
- Lowest risk but more complex SiteGround config

**Trial phase:** Not applicable - trial domain points directly to Cloud Run. This decision only matters when migrating `masterconcept.ai`.

---

## 7. Payload CMS Features
`NOTED`

### Core Architecture
- Lives **inside** your Next.js app (not a separate service)
- 100% TypeScript with full type safety
- Config-based: everything defined in code, version controlled
- Free & open source (MIT license)

### Content Management (Like WordPress)

| Feature | How It Works |
|---------|-------------|
| **Slug management** | Auto-generated from title, editable, unique per collection |
| **Tags** | Relationship collection, create/edit/delete in admin, assign to posts |
| **Categories** | Relationship collection with parent/child hierarchy |
| **Page hierarchy** | Parent/child pages via Nested Docs plugin (breadcrumbs, tree) |
| **Drafts** | Save as draft without publishing, autosave |
| **Scheduled publishing** | Set publish date/time, auto-publishes |
| **Version history** | Full diff viewer, one-click restore to any version |
| **Media library** | Upload, crop, focal point, auto-generated thumbnails |
| **Search** | Built-in search across all collections |
| **Bulk operations** | Bulk select, edit, delete |

### Content Modeling
- **Collections**: Pages, Posts, Media, Categories, Tags, Users, Forms
- **Globals**: Site Settings, Navigation, Footer (singleton data)
- **Blocks Field**: Modular content blocks (like Gutenberg but code-defined)
- **30+ field types**: text, richText, number, date, relationship, array, group, blocks, tabs, upload, select, checkbox, and more
- **Conditional logic**: show/hide fields based on other field values

### Admin Panel (`/admin`)
- **Auto-generated UI** from config - no manual admin building
- **Live Preview**: real-time side-by-side editing
- **Rich Text Editor**: Lexical-based, similar to WordPress Gutenberg
- **Dark mode**, **white-labeling** (custom logo/colors)
- **Anyone can create pages from existing blocks** - no code needed, just pick blocks and fill content

### Authentication & Roles

| Role | Who | Permissions |
|------|-----|-------------|
| **Admin** | Tommy | Full access |
| **Editor** | Marketing team | Create/edit/publish posts & pages, upload media |
| **Author** | Blog writers | Create/edit own posts only |
| **API Key** | Claude Code | API access (no browser login) |

### API (3 Ways to Access Content)
- **REST API** - full CRUD, pagination, filtering
- **GraphQL API** - flexible queries & mutations
- **Local API** - direct database access from server code (Claude Code uses this)

---

## 8. Payload Plugins
`NOTED`

| Plugin | What It Does |
|--------|-------------|
| `@payloadcms/plugin-seo` | Meta title, description, Open Graph fields |
| `@payloadcms/storage-gcs` | Store uploads in GCP Cloud Storage |
| `@payloadcms/plugin-form-builder` | Admin-built forms, submissions, email notifications |
| `@payloadcms/plugin-redirects` | URL redirect management (critical for migration) |
| `@payloadcms/plugin-nested-docs` | Parent/child page hierarchy with breadcrumbs |
| `@payloadcms/plugin-search` | Fast search collection sync |

---

## 9. Payload CMS vs WordPress
`NOTED`

| Feature | WordPress | Payload CMS 3.x |
|---------|-----------|-----------------|
| Pages & Posts | Built-in | Collections (you define) |
| Page hierarchy | Parent page dropdown | Nested Docs plugin (breadcrumbs + tree) |
| Categories & Tags | Built-in | Relationship collections |
| Rich text editor | Gutenberg | Lexical (similar experience) |
| Media library | Built-in | Upload collections + auto thumbnails + crop |
| Page templates | PHP template files | React/shadcn components |
| Live preview | Preview in new tab | Side-by-side iframe (real-time) |
| Version history | Basic revisions | Full diffs + one-click restore |
| SEO | Yoast/RankMath (mature) | SEO plugin (simpler but sufficient) |
| Access control | 5 fixed roles | Custom RBAC, document + field level |
| API | REST only | REST + GraphQL + Local API |
| Performance | 180-250ms TTFB | 50-120ms TTFB |
| Plugin ecosystem | 60,000+ | ~50 official (growing) |
| AI/vibe coding | Not possible | Native (code-first, TypeScript) |
| Block flexibility | Static HTML output | Full-stack React (can call APIs, query DB, use any npm package at render time) |

---

## 10. Block System & Templates
`NOTED`

### Starting Point: Payload Website Template

```bash
npx create-payload-app@latest -t website
```

**Pre-built blocks included:**

| Block | Description |
|-------|-------------|
| Hero | Headline, subtitle, CTA buttons, background image |
| Content | Rich text content section |
| Media Block | Full-width or inline images/videos |
| Call to Action | CTA banner with heading + button |
| Archive | Auto-lists posts (like WordPress blog archive) |
| Form Block | Dynamic forms |
| Code Block | Syntax-highlighted code |
| Banner | Alert/notice banner |

### shadcn/ui Block Library (Additional UI Components)

Hero sections, feature grids, pricing cards, testimonials, FAQ, CTA, footer, header/nav, stats, team cards, blog list, and more.

### Custom Blocks Needed

| Block | Based on |
|-------|----------|
| HubSpotFormBlock | Your existing HubSpot forms |
| DownloadBlock | Ebook download pages |
| FlipCardsBlock | `html-widgets/` folder (already have HTML) |
| ComparisonTableBlock | `html-widgets/` folder |
| ServiceTabsBlock | `html-widgets/` folder |

**Your `html-widgets/` folder = block prototypes.** Claude Code converts each to a Payload block component.

### Why Blocks Are More Flexible Than WordPress

WordPress blocks output **static HTML**. Payload blocks are **full-stack React components** that can:
- Call external APIs at render time (dynamic pricing, live data)
- Query database (show related posts, filter by user)
- Use any npm package (charts, maps, 3D, animations)
- Personalize by user/location
- Integrate AI at render time

---

## 11. Internal Link Integrity
`NOTED`

**Key advantage over WordPress.**

WordPress stores internal links as raw URL strings. If you change a slug, links break. Payload stores links as **relationship references** (page ID):

| Scenario | WordPress | Payload CMS |
|----------|-----------|-------------|
| Change slug | Links break or redirect | All links auto-resolve |
| Delete a page | 404 errors | Admin warns "3 pages link to this" |
| Move page (change parent) | URL breaks | ID unchanged, auto-resolves |
| Find pages linking to X | Need plugin | Built-in query |

---

## 12. File & Media Management
`NOTED`

```
All uploads → Payload API → GCP Cloud Storage (mc-media bucket)
                  │
                  └→ Database record (filename, alt text, dimensions, thumbnails)
```

| Who | How | Tracked in CMS? |
|-----|-----|-----------------|
| Claude Code | Payload Local API | Yes |
| Admin user | Drag & drop in `/admin` | Yes |
| **Never** upload directly to Cloud Storage - always through Payload API |

---

## 13. Local Development & Preview
`PLANNED`

### Setup

```
Your laptop (localhost:3000)
├── Next.js dev server        ← hot reload
├── Payload CMS admin         ← localhost:3000/admin
├── Local PostgreSQL          ← dev content (safe to break)
└── Local file uploads        ← /media folder (dev only)
```

### Complete Vibe Coding Flow

```
Step 1: Claude creates block .tsx file
        → Hot reload → see empty block instantly

Step 2: Claude creates seed script + runs on LOCAL DB
        → Page appears at localhost:3000/pricing with content

Step 3: You fine-tune in localhost:3000/admin
        → Live preview, adjust text/images

Step 4: Claude syncs: reads local DB → updates seed script
        → Seed script now matches your final edits
        → Commit to Git

Step 5: git push → GitHub Actions → deploys to Cloud Run

Step 6: Run seed on production → page appears live
```

### Two Databases

| Environment | Database | Purpose |
|-------------|----------|---------|
| Local | Local PostgreSQL | Dev/test content (safe to break) |
| Production | Cloud SQL `mc-db` | Real content |

### Seed Scripts (Content as Code)

Seed scripts in Git capture initial page content. Idempotent (safe to run multiple times):

```
Git repo
├── src/blocks/PricingBlock.tsx    ← design (code)
├── src/seed/pricing-page.ts       ← content (code)
└── src/seed/assets/               ← seed images
```

After initial creation, ongoing content edits happen directly in production `/admin`.

---

## 14. Git, CI/CD & Deployment
`PLANNED`

### GitHub Repository: `mc-website`

### What Goes to GitHub vs What Doesn't

| GitHub (version controlled) | NOT in GitHub |
|-----------------------------|---------------|
| All code (`.tsx`, `.ts`, `.css`) | Media files → Cloud Storage |
| Block components | Database content → Cloud SQL |
| Payload collection schemas | `.env` files (secrets) |
| Seed scripts + seed assets | `node_modules/`, `.next/` |
| CLAUDE.md, configs | Service account keys |
| Dockerfile, GitHub Actions | User uploads |

### GitHub Actions Auto-Deploy

On `git push` to `main`:
1. Build Docker image
2. Push to GCP Artifact Registry (`mc-registry`)
3. Deploy to Cloud Run (`mc-website`)

### GitHub Secrets

| Secret | Value |
|--------|-------|
| `GCP_SA_KEY` | Service account JSON key |
| `DATABASE_URI` | Cloud SQL connection string |
| `PAYLOAD_SECRET` | Payload auth secret |
| `GCS_BUCKET` | `mc-media` |

### Day-to-Day Development Process

| Scenario | Process |
|----------|---------|
| **New feature/block** | `git checkout -b feature/new-block` → develop → PR → review → merge to `main` → auto-deploys |
| **Bug fix** | `git checkout -b fix/issue-name` → fix → PR → merge → auto-deploys |
| **Hotfix (urgent)** | Push directly to `main` → auto-deploys immediately |
| **Content update** | No deploy needed - edit in `/admin` directly |
| **Rollback bad deploy** | `git revert <commit>` → push → auto-deploys previous version |

### Future Updates & Maintenance

| Task | How |
|------|-----|
| **Update Payload CMS version** | `npm update @payloadcms/...` → test locally → push |
| **Update Next.js version** | `npm update next` → test locally → push |
| **Update shadcn/ui components** | `npx shadcn@latest add <component>` → test → push |
| **Add new Payload plugin** | `npm install @payloadcms/plugin-xxx` → configure → test → push |
| **Security patches** | Dependabot PRs on GitHub → review → merge → auto-deploys |

---

## 15. Vibe Coding Workflow
`NOTED`

### Three Channels, Each Handles Its Own Concern

| Channel | What | When |
|---------|------|------|
| **GitHub → Cloud Run** | Code changes (design, components, schemas) | Git push triggers auto-deploy |
| **Payload API → Cloud SQL** | Content changes (text, pages, SEO) | Instant via `/admin` or Claude Code API |
| **Payload API → Cloud Storage** | Media uploads (images, PDFs) | Instant via `/admin` or Claude Code API |

### Day-to-Day After Setup

| Task | Who | Where | Git? |
|------|-----|-------|------|
| Fix a typo | Anyone | `/admin` | No |
| Change pricing | Anyone | `/admin` | No |
| New blog post | Marketing | `/admin` | No |
| Create page from existing blocks | Anyone | `/admin` | No |
| Upload images | Anyone | `/admin` | No |
| **Design a NEW block type** | Claude Code | `.tsx` file | Yes |
| **Redesign how a block looks** | Claude Code | `.tsx` file | Yes |
| **Change site layout** | Claude Code | `.tsx` file | Yes |

**90% of daily work = `/admin` panel. Only new designs need Claude Code.**

---

## 16. 3-Layer Guardrails
`PLANNED`

| Layer | Tool | Purpose |
|-------|------|---------|
| Design System | shadcn/ui | All UI must use these components |
| Content & Pages | Payload CMS | Schema-enforced content types |
| AI Rules | CLAUDE.md + .cursorrules | Folder structure, naming, code patterns |

---

## 17. Migration Phases
`PLANNED`

### Phase 0: Trial on `site.mcai.dev` `ACTIVE`
- Build new site on separate domain
- No risk to production
- Learn the stack, build blocks, test admin panel
- Recreate a few pages to validate approach

### Phase 1: Set Up Production Infrastructure
- Set up Cloud SQL, Cloud Storage, Cloud Run on GCP
- Configure GitHub Actions auto-deploy
- Define all Payload collections
- Build header/footer/nav matching current brand
- Install Payload plugins

### Phase 2: Configure Domain Routing
- Set up Cloudflare Worker for `masterconcept.ai`
- Route `/blog/*` to WordPress, everything else to Cloud Run
- Test cross-linking between systems
- A record unchanged, just Worker route added

### Phase 3: Migrate Landing Pages (One by One)
- Screenshot → screenshot-to-code → React/Tailwind blocks
- Upload images via Payload API
- Create page in admin, same slug
- Monitor SEO per page (see [SEO Safety](#18-seo-safety))
- Migrate HubSpot form pages, ebook/download pages

### Phase 4: Migrate Blog (Optional - When Ready)
- Export WordPress posts via REST API
- Import into Payload Posts collection
- Marketing team switches to `/admin`
- Remove WordPress proxy rules
- Downgrade/cancel SiteGround

### Phase 5: Cleanup & Optimization
- Performance testing
- SEO verification (meta, structured data, sitemaps, redirects)
- Configure user roles
- Set up version history retention

---

## 18. SEO Safety
`PLANNED`

### What Google Cares About

| Factor | Risk | Action |
|--------|------|--------|
| Same URLs | Zero risk | Keep same slugs in Payload |
| Same content | Low risk | Copy exact content |
| No broken links | Low risk | 301 redirects via Payload plugin |
| Crawlable HTML | Zero risk | Next.js SSR = fully crawlable |
| Page speed | **Improves** | Next.js much faster than Elementor |
| Domain authority | Zero risk | Same domain |
| Backlinks | Zero risk | Same URLs |

### Page-by-Page Migration (No Big Bang)

```
Move ONE page → monitor 2 weeks → rankings stable? → move next page
```

If any page drops: investigate, fix, or rollback that single page (WordPress still running).

### Per-Page Checklist

```
Before: Record rankings, impressions, clicks
After:  Verify URL, content, meta, images, structured data, speed
        Submit for re-indexing in GSC
        Monitor for 2 weeks
```

### Performance Should IMPROVE

| Factor | WordPress + Elementor | Next.js + Payload |
|--------|----------------------|-------------------|
| Page load | 2-4 seconds | 0.5-1.5 seconds |
| TTFB | 180-250ms | 50-120ms |
| JS bundle | 500KB-2MB | 50-200KB |

---

## 19. Converting Elementor Pages
`NOTED`

1. **Screenshot-to-code** - AI generates React/Tailwind from screenshots
2. **Images** - upload to Payload Media → GCP Cloud Storage
3. **Header/Footer** - rebuild once, match current brand (same logo, colors, fonts, nav)
4. **Popups** - rebuild as React modal components
5. **HubSpot forms** - embed via HubSpotFormBlock
6. **Ebooks/downloads** - upload to Payload Media
7. **`html-widgets/`** - convert existing HTML widgets to Payload block components

### Style Consistency During Transition

Blog (WordPress) and landing pages (Next.js) can coexist on same domain. Keep same header/footer/nav/brand colors/fonts = visitors won't notice. Landing pages already look different from blog posts on most sites.

---

## 20. Cross-Linking Blog & Landing Pages
`PLANNED`

| Scenario | Solution |
|----------|----------|
| Blog post links to landing page | Normal links (`/pricing`) - same domain |
| Landing page shows blog posts | Fetch from WordPress REST API (transition) → later from Payload |
| Keep original URLs | Same slug in Payload CMS |
| Images from WordPress | `next/image` remote patterns (transition) |

---

## 21. HubSpot Integration
`NOTED`

| Feature | How |
|---------|-----|
| Embed HubSpot forms | HubSpotFormBlock (paste Portal ID + Form ID in admin) |
| Sync submissions | Payload `afterChange` hook → HubSpot API |
| Ebook download + form | HubSpotFormBlock + PDF link from Payload Media |

---

## 22. AI-Automated SEO
`NOTED`

| SEO Task | How |
|----------|-----|
| Meta titles & descriptions | Claude reads content → generates → writes to Payload SEO fields |
| Open Graph images | Auto-generate via Next.js `ImageResponse` API |
| Alt text for images | Claude generates when uploading via Payload API |
| Internal linking | Claude reads all pages → suggests/adds links |
| Structured data (JSON-LD) | Claude generates schema markup from content |
| Sitemap | Next.js auto-generates from Payload pages/posts |
| Bulk SEO audit | Claude reads all pages → flags missing meta, broken links |

---

## 23. WebMCP Readiness
`NOTED`

**WebMCP** (W3C Draft, Feb 2026) lets websites expose capabilities to AI agents in the browser. Different from regular MCP - runs client-side via `navigator.modelContext`.

- Status: Chrome 146 Canary early preview, cross-browser ~late 2027
- **Works on ANY website** (WordPress or Next.js) - not a migration reason
- Next.js has `webmcp-next` plugin for easier integration
- Can add to WordPress too via HTML `toolname` attributes or JS API

When ready, Payload APIs can be exposed as WebMCP tools for AI agents to create/update pages, upload media, run SEO audits.

---

## 24. Future Enhancements
`NOTED`

- WebMCP server when protocol matures
- AI SEO automation (scheduled audits)
- Full blog migration from WordPress to Payload
- Cancel SiteGround after full migration
- Multi-language via Payload's built-in localization
- AI content generation via Payload API

---

## 25. Reference URLs

- Payload CMS demo: https://demo.payloadcms.com/admin/login
- Payload CMS docs: https://payloadcms.com/docs
- Payload vs WordPress: https://payloadcms.com/compare/wordpress
- Payload website template: https://github.com/payloadcms/template-website
- Payload plugins: https://payloadcms.com/docs/plugins/overview
- screenshot-to-code: https://github.com/abi/screenshot-to-code
- shadcn/ui: https://ui.shadcn.com
- GCS Storage plugin: https://payloadcms.com/docs/plugins/cloud-storage
- Cloud Run docs: https://cloud.google.com/run/docs
- GitHub Actions + Cloud Run: https://cloud.google.com/run/docs/continuous-deployment-with-cloud-build
