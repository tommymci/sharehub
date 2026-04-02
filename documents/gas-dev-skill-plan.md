# Claude Skill: `gas-dev` — Google Apps Script Development Assistant

## Table of Contents

- [Overview](#overview)
- [Framework Overview](#framework-overview)
- [1. Prerequisites](#1-prerequisites)
  - [1.1 CLI Tools](#11-cli-tools)
  - [1.2 Google Cloud Platform Setup](#12-google-cloud-platform-setup-one-time-manual)
  - [1.3 Authentication & Multi-Account Management](#13-authentication--multi-account-management)
- [2. Project Structure](#2-project-structure)
  - [2.1 Standard File Layout](#21-standard-file-layout)
  - [2.2 Mirrored Folder Structure (Local ↔ Google Drive)](#22-mirrored-folder-structure-local--google-drive)
  - [2.3 .claspignore](#23-claspignore-always-include)
  - [2.3 accounts.json Structure](#23-accounts-json-structure)
  - [2.4 Real Project Examples](#24-real-project-examples)
  - [2.5 Standalone vs Sheet-Bound Apps Script](#25-standalone-vs-sheet-bound-apps-script)
- [3. Development Workflow](#3-development-workflow)
  - [3.1 Creating a New Project](#31-creating-a-new-project)
  - [3.2 Local Development](#32-local-development)
  - [3.3 Deployment](#33-deployment)
- [4. Coding Preferences](#4-coding-preferences)
  - [4.1 File Organization](#41-file-organization)
  - [4.2 Backend Patterns](#42-backend-patterns-codejs)
  - [4.3 Key Gotchas](#43-key-gotchas)
  - [4.4 Frontend Patterns](#44-frontend-patterns)
- [5. Skill File Structure](#5-skill-file-structure)
- [6. Verification](#6-verification)

---

## Overview

A Claude skill that encodes the full Google Apps Script (GAS) development workflow — from project creation to deployment — so Claude can assist efficiently with any GAS project without re-learning the setup each time.

- **Scope:** Global (`~/.claude/skills/gas-dev/`)
- **Trigger:** Auto-triggered when working with Apps Script, clasp, Google Sheets apps, or web app deployment
- **Location:** `/Users/tommy/.claude/skills/gas-dev/`

---

## Framework Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    GAS Development Framework                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  SETUP   │───▶│  DEVELOP │───▶│   TEST   │───▶│  DEPLOY  │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│       │               │               │               │         │
│       ▼               ▼               ▼               ▼         │
│  GCP Project     Code.js          Local Dev       clasp push    │
│  OAuth Creds     index.html       Mock Data       clasp deploy  │
│  clasp login     styles.html      http-server     Same URL!     │
│  accounts.json   components       Polyfill                      │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INFRASTRUCTURE                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    accounts.json                         │    │
│  │  (Central Registry: projects, accounts, credentials)     │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                          │
│       ├── .accounts/<email>/     (per-account credentials)       │
│       ├── tools/gws-auth/        (OAuth flow + API client)       │
│       ├── tools/gws-client.js    (Sheets/Drive API wrapper)      │
│       └── tools/file-picker/     (Drive file whitelist UI)       │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ARCHITECTURE (per project)                                      │
│                                                                  │
│  ┌─ LOCAL ──────────────┐    ┌─ GOOGLE CLOUD ──────────────┐   │
│  │                      │    │                              │   │
│  │  ProjectName/        │    │  Google Drive/App/           │   │
│  │  ├── Code.js    ─────┼──▶│  ├── ProjectName (GAS)       │   │
│  │  ├── index.html      │    │  ├── ProjectName (Sheet)     │   │
│  │  ├── styles.html     │    │  └── assets/                 │   │
│  │  ├── components.html │    │                              │   │
│  │  ├── server.js  ───┐ │    │  Apps Script Runtime         │   │
│  │  ├── mock-data.json│ │    │  ├── doGet() → HtmlService   │   │
│  │  └── .clasp.json   │ │    │  ├── Sheets API (database)   │   │
│  │                     │ │    │  ├── PropertiesService       │   │
│  │  localhost:PORT  ◀──┘ │    │  ├── MailApp (email)         │   │
│  │  (dev server)         │    │  └── DriveApp (files)        │   │
│  └───────────────────────┘    └──────────────────────────────┘   │
│                                                                  │
│           clasp push ──────▶ sync code to GAS                    │
│           clasp deploy -i ─▶ update SAME web app URL             │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FRONTEND PATTERNS (iframe-compatible SPA)                       │
│                                                                  │
│  ┌─ Hash Router ───────────────────────────────────────────┐    │
│  │  #form ──▶ #success ──▶ #admin ──▶ #dashboard           │    │
│  │    │                       │            │                │    │
│  │    │    localStorage       │   password │  Sheets data   │    │
│  │    │    (persist state)    │   check    │  (cached)      │    │
│  │    ▼                       ▼            ▼                │    │
│  │  Form View          Admin Login    Dashboard View        │    │
│  │  ├── Contact Info                  ├── Summary (charts)  │    │
│  │  ├── Date Picker                   ├── Submissions       │    │
│  │  ├── AI Questions                  └── Settings          │    │
│  │  └── Submit → email                                      │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Back button ✓  Refresh ✓  iframe ✓  Mobile ✓                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```
MULTI-ACCOUNT FLOW

  ┌──────────────┐         ┌──────────────┐
  │ Account A    │         │ Account B    │
  │ user@co.com  │         │ user@gmail   │
  └──────┬───────┘         └──────┬───────┘
         │                        │
         ▼                        ▼
  .accounts/user@co.com/   .accounts/user@gmail/
  ├── .clasprc.json        ├── .clasprc.json
  ├── gws-credentials      ├── gws-credentials
  ├── gws-token.json       ├── gws-token.json
  └── allowed-files.json   └── allowed-files.json
         │                        │
         └────────┬───────────────┘
                  ▼
           accounts.json
           (central registry)
                  │
         ┌────────┴────────┐
         ▼                 ▼
    Project A          Project B
    (Account A)        (Account B)
```

---

## 1. Prerequisites

### 1.1 CLI Tools

| Tool | Install | Purpose |
|------|---------|---------|
| Node.js (v18+) | `brew install node` | Runtime for local dev servers and scripts |
| npm | Bundled with Node.js | Package management |
| npx | Bundled with npm | Run one-off packages (e.g. http-server) |
| clasp | `npm install -g @google/clasp` | Google Apps Script CLI for push/deploy |

**Note:** `gcloud` CLI is NOT required. All GCP configuration is done via the web console.

### 1.2 Google Cloud Platform Setup (One-time, Manual)

All steps below are done in [console.cloud.google.com](https://console.cloud.google.com) — they cannot be automated via CLI.

**Step 1: Create a GCP Project**
- GCP Console → New Project → name it (e.g. "claude-integration")
- Note the **Project Number** (used later to link Apps Script)

**Step 2: Enable APIs**
- GCP Console → APIs & Services → Library
- Search and enable each:
  - Google Sheets API
  - Google Drive API
  - Apps Script API
  - Gmail API (only if using MailApp with email aliases)

**Step 3: Configure OAuth Consent Screen**
- GCP Console → APIs & Services → OAuth consent screen
- Choose "External" (or "Internal" for Google Workspace orgs)
- Add required scopes: `spreadsheets`, `drive`, `script.projects`
- Add test users if in "Testing" mode

**Step 4: Create OAuth Credentials**
- GCP Console → APIs & Services → Credentials → Create Credentials → OAuth Client ID

| Type | Use | Save as |
|------|-----|---------|
| Desktop application | CLI/Node.js tools (clasp, gws-client) | `gws-credentials.json` |
| Web application | Browser-based file picker UI | `gws-web-credentials.json` |

- Download both JSON files and store in `.accounts/<email>/`

**Step 5: Link Apps Script to GCP (per project)**
- Open Apps Script Editor for the project
- Project Settings → Google Cloud Platform (GCP) Project
- Enter the **GCP Project Number**
- This enables `clasp logs` and advanced API access

### 1.3 Authentication & Multi-Account Management

Each Google account gets its own credential folder:

```
.accounts/
├── user1@gmail.com/
│   ├── .clasprc.json          # clasp auth token
│   ├── gws-credentials.json   # OAuth desktop credentials
│   ├── gws-web-credentials.json # OAuth web credentials
│   ├── gws-token.json         # Access/refresh tokens (auto-generated)
│   └── allowed-files.json     # Whitelisted Drive file IDs
├── user2@company.com/
│   └── ... (same structure)
```

**Central registry:** `accounts.json` at project root maps every project to its account, scriptId, sheetId, deploymentId, and webAppUrl.

**Setup steps per account:**
1. `clasp login` → stores token in `~/.clasprc.json`
2. Copy to account folder: `cp ~/.clasprc.json .accounts/<email>/.clasprc.json`
3. Run GWS auth: `cd tools/gws-auth && node auth.js <email>` → generates `gws-token.json`
4. Run file picker: `cd tools/file-picker && npm start` → manage `allowed-files.json`

**Switching accounts:** `cp .accounts/<email>/.clasprc.json ~/.clasprc.json`

**Multiple accounts can share the same GCP project** for OAuth credentials.

---

## 2. Project Structure

### 2.1 Standard File Layout

**Simple project (form, landing page):**
```
ProjectName/
├── .clasp.json           # scriptId, rootDir
├── .claspignore          # exclude local dev files
├── appsscript.json       # runtime config, webapp settings
├── Code.js               # backend: doGet, CRUD, email
├── Index.html            # main UI entry
└── styles.html           # CSS (separate file)
```

**Full-featured project (dashboard, SPA):**
```
ProjectName/
├── .clasp.json
├── .claspignore
├── appsscript.json
├── Code.gs               # backend
├── index.html            # entry point with includes
├── styles.html           # all CSS
├── header.html           # nav/header component
├── forms.html            # form components
├── dashboard.html        # dashboard view
├── server.js             # local dev server (NOT deployed)
├── package.json          # local dev dependencies
├── mock-data.json        # synced Sheet data for local testing
├── .settings.json        # local settings cache
└── node_modules/         # NOT deployed
```

### 2.2 Mirrored Folder Structure (Local ↔ Google Drive)

Local project folders and Google Drive folders should mirror each other:

```
LOCAL:                              GOOGLE DRIVE:
~/Project/Appscript/                My Drive/App/
├── ProjectA/                       ├── ProjectA/
│   ├── .clasp.json                 │   ├── ProjectA (Apps Script)
│   ├── Code.js                     │   ├── ProjectA (Google Sheet)
│   ├── Index.html                  │   └── (other assets)
│   └── ...                         │
├── ProjectB/                       ├── ProjectB/
│   └── ...                         │   └── ...
├── .accounts/                      │
├── accounts.json                   │
└── tools/                          │
```

- Each project has a local folder AND a Drive folder with the same name
- The Drive folder contains: the Apps Script file, the Google Sheet, and any uploaded assets
- When creating a new project, create both the local folder and Drive folder
- Store the Drive folder ID as `driveFolderId` in accounts.json

### 2.3 .claspignore (always include)

```
node_modules/**
server.js
package.json
package-lock.json
mock-data.json
.settings.json
.clasp.json
.claspignore
```

### 2.3 accounts.json Structure

```json
{
  "projects": {
    "ProjectName": {
      "account": "user@domain.com",
      "scriptId": "...",
      "sheetId": "...",
      "deploymentId": "...",
      "webAppUrl": "https://script.google.com/macros/s/.../exec",
      "webAppAccess": "Anyone_Anonymous",
      "driveFolderId": "...",
      "note": "Deploy: clasp push && clasp deploy -i <deploymentId>"
    }
  },
  "accounts": {
    "user@domain.com": {
      "clasp": ".accounts/user@domain.com/.clasprc.json",
      "gws": ".accounts/user@domain.com/gws-credentials.json",
      "gcpProject": "project-name"
    }
  },
  "default": "user@domain.com"
}
```

> **Note:** The accounts.json format may evolve as more projects are added. The skill should always read the current accounts.json to understand available projects rather than hardcoding assumptions about its structure.

### 2.4 Real Project Examples

**Google Drive root folder:** [App](https://drive.google.com/drive/folders/1KDBskPXvLC8cYzPbrb58Q_x6veD3V8_X?usp=drive_link)

| Project | Account | GAS Type | Web App Access | Drive Folder |
|---------|---------|----------|---------------|-------------|
| **PayU** | hkmci.com | Standalone | Anyone | App/PayU/ |
| **Master Draw** | hkmci.com | Sheet-bound | Anyone | App/Master Draw/ |
| **Master Form** | hkmci.com | Standalone | Anyone_Anonymous | App/Master Form/ |

**Deployment & Embed Links:**

| Project | Apps Script Web App | iframe Embed |
|---------|-------------------|--------------|
| **PayU** | [Deploy Link](https://script.google.com/macros/s/AKfycbziWvHyUUoVbdFMMSqKUdfQyjvXlQSvkA8BPwl7xKFEgwZWKF0MfjMdi06ZvW47Su_UKQ/exec) | [mcai.dev/payu](https://mcai.dev/payu/) |
| **Master Draw** | [Deploy Link](https://script.google.com/macros/s/AKfycbwf3jwQsQQoYIBGyG3_uoIUO_vtLfHkSlZ23bbKcBoKMaXwbDVsbkTD8ivX_wFEoGPgdA/exec) | [masterconcept.ai/link/master-draw](https://masterconcept.ai/link/master-draw/) |
| **Master Form** | [Deploy Link](https://script.google.com/macros/s/AKfycbzOOyRuaWedAIpTcXlZvPjb3J_KNBqUAJz-_EdRZKVKhJbb-atCce2H8NeNDfKNJoDTOA/exec) | [masterconcept.ai/link/eohk](https://masterconcept.ai/link/eohk) |

### 2.5 Standalone vs Sheet-Bound Apps Script

There are two types of Apps Script projects:

**Standalone GAS** (e.g. PayU, Master Form)
- Created via `clasp create` — produces an independent `.gs` file in Drive
- Can be moved freely between Drive folders via API
- Has its own scriptId, separate from any Sheet
- Appears as a separate file in Drive folder alongside the Sheet

**Sheet-Bound GAS** (e.g. Master Draw)
- Created from within a Google Sheet (Extensions → Apps Script)
- The script is embedded inside the Sheet — no separate file in Drive
- Cannot be moved independently via Drive API (move the Sheet instead)
- `parents` returns `undefined` when querying via Drive API
- Access via: open Sheet → Extensions → Apps Script
- The scriptId still works with clasp for push/deploy

**How to tell which type:**
- Query `drive.files.get(scriptId, { fields: 'parents' })`
- If `parents` is `undefined` → likely **sheet-bound**
- If `parents` has a folder ID → **standalone**

**When to use which:**
- **Standalone** — when the web app is the primary product, Sheet is just the database
- **Sheet-bound** — when the Sheet is the primary product, script adds functionality to it

---

## 3. Development Workflow

### 3.1 Creating a New Project

1. Create Google Sheet via Sheets API (using `gws-client.js`)
2. Create Drive folder for the project
3. `clasp create --title "ProjectName"` in project folder (creates standalone GAS)
4. Move GAS file to Drive folder via Drive API
5. Set up `appsscript.json` with timezone, webapp config
6. Register in `accounts.json` with all IDs
7. Create boilerplate: Code.js, Index.html, styles.html, .claspignore
8. First push + deploy: `clasp push && clasp deploy`
9. Save deploymentId to accounts.json

### 3.2 Local Development

**For simple projects:** Use `npx http-server -p 3850 -c-1` to serve files locally with no caching.

**For projects needing API access:** Create `server.js` with:
- Express server on custom port
- `<?!= include('filename') ?>` tag processing (reads local HTML files)
- `google.script.run` polyfill using Proxy pattern → routes to `/api/<functionName>` endpoints
- API endpoints that mirror Code.js functions using `gws-client.js`
- Template expression replacement (e.g. `<?= getAppIcon() ?>`)

**Mock data pattern:**
- `tools/sync-mock-data.js` — pulls live Sheet data into `mock-data.json`
- Local server reads `mock-data.json` for instant responses
- Run `node tools/sync-mock-data.js` to refresh local data

### 3.3 Deployment

**CRITICAL RULE: Always deploy to the same URL.**

```bash
# Switch account if needed
cp .accounts/<email>/.clasprc.json ~/.clasprc.json

# Push code then deploy to existing deployment
clasp push --force
clasp deploy -i "<deploymentId>" -d "Description"
```

- **NEVER** use `clasp deploy` without `-i` flag (creates new deployment = new URL)
- **ALWAYS** use `clasp deploy -i <deploymentId>` to update existing deployment
- Store deploymentId in `accounts.json` for reference

---

## 4. Coding Preferences

### 4.1 File Organization
- **Break down large HTML** into modular includes: `styles.html`, `header.html`, component files
- Use `<?!= include('filename') ?>` for server-side HTML includes
- **Keep each file focused** — refactor when a file grows beyond manageable size
- **CSS in separate `styles.html`**, not inline in the main page
- **One Code.js/Code.gs** for backend (unless very large, then split by domain)

### 4.2 Backend Patterns (Code.js)

```javascript
// Cached SpreadsheetApp
var _ss = null;
function SS() { if (!_ss) _ss = SpreadsheetApp.openById(SHEET_ID); return _ss; }

// doGet with page routing
function doGet(e) {
  var page = e && e.parameter && e.parameter.page;
  return HtmlService.createHtmlOutputFromFile('Index')
    .setTitle('App Title')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

// Include helper
function include(filename) {
  return HtmlService.createHtmlOutputFromFile(filename).getContent();
}

// Settings via PropertiesService
function getSetting(key) {
  return PropertiesService.getScriptProperties().getProperty(key) || '';
}

// Date formatting (HKT)
var now = Utilities.formatDate(new Date(), 'Asia/Hong_Kong', 'yyyy-MM-dd HH:mm:ss');
```

### 4.3 Key Gotchas

| Issue | Cause | Fix |
|-------|-------|-----|
| `google.script.run` returns null | Can't serialize Date objects or complex objects in ANYONE_ANONYMOUS mode | Convert Dates to ISO strings; return simple strings ('ok') not objects |
| `alert()` blocked | iframe sandbox blocks modals | Use inline error messages instead |
| Phone `+852` shows #ERROR in Sheet | `+` interpreted as formula | Strip non-digits in frontend, or prefix with `'` in backend |
| Form hung on "Submitting..." | Return value can't serialize | Return `'ok'` string, not `{ success: true }` |
| Timestamp wrong timezone | `new Date()` returns UTC in anonymous mode | Use `Utilities.formatDate(new Date(), 'Asia/Hong_Kong', ...)` |

### 4.4 Frontend Patterns

**Hash-based SPA routing (for iframe compatibility):**
- Routes: `#form`, `#success`, `#admin`, `#dashboard`
- `hashchange` event listener for navigation
- `localStorage` to persist current view across refresh
- Success page: use `justSubmitted` flag — don't persist on refresh
- `navigate()` function handles same-hash re-routing

**iframe embedding & branding:**

Apps Script web app URLs are long and ugly (e.g. `script.google.com/macros/s/AKfyc.../exec`). The standard practice is to embed them in an iframe on a branded domain (e.g. `masterconcept.ai/link/eohk` or `mcai.dev/payu`). This has important implications:

- **Web app access MUST be `ANYONE` or `ANYONE_ANONYMOUS`** — if the GAS requires Google login (`MYSELF` or `DOMAIN`), the OAuth prompt cannot render inside an iframe and the app will fail to load
- **For private apps, use password protection instead of Google auth** — implement a client-side password check (hash-based) or server-side via Script Properties, rather than relying on Google login
- **`setXFrameOptionsMode(ALLOWALL)`** is required in doGet for iframe embedding
- **`alert()` / `confirm()` / `prompt()` are blocked** in iframe sandbox — always use inline error messages and custom UI instead
- **`localStorage`** works within the iframe origin for state persistence across refresh
- **Hash routing** creates browser history entries so back button works inside the iframe

**iframe-specific gotchas:**
- `google.script.run` success/failure callbacks may silently fail if the return value can't be serialized — always return simple strings
- Authorization popups from `google.script.run` cannot display inside iframe — all server functions must be pre-authorized by running them once in the script editor
- Form submissions that hang with no response usually mean the function needs re-authorization

**Responsive design:**
- Raleway font + Material Icons Outlined
- Mobile breakpoints: 768px (tablet), 540px (mobile), 360px (small mobile)
- `inputmode="numeric"` for phone fields (shows number keyboard on mobile)
- Font size 16px on mobile inputs (prevents iOS zoom)

---

## 5. Skill File Structure

```
~/.claude/skills/gas-dev/
├── SKILL.md                        # Main skill (auto-triggered)
└── references/
    ├── project-setup.md            # New project creation checklist
    ├── local-dev.md                # Local server, polyfill, mock data
    ├── deployment.md               # Clasp push/deploy, same-URL rule
    ├── frontend-patterns.md        # SPA routing, iframe, localStorage, responsive
    ├── backend-patterns.md         # Code.js patterns, Sheets CRUD, email, gotchas
    ├── gws-client.md               # Google Workspace API usage from Node.js
    └── accounts.md                 # Multi-account setup, accounts.json, credentials
```

### SKILL.md Frontmatter

```yaml
---
name: gas-dev
description: >
  Google Apps Script development assistant. Handles project creation, local dev servers,
  clasp deployment, Google Sheets as database, Script Properties for settings, and
  web app frontend with iframe/SPA support. TRIGGER when: working in an Appscript
  project directory, user mentions Apps Script / GAS / clasp / doGet / google.script.run,
  creates Google Sheets apps, deploys web apps, builds forms or dashboards with
  Google Sheets backend, or asks about Google Workspace API integration.
  Even if user just says "create a form" or "deploy it" in an Appscript context,
  use this skill.
---
```

### SKILL.md Body Sections

1. **Quick Reference** — most common commands (push, deploy, switch account, sync data)
2. **Project Structure** — standard layout, accounts.json, .claspignore
3. **Account Management** — switch accounts, credential paths
4. **Creating a New Project** — step-by-step with API calls
5. **Local Development** — server setup, mock data, polyfill
6. **Deployment Rules** — ALWAYS same URL, push before deploy
7. **Frontend Patterns** — routing, iframe, localStorage, responsive
8. **Backend Patterns** — doGet, Sheets CRUD, email, settings
9. **Coding Preferences** — modular HTML includes, separate styles, keep files short
10. **Common Gotchas** — Date serialization, alert() blocked, return strings not objects
11. **Reference pointers** — when to read each reference file

---

## 6. Verification

1. Check skill appears in Claude Code's available skills list (look for `gas-dev` in system reminder)
2. Test auto-triggering with prompts: "create a new Apps Script project", "deploy my web app", "add a Google Sheet as database"
3. Verify reference files load when relevant (check if Claude reads them during complex tasks)
4. End-to-end test: create project → local dev → push → deploy → verify same URL
