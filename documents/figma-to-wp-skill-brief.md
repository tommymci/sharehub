---
title: "Figma → WordPress Page Builder — Skill Brief"
date: 2026-09-02
---

# Figma → WordPress Page Builder — Skill Brief

A working note of **how we currently build/update `masterconcept.ai` pages from Figma via the WP REST API**, so the skill (like *mcai-webapp*, but targeting WordPress) captures the real gotchas — not just the happy path.

**Site facts:** SiteGround-hosted · **WPML** multilingual (EN / 繁 `zh-hant` / 简 `zh-hans`) · Elementor + ElementsKit · **Permalink Manager Pro** · WP Buddy custom plugin.

**Runs on:** Claude Code (terminal) **and** Claude Cowork — one skill, same as *mcai-webapp*.
**Website requests come from:** the Asana board → <https://app.asana.com/1/9811672847608/project/1193839356417761/board/1205105526607663>

---

## Features the skill must have
- [ ] **Update an existing AI-built page** — edit/refresh a page already built by the AI builder (swap an image, update a section, change a related-posts block, fix copy).
- [ ] **Build a new page from scratch** — from a Figma design → a full new WP page.
- [ ] **Read the Figma design** via the Figma API (file key + token) — frames, text, assets.
- [ ] **Export images/icons from Figma → rename to good filenames → upload to WP Media Library**; reference the uploaded URLs (re-use existing assets, WebP, mind the upload WAF).
- [ ] **Reserve ~90px top clearance** for the fixed site header on every page.
- [ ] **Write the page body as raw HTML** via `mc/set-post-html` (avoids KSES stripping) — not Elementor.
- [ ] **Insert reusable shortcodes** for dynamic sections — related posts `[wpb_post_list]`, authors `[wpb_authors]`, etc. (never hardcode these).
- [ ] **Use relative links** (`/solutions/…`), not absolute, so WPML can localize.
- [ ] **Wire the "Contact us" CTA** via **`[wpb_hs_form form="2b601d13-6d01-4228-bd3b-d4e794d81f0b"]`** — one HubSpot form for all languages; don't rebuild the form.
- [ ] **Set slug / parent / category**, regenerate the Permalink Manager URI, and add **301 redirects** on any slug change.
- [ ] **Enqueue JS** via the assets script for interactions (inline `<script>` is stripped on output).
- [ ] **Draft → review → publish** (status control).
- [ ] **Purge SiteGround cache** after publishing.
- [ ] **Translation (WPML)** — create + link the other-language versions into one translation group, with language-aware terms. *(scope TBD)*

---

## 0. Access needed (prerequisites)
- A **WP user with edit + publish page** capability (Editor/Admin) + an **Application Password** for REST auth.
- Access to the **WP Buddy abilities** (via the MCP/REST layer), especially **`mc/set-post-html`**.
- Work **draft → review → publish** (set `status`), not publish-in-one-shot.

## 0b. Figma access + API token
**Access:** use the **shared Figma account** (Tommy provides the login) — work inside that account.

**How to get the Figma Personal Access Token** (the skill uses this to read designs/assets):
1. Log into Figma (shared account) → top-left menu / your avatar → **Settings**.
2. Open the **Security** tab.
3. Under **Personal access tokens** → **Generate new token**.
4. Name it (e.g. `wp-skill`), set an expiry, and set **scopes to read-only** — *File content: Read* (and *Library assets: Read* if using components).
5. **Generate token → copy it immediately** (Figma shows it **only once**).
6. Put it in the skill config as an env var / secret, e.g. `FIGMA_TOKEN` — **never commit it or paste it in chat**.

**File key:** it's in the design URL → `figma.com/design/<FILE_KEY>/…`. Share the file URL(s); the key isn't secret.

**Security (shared account):** treat the token as a password. Store it as a secret only; to revoke, go to **Settings → Security → Personal access tokens → delete** the token (and rotate it if the dev leaves or it leaks).

## 1. Page body = raw HTML (NOT Elementor)
- Pages are built as **raw HTML** adapted from the Figma export.
- **Reserve ~90px top clearance** for the fixed site header — add top padding / a spacer to the first section so the hero isn't hidden behind the header.
- Write it with **`mc/set-post-html`** — the normal REST create/update runs **KSES and strips** `<div>`/`<style>`/structural HTML.
- ⚠️ **Never open these pages with "Edit with Elementor"** — it converts them into an Elementor page and hides the AI/HTML content.
- Inline `<script>` is **stripped on output** and SiteGround **combines JS** → put interactivity in the **enqueued script** (`assets/wpbuddy-page.js`), not inline `<script>`.

## 1b. JavaScript / interactivity
- **You CANNOT use inline `<script>` in the page** — it's stripped on output (and SG combines JS). Raw `<script>` pasted into the page **won't run**.
- **JS is not banned** — put interaction code in the enqueued script **`assets/wpbuddy-page.js`** (part of WP Buddy) and trigger it from the HTML via **classes / data-attributes**.
- **Ready-made — no JS to write, just use these classes:**
  - **Tabs** → `.ptab` buttons + `.ppanel` panels, linked by `data-p`.
  - **Accordion (single-open)** → `.acc-head` + `.acc-item`.
- For a **new** interaction not covered, **add it to `wpbuddy-page.js`** in the repo (it deploys with the plugin) — never inline in the page.
- "Dynamic" content (post lists, contact form, embeds) → use the **shortcodes** (`[wpb_post_list]`, `[wpb_hs_form]`, `[wpb_zoom]`), not JS.

