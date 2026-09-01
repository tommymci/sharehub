---
title: "Figma → WordPress Page Builder — Skill Brief"
date: 2026-09-01
---

# Figma → WordPress Page Builder — Skill Brief

A working note of **how we currently build/update `masterconcept.ai` pages from Figma via the WP REST API**, so the skill (like *mcai-webapp*, but targeting WordPress) captures the real gotchas — not just the happy path.

**Site facts:** SiteGround-hosted · **WPML** multilingual (EN / 繁 `zh-hant` / 简 `zh-hans`) · Elementor + ElementsKit · **Permalink Manager Pro** · WP Buddy custom plugin.

---

## 0. Access needed (prerequisites)
- A **WP user with edit + publish page** capability (Editor/Admin) + an **Application Password** for REST auth.
- Access to the **WP Buddy abilities** (via the MCP/REST layer), especially **`mc/set-post-html`**.
- Work **draft → review → publish** (set `status`), not publish-in-one-shot.

## 1. Page body = raw HTML (NOT Elementor)
- Pages are built as **raw HTML** adapted from the Figma export.
- Write it with **`mc/set-post-html`** — the normal REST create/update runs **KSES and strips** `<div>`/`<style>`/structural HTML.
- ⚠️ **Never open these pages with "Edit with Elementor"** — it converts them into an Elementor page and hides the AI/HTML content.
- Inline `<script>` is **stripped on output** and SiteGround **combines JS** → put interactivity in the **enqueued script** (`assets/wpbuddy-page.js`), not inline `<script>`.

## 2. Images / icons from Figma
- **Export from Figma → upload to the WP Media Library** (`/wp/v2/media`) → reference the **uploaded URL** in the HTML. Do **not** hotlink Figma URLs.
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

## 5. "Contact us" CTA = shared popup (don't rebuild the form)
- The CTA opens the **existing shared contact popup** (Elementor/ElementsKit popup) — which has a **different ID per language** (contact-popup family, e.g. `809 / 75990 / 5368 / 3757`).
- **Reuse the existing popup trigger**; the WP Buddy **Header-CTA** tool already maps the right popup + label per language. Don't create a new form.

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
