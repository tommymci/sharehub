---
title: Master Concept Website Migration Plan
---

# Master Concept Website Migration Plan

## Table of Contents

1. [Domains](#domains)
2. [Architecture Overview](#architecture-overview)
3. [Context](#context)
4. [Target Architecture](#target-architecture)
5. [Tech Stack](#tech-stack)
6. [Payload CMS Feature Overview](#payload-cms-feature-overview)
7. [Official Payload Plugins](#official-payload-plugins)
8. [3-Layer Guardrail System](#3-layer-guardrail-system-structured-vibe-coding)
9. [How Landing Pages Work](#how-landing-pages-work-code--cms-hybrid)
10. [File & Media Management](#file--media-management)
11. [HubSpot Integration](#hubspot-integration)
12. [Domain Routing](#domain-routing-split-architecture)
13. [Cross-Linking Blog & Landing Pages](#cross-linking-between-blog--landing-pages)
14. [Migration Phases](#migration-phases)
15. [Hosting & Cost](#hosting--cost)
16. [Converting Elementor Pages](#converting-elementor-pages-to-code)
17. [Payload CMS vs WordPress](#payload-cms-vs-wordpress-comparison)
18. [AI-Automated SEO](#ai-automated-seo)
19. [WebMCP Readiness](#webmcp-readiness)
20. [Future Enhancements](#future-enhancements)
21. [Reference URLs](#key-urls-for-reference)

---

## Domains

| Domain | Purpose |
|--------|---------|
| `masterconcept.ai` | **Final production** |
| `site.masterconcept.ai` | Trial (current WordPress site, also used for new stack trial) |
| `site.mcai.dev` | Trial / dev (build new structure here first) |
| `mcai.dev` | Other uses |

**Strategy:** Build and test on `site.mcai.dev` first → then trial on `site.masterconcept.ai` → final production on `masterconcept.ai`.

---

## Architecture Overview

### Front Door

> **Firebase App Hosting** (All GCP)
> Domains: `masterconcept.ai` (production) / `site.mcai.dev` (trial)

### Routing

| Path | Destination |
|------|-------------|
| `/blog/*` | WordPress on SiteGround (during transition) |
| `/wp-admin/*` | WordPress on SiteGround (during transition) |
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

### Payload CMS API Consumers

| Consumer | How |
|----------|-----|
| **Claude Code** | Vibe coding + file upload via Local API |
| **Admin Users** | Edit content in browser at `/admin` |
| **WebMCP** (future) | AI agents via MCP protocol |

### Data Layer (All GCP)

| Service | Stores |
|---------|--------|
| **Cloud SQL PostgreSQL** | Pages, Posts, Categories, Tags, Users, Forms, SEO, Navigation, Settings |
| **GCP Cloud Storage** (Firebase Storage) | Images (CDN), Ebooks/PDFs, Downloads, Auto-thumbnails |

### 3-Layer Guardrails

| Layer | Tool | Constrains |
|-------|------|------------|
| UI Design | shadcn/ui | Consistent components, prevents messy UI |
| Content Structure | Payload CMS | Schema-enforced content types |
| Code Patterns | CLAUDE.md / .cursorrules | Folder structure, naming, best practices |

---

## Context

Current site runs on WordPress + Elementor on SiteGround (GoGeek shared hosting). The site has two parts:
- **Blog** - managed by marketing team
- **Landing pages** - managed by developer (Tommy)

Problems with current setup:
- Elementor is not vibe-codeable (content stored as serialized DB data, not code files)
- AI cannot easily read, edit, or generate Elementor pages
- Performance issues and plugin conflicts (e.g. ElementsKit shortcode rendering failures)
- No path to WebMCP integration

**Goal:** Migrate all pages to a modern, AI-friendly stack. All content (pages, blog, media, files) managed via Payload CMS admin panel. Landing page design is vibe-coded. Blog stays on WordPress during transition. Same domain, no broken links, no SEO impact. **Trial first on `site.mcai.dev`**, then migrate production.

---

## Target Architecture

```
Firebase App Hosting (front door, site.mcai.dev → masterconcept.ai)
  ├── /blog/*       → proxy to WordPress on SiteGround (during transition)
  ├── /wp-admin/*   → proxy to WordPress on SiteGround (during transition)
  ├── /admin/*      → Payload CMS admin panel
  ├── /api/*        → Payload REST + GraphQL API
  └── /*            → Next.js app (landing pages + eventually blog)

All GCP (covered by credits):
  ├── Cloud SQL PostgreSQL  → Payload database
  └── GCP Cloud Storage     → Media / files (CDN served)
      ├── Images
      ├── Ebooks / PDFs (downloadable)
      └── Any uploaded files
```

---

## Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| Framework | **Next.js 15** (App Router, TypeScript) | Frontend + backend in one app |
| CMS | **Payload CMS 3.x** | Content management, admin panel, API, auth |
| Design System | **shadcn/ui** | Consistent UI components for vibe coding |
| Styling | **Tailwind CSS** | Utility-first CSS via shadcn theming |
| Storage | **GCP Cloud Storage** (Firebase Storage) | All media/files, served via CDN |
| Database | **Cloud SQL PostgreSQL** (GCP native, covered by GCP credits) | Payload data storage - same GCP network as Firebase for low latency |
| Hosting | **Firebase App Hosting** | Auto-deploy from Git, Cloud Run behind the scenes |
| Forms | **HubSpot** (existing) | Lead capture, embedded via block component |
| AI Guardrails | **CLAUDE.md / .cursorrules** | Enforces code patterns for structured vibe coding |
| Blog (transition) | **WordPress on SiteGround** | Until blog migrated to Payload |

---

## Payload CMS Feature Overview

### Core Architecture
- Lives **inside** your Next.js app (not a separate service)
- 100% TypeScript with full type safety
- Config-based: everything defined in code, version controlled
- Free & open source (MIT license)

### Content Modeling
- **Collections**: Pages, Posts, Media, Categories, Tags, Users, Forms, etc.
- **Globals**: Site Settings, Navigation, Footer (singleton data)
- **Blocks Field**: Modular content blocks (like Gutenberg but code-defined)
- **30+ field types**: text, richText, number, date, relationship, array, group, blocks, tabs, upload, select, checkbox, radio, point, JSON, email, code, and more
- **Conditional logic**: show/hide fields based on other field values
- **Validation**: built-in and custom per-field

### Admin Panel (`/admin`)
- **Auto-generated UI** from your config - no manual admin building
- **Live Preview**: see page changes in real-time as you edit (side-by-side iframe)
- **Version History**: full diff viewer, one-click restore to any prior version
- **Drafts & Autosave**: publish/unpublish workflow, auto-saves without affecting live content
- **Bulk operations**: bulk select, edit, delete
- **Dark mode**: built-in
- **White-labeling**: custom logo, colors, favicon - fully rebrandable
- **Custom views**: add custom React components to admin panel

### Rich Text Editor (Blog Writing)
- **Lexical-based editor** (modern, customizable)
- Bold, italic, headings, lists, links, images, embeds
- Inline and block-level embeds
- Custom features and nodes can be added
- Similar experience to WordPress Gutenberg for marketing team

### API (3 Ways to Access Content)
- **REST API** (auto-mounted at `/api`) - full CRUD, pagination, filtering, sorting
- **GraphQL API** (auto-mounted at `/api/graphql`) - full CRUD queries & mutations
- **Local API** - direct database access from server code (no HTTP overhead, fully typed)
- **Claude Code uses Local API** to create pages, upload files, seed content

### Media Management
- **Upload collections**: any collection can handle file uploads
- **Image resizing**: auto-generates multiple sizes (thumbnail, medium, large, etc.)
- **Image cropping**: built-in crop tool in admin UI
- **Focal point selector**: set focal point, all sizes re-crop around it
- **Cloud storage**: official plugin for GCP Cloud Storage (`@payloadcms/storage-gcs`)
- **All uploads go through Payload API** → tracked in database + stored in GCP Cloud Storage

### Page & Content Hierarchy
- **Parent/child pages** via Nested Docs plugin (breadcrumbs, tree structure)
- **Categories & Tags** as relationship collections (like WordPress)
- **Any custom taxonomy** you need (not limited to categories/tags)

### Authentication & Access Control
- **Built-in email/password auth** with JWT tokens
- **Role-based access control (RBAC)**: Admin, Editor, Author, custom roles
- **Document-level access**: per-operation (create, read, update, delete)
- **Field-level access**: individual fields can have their own rules
- **API keys** for machine-to-machine auth
- **CMS Login URL**: `https://site.mcai.dev/admin` (auto-generated login page)
- **Password security**: bcrypt hashing, account lockout, password reset via email

### User Roles Plan

| Role | Who | Permissions |
|------|-----|-------------|
| **Admin** | Tommy | Full access - everything |
| **Editor** | Marketing team | Create/edit/publish posts & pages, upload media |
| **Author** | Blog writers | Create/edit own posts only, upload media |
| **API Key** | Claude Code | Create/edit pages, upload media via API (no browser login) |

Claude Code authenticates via API Key stored in `.env`:
```
PAYLOAD_API_KEY=your-secret-key-here
```

### SEO
- **SEO Plugin** (`@payloadcms/plugin-seo`): meta title, description, Open Graph fields
- Injected into any collection automatically
- Works with Next.js metadata API for full control

### Forms
- **Form Builder Plugin** (`@payloadcms/plugin-form-builder`): create forms in admin panel
- Submission storage in database, viewable in admin
- Email notifications on submission
- **HubSpot integration**: use `afterChange` hooks to POST submissions to HubSpot API
- **Or embed HubSpot forms directly** via a HubSpotFormBlock component

### Redirects
- **Redirects Plugin** (`@payloadcms/plugin-redirects`): manage URL redirects in admin
- 301/302/307/308 support
- Critical for migration: preserve old WordPress URLs

### Localization (if needed)
- Built-in, field-level localization
- Unlimited locales

### Email
- Built-in SMTP email support (via nodemailer)
- Used for password resets, form notifications, any transactional email

### Webhooks / Integrations
- **Hooks system**: `beforeChange`, `afterChange`, `beforeDelete`, `afterDelete` on any collection
- Use hooks to: sync to HubSpot, trigger rebuilds, notify Slack, fire webhooks to any URL

---

## Official Payload Plugins

| Plugin | What It Does |
|--------|-------------|
| `@payloadcms/plugin-seo` | Meta title, description, Open Graph, structured data fields |
| `@payloadcms/storage-gcs` | Store uploads in GCP Cloud Storage |
| `@payloadcms/plugin-form-builder` | Admin-built forms, submission storage, email notifications |
| `@payloadcms/plugin-redirects` | URL redirect management (critical for migration) |
| `@payloadcms/plugin-nested-docs` | Parent/child page hierarchy with breadcrumbs |
| `@payloadcms/plugin-search` | Fast search collection sync |
| `@payloadcms/plugin-stripe` | Stripe payment integration |
| `@payloadcms/plugin-multi-tenant` | Multi-site / SaaS tenant isolation |
| `@payloadcms/plugin-sentry` | Error tracking |

---

## 3-Layer Guardrail System (Structured Vibe Coding)

| Layer | Tool | Purpose |
|-------|------|---------|
| Design System | shadcn/ui | All UI must use these components - prevents messy/inconsistent UI |
| Content & Pages | Payload CMS | All content managed via admin panel + API |
| AI Rules | CLAUDE.md + .cursorrules | Enforces code patterns, folder structure, naming conventions |

---

## How Landing Pages Work: Code + CMS Hybrid

### What AI vibe-codes (code files in Git):
- Block component design & layout (React + shadcn/ui + Tailwind)
- Page templates (how blocks are arranged)
- New block types
- Animations, interactions

### What you edit in `/admin` (no code):
- Page title, slug, cover image, meta description
- Text content, headings, CTAs
- Which blocks to use and their order
- Upload images/files
- SEO fields
- Publish / draft / schedule

### What Claude Code can do in one shot:
1. Create new block components (design)
2. Upload images/files to GCP Cloud Storage via Payload API
3. Create pages with content in CMS via Payload Local API
4. Page is live - fine-tune in `/admin` if needed

---

## File & Media Management

### Storage Architecture
```
All uploads → Payload API → GCP Cloud Storage (Firebase Storage)
                  │
                  └→ Database record created
                     (filename, size, dimensions, alt text,
                      auto-generated thumbnails, linked to pages)
```

### Who Can Upload
| Who | How | Tracked in CMS? |
|-----|-----|-----------------|
| Claude Code | Payload Local API (`payload.create({ collection: 'media' })`) | Yes ✅ |
| Admin user | Drag & drop in `/admin` → Media | Yes ✅ |
| **Never** upload directly to Cloud Storage - always go through Payload API |

### File Types
- Images (jpg, png, webp, svg) → auto-generates thumbnails
- PDFs / Ebooks → downloadable links
- Any file type you configure

---

## HubSpot Integration

| Feature | How |
|---------|-----|
| Embed HubSpot forms | HubSpotFormBlock component (paste Portal ID + Form ID in admin) |
| Sync form submissions to HubSpot | Payload `afterChange` hook on form-submissions collection |
| Ebook download with HubSpot form | HubSpotFormBlock + link to PDF in Payload Media |

---

## Domain Routing (Split Architecture)

**Firebase Hosting as the front door** using `firebase.json` rewrites:
- `/blog/**` → proxy to WordPress on SiteGround (during transition)
- `/wp-admin/**` → proxy to WordPress on SiteGround (during transition)
- `/**` (everything else) → Next.js + Payload app on Firebase

**Result:** Single domain, visitors see no difference.

---

## Cross-Linking Between Blog & Landing Pages

| Scenario | Solution |
|----------|----------|
| Blog post links to landing page | Normal links (`/pricing`) - same domain |
| Landing page shows blog post list | BlogListBlock fetches from WordPress REST API (transition) → later from Payload |
| Keep original landing page URLs | Same slug in Payload CMS |
| Images from WordPress | `next/image` remote patterns (transition) → later all in GCP Cloud Storage |

---

## Migration Phases

### Phase 1: Set Up New Stack
- Initialize Next.js 15 project with TypeScript
- Install Payload CMS 3.x (inside Next.js app)
- Install shadcn/ui + Tailwind CSS
- Set up GCP Cloud Storage via `@payloadcms/storage-gcs`
- Set up Cloud SQL PostgreSQL instance on GCP (db-f1-micro, same region as Firebase)
- Set up Firebase App Hosting with Git auto-deploy
- Create CLAUDE.md with project rules
- Install Payload plugins: SEO, Nested Docs, Redirects, Form Builder
- Define collections: Pages, Posts, Media, Categories, Tags, Navigation, Site Settings
- Build shared layout (header, footer, nav) using screenshot-to-code

### Phase 2: Configure Domain Routing
- Set up Firebase Hosting rewrites in `firebase.json`
- Route `/blog/*` and `/wp-admin/*` to SiteGround WordPress
- Route everything else to Next.js + Payload
- Test cross-linking between systems
- Set up redirects plugin for any URL changes

### Phase 3: Migrate Landing Pages (One by One)
- For each Elementor landing page:
  1. Screenshot the page
  2. Use screenshot-to-code → React/Tailwind block components
  3. Register new block types in Payload if needed
  4. Upload images via Payload API → GCP Cloud Storage
  5. Create page in Payload admin, set same slug
  6. Arrange blocks, fill in content
  7. Test the page
- Migrate HubSpot form pages (create HubSpotFormBlock)
- Migrate ebook/download pages (upload files to Payload Media)

### Phase 4: Migrate Blog (Optional - When Ready)
- Export WordPress posts via REST API
- Import into Payload Posts collection (title, content, categories, tags, images)
- Marketing team switches from wp-admin to `/admin`
- Migrate WordPress media to GCP Cloud Storage via Payload API
- Remove WordPress proxy rules from Firebase
- Downgrade/cancel SiteGround

### Phase 5: Cleanup & Optimization
- Performance testing (expect major speed improvement)
- Verify all SEO: meta tags, structured data, sitemaps, redirects
- Set up caching headers
- Configure Payload version history retention
- Set up user roles (Admin for you, Editor for marketing team)

---

## Hosting & Cost

**All GCP services covered by your free GCP credits.**

| Service | Role | Cost (after credits) |
|---------|------|------|
| Firebase App Hosting (Blaze) | Next.js + Payload + front door | ~$5-10/month |
| Cloud SQL PostgreSQL | Payload database | ~$7-10/month (db-f1-micro) |
| GCP Cloud Storage | Media/files | ~$1-2/month (pay per GB) |
| SiteGround GoGeek | WordPress blog (during transition) | ~$35/month (existing) |
| **After full migration** | All GCP only | **~$15-25/month total** |
| **With GCP credits** | Everything covered | **$0 while credits last** |

**All under one GCP console + billing. No external database accounts needed.**

---

## Converting Elementor Pages to Code

1. **Screenshot-to-code** (github.com/abi/screenshot-to-code) - screenshot existing pages, AI generates React/Tailwind block components
2. **Images** - upload to Payload Media (→ GCP Cloud Storage) via API
3. **Header/Footer** - rebuild once as React components
4. **Popups** - rebuild as React modal components
5. **HubSpot forms** - embed via HubSpotFormBlock
6. **Ebooks/downloads** - upload to Payload Media, link in CTA blocks

---

## Payload CMS vs WordPress Comparison

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
| SEO | Yoast/RankMath (mature) | SEO plugin (meta, OG - simpler but sufficient) |
| Forms | CF7/Gravity Forms | Form Builder plugin + HubSpot embed |
| Access control | 5 fixed roles | Custom RBAC, document + field level |
| API | REST (+ GraphQL plugin) | REST + GraphQL + Local API (all built-in) |
| Performance | 180-250ms TTFB | 50-120ms TTFB |
| Plugin ecosystem | 60,000+ plugins | ~50 official + community (growing) |
| AI/vibe coding | Not possible | Native (code-first, TypeScript) |
| Type safety | None (PHP) | Full TypeScript end-to-end |

---

## AI-Automated SEO

This stack enables AI to automate SEO tasks that were impossible with WordPress + Elementor:

| SEO Task | How AI Automates It |
|----------|-------------------|
| **Meta titles & descriptions** | Claude reads page content via Payload API → generates optimized meta → writes to SEO plugin fields |
| **Open Graph images** | Auto-generate OG images using Next.js `ImageResponse` API per page |
| **Alt text for images** | Claude generates descriptive alt text when uploading images via Payload API |
| **Internal linking** | Claude reads all pages/posts via Payload API → suggests/adds relevant internal links |
| **Structured data (JSON-LD)** | Claude generates schema markup (Product, FAQ, Article, etc.) based on page content |
| **Sitemap** | Next.js auto-generates `sitemap.xml` from all Payload pages/posts |
| **Content optimization** | Claude analyzes existing pages → suggests keyword/readability improvements |
| **Redirect management** | Claude uses Payload Redirects plugin to bulk-manage 301s during migration |
| **Bulk SEO audit** | Claude reads all pages via API → flags missing meta, broken links, missing alt text |
| **Blog post SEO** | Claude reviews draft posts → suggests title/description/heading improvements before publish |

**Why this works:** All content is accessible via Payload API (REST/GraphQL/Local). Claude Code can read every page, analyze it, and update SEO fields programmatically - no manual page-by-page editing.

**Example workflow:**
```
You: "Audit all landing pages for SEO and fix issues"

Claude Code:
1. Fetches all pages via Payload Local API
2. Checks each page for: missing meta description, missing alt text,
   no internal links, missing structured data
3. Generates and writes fixes directly to Payload
4. Reports: "Fixed 12 pages: added meta descriptions (8),
   alt text (15 images), structured data (5 pages)"
```

---

## WebMCP Readiness

This stack is **already structured for WebMCP**. Here's why:

### What exists now (ready to expose as MCP tools):
- **Payload REST API** (`/api/pages`, `/api/posts`, `/api/media`) - full CRUD
- **Payload GraphQL API** (`/api/graphql`) - flexible queries
- **Next.js API routes** - custom endpoints for any logic

### When WebMCP matures, you just add an MCP server layer:
```
MCP Server (Next.js API route)
  ├── Tool: "create-page"      → calls Payload Local API
  ├── Tool: "update-content"   → calls Payload Local API
  ├── Tool: "upload-media"     → calls Payload Local API
  ├── Tool: "publish-post"     → calls Payload Local API
  ├── Tool: "get-analytics"    → calls analytics API
  └── Tool: "audit-seo"       → reads all content, returns report
```

**AI agents (Claude, ChatGPT, etc.) could then:**
- Create and publish pages on your site via natural language
- Update content across multiple pages at once
- Upload and manage media
- Run SEO audits and apply fixes
- All through MCP protocol - no custom integration needed

The APIs already exist in Payload. WebMCP just wraps them in the MCP protocol format.

---

## Future Enhancements

- **WebMCP server:** Wrap Payload APIs as MCP tools when protocol matures
- **AI SEO automation:** Scheduled SEO audits and auto-optimization
- **Payload Live Preview:** Real-time editing preview (side-by-side)
- **Full blog migration:** Move all blog content from WordPress to Payload
- **Cancel SiteGround:** Once everything runs on Firebase + Payload
- **Multi-language:** Payload's built-in localization if needed
- **AI content generation:** Claude generates blog drafts, landing page copy via Payload API

---

## Key URLs for Reference

- Payload CMS demo: https://demo.payloadcms.com/admin/login
- Payload CMS docs: https://payloadcms.com/docs
- Payload vs WordPress: https://payloadcms.com/compare/wordpress
- Payload website template: https://github.com/payloadcms/template-website
- Payload plugins: https://payloadcms.com/docs/plugins/overview
- screenshot-to-code: https://github.com/abi/screenshot-to-code
- shadcn/ui: https://ui.shadcn.com
- Firebase App Hosting docs: https://firebase.google.com/docs/app-hosting
- GCS Storage plugin: https://payloadcms.com/docs/plugins/cloud-storage