## 2. Images / icons from Figma
- **Export from Figma → upload to the WP Media Library** (`/wp/v2/media`) → reference the **uploaded URL** in the HTML. Do **not** hotlink Figma URLs.
- **Rename the exported files to good, descriptive names** (kebab-case, e.g. `geospatial-fleet-hero.webp`) *before* upload — never the default Figma names (`Group 123.png`). Good filenames help SEO and make media re-use easier.
- Watch the **media-upload WAF** (SiteGround can block some uploads) — and **re-use already-uploaded assets** instead of re-uploading duplicates.
- Serve **WebP** + right-sized images (mobile PageSpeed is sensitive to image weight).

## 3. Reusable sections = shortcodes (don't hardcode)
Use the WP Buddy `wpb_` shortcodes so content stays dynamic + language-aware:
- **Related posts / post lists →** `[wpb_post_list preset="…"]` (managed in **WP Buddy → Post Lists**). Don't hardcode post cards.
- **Author list →** `[wpb_authors]`
- **Zoom webinar →** `[wpb_zoom]` / `[zoom_webinar]` · **Menu →** `[wpbuddy_menu]`

## 4. Links = relative, not hardcoded
- Use **site-relative** links (`/solutions/xxx/`), **not** absolute (`https://masterconcept.ai/...`).
- Hardcoded absolute links **break WPML language switching** (and staging). Relative links let WPML localize them.

## 5. "Contact us" CTA = HubSpot form popup shortcode
- **The general Contact-us form is ONE HubSpot form for all languages** — GUID **`2b601d13-6d01-4228-bd3b-d4e794d81f0b`** (same form on the EN / 繁 / 简 contact pages).
- **Use this one line — it works in every language** (renders a button that opens the form in a lightbox popup):
  ```
  [wpb_hs_form form="2b601d13-6d01-4228-bd3b-d4e794d81f0b" text="Contact us" style="primary" align="center"]
  ```
  Defined in WP Buddy `hs-form-popup.php` (alias `[wpb_hubspot_form]`). Generate/verify it in **WP Buddy → Shortcode → HubSpot Form Button**.
- Attributes: `form` (GUID, required) · `text` · `style` primary/secondary/link · `align` · `title` (optional); `portal`/`region` default to MC's HubSpot (`3456479`/`na1`).
- **Don't rebuild a form per page or per language** — same GUID everywhere.
- *(Legacy method on some older pages, e.g. geospatial: a `data-cta="form"` link → `/link/…contact-us-form` that opens the Elementor contact popup — those popups ARE per-language: EN=`809` / 繁=`3757` / 简=`5368`. Prefer the shortcode above.)*

## 6. WPML — the hard part (get this right)
- Each page is a **separate post per language** (EN / 繁 / 简).
- You **must link the language versions into one translation group** (WPML `trid`) — otherwise the **language switcher won't connect them**. Creating 3 pages is **not** enough.
- **Terms (category/tag)** must be the **language-appropriate translation** (resolve via `wpml_object_id`). Assigning an EN term to a 繁 page mis-links it (and can silently render in the wrong language).
- For custom/virtual pages, the **language-switcher URLs may need registering** so switching lands on the translated page (not the home page).

## 7. Slug / category / permalink
- Set the page **slug + parent** for the URL hierarchy (e.g. parent `solutions` → `/solutions/xxx/`).
- URLs are governed by **Permalink Manager Pro** (not just the native slug) — **regenerate its URI** after a slug change.
- Changing a slug → **add 301 redirects** (old → new), **including child pages**.
- Set the page **category/tag** as needed for the permalink/listing structure.

## 8. Publish workflow + cautions
- **Draft → review → publish.**
- ⚠️ **Post revisions are DISABLED site-wide → there is no undo.** Keep the **Figma + exported HTML as the source/backup**.
- After publishing, **purge SiteGround cache** (logged-out visitors are page-cached).

---

## Checklist — what the developer needs
- [ ] **Figma:** shared-account login + a **read-only Personal Access Token** (`FIGMA_TOKEN`) + the file key/URL
- [ ] WP **Application Password** (edit + publish) + REST base URL
- [ ] Access to **`mc/set-post-html`** (and other WP Buddy abilities) via MCP/REST
- [ ] The **`wpb_` shortcode** list + **Post List presets**
- [ ] **Contact-popup IDs per language** + how the trigger works
- [ ] **WPML** language codes (`en`, `zh-hant`, `zh-hans`) + the **translation-linking** method (`trid`)
- [ ] **Permalink Manager Pro** awareness (URI regen + redirects)
- [ ] **Media** upload path + the **WAF** caveat
- [ ] The **brand/style guide** (WP Buddy → Brand Kit / AI page patterns)

---

## Suggested skill flow (draft)
1. Read the Figma design (or its exported HTML/assets).
2. Upload images/icons → Media Library; collect the URLs.
3. Assemble the page HTML (relative links, `wpb_` shortcodes for dynamic bits, shared popup for CTA).
4. Create/update the page **as a draft** via `mc/set-post-html`; set slug + parent.
5. Repeat per language; **link the translations** (WPML `trid`) and map terms via `wpml_object_id`.
6. Regenerate the Permalink Manager URI; add redirects if a slug changed.
7. Review draft → **publish** → **purge SG cache**.
