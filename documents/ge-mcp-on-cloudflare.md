---
title: "GE MCP on Cloudflare — Project Summary"
date: 2026-04-30
---

# GE MCP Integration — Project Summary

Goal: connect Gemini Enterprise (Agentspace) to Alpha Vantage and SEC EDGAR via custom MCP servers, with an OAuth 2.0 proxy in front so GE can authenticate.

## 1. Cloudflare Workers (deployed, all functional)

| Worker | URL | Purpose |
|---|---|---|
| **OAuth Proxy** | https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev | Issues JWTs; routes `/mcp` → Alpha Vantage, `/sec/mcp` → SEC EDGAR via service bindings |
| **Alpha Vantage MCP** | https://alphavantage-mcp.masterconcept-hongkong.workers.dev | 4 financial-data tools (`get_stock_quote`, `get_company_overview`, `search_symbols`, `get_daily_time_series`) |
| **SEC EDGAR MCP** | https://sec-edgar-mcp.masterconcept-hongkong.workers.dev | 4 SEC-filings tools (`search_companies`, `get_company_filings`, `get_company_facts`, `get_filing_text`) |

Cloudflare account: **Master Concept Demo** (`mcmsp.dev`).

### Architecture

```
┌─────────────────────────────────────────────────┐
│  Gemini Enterprise (Google Cloud — Agentspace)  │
│  • Agent UI / chat                              │
│  • Calls MCP via OAuth + Streamable-HTTP        │
└──────────────────┬──────────────────────────────┘
                   │  POST /oauth/token (client_credentials) → JWT
                   │  POST /mcp or /sec/mcp (Bearer JWT) → MCP JSON-RPC
                   ▼
┌─────────────────────────────────────────────────┐
│  ge-mcp-oauth-proxy (Cloudflare Worker)         │
│  • Issues + verifies JWTs (HMAC, KV-revocable)  │
│  • Routes by path → service-binding to backend  │
└────┬─────────────────────────────────┬──────────┘
     │ /mcp                            │ /sec/mcp
     ▼                                 ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│ alphavantage-mcp        │   │ sec-edgar-mcp           │
│ (Cloudflare Worker)     │   │ (Cloudflare Worker)     │
│ Calls AV REST + injects │   │ Calls SEC EDGAR REST    │
│ API key server-side     │   │ with User-Agent header  │
└─────────────────────────┘   └─────────────────────────┘
```

## 2. Gemini Enterprise resources

| Resource | URL / ID |
|---|---|
| **MC GE Agents workspace** (clean financial-demo workspace) | https://vertexaisearch.cloud.google.com/home/cid/6bd325f8-e1e4-4bb3-b5d0-064176f5f140 |
| Cloud Console — engine admin | https://console.cloud.google.com/gen-app-builder/engines/mc-ge-agents/data?project=solutionday-cloudsummit |
| GE Engine | `projects/745425319636/locations/global/collections/default_collection/engines/mc-ge-agents` |
| Alpha Vantage MCP data store | `alpha-vantage-mcp_1777445667644_mcp_data` (FEDERATED — attached to `mc-ge-agents` + `zorro-agentspace`) |
| SEC EDGAR MCP data store | `sec-edgar-mcp_1777449696212_mcp_data` (FEDERATED — same engines) |
| GCP project | `solutionday-cloudsummit` (project number `745425319636`) |

## 3. Local code layout

```
~/Project/Gemini Enterprise Agent/
├── ge-mcp-oauth-proxy/    OAuth proxy (TypeScript Worker)
├── alphavantage-mcp/      Alpha Vantage MCP server (TypeScript Worker)
├── sec-edgar-mcp/         SEC EDGAR MCP server (TypeScript Worker)
└── PROJECT_SUMMARY.md     (this file)
```

Each Worker folder is self-contained: `src/`, `wrangler.{toml|jsonc}`, `package.json`, `NOTES.md`.

## 4. Reverse-engineered API contracts (now part of the build-ge-agent skill)

Several pieces of the Discovery Engine v1alpha API surface aren't documented anywhere public; we discovered them through trial-and-error and saved them as scripts:

| What | Where |
|---|---|
| Create a fresh GE engine | `~/.claude/skills/build-ge-agent/create-engine.sh` |
| Register Custom MCP Server as a data connector (FEDERATED or ACTIONS mode) | `~/.claude/skills/build-ge-agent/register-mcp-tool.sh --mode FEDERATED\|ACTIONS` |
| Standalone OAuth Authorization resource | `~/.claude/skills/build-ge-agent/register-mcp-auth.sh` |
| Full MCP-on-GE recipe | `~/.claude/skills/build-ge-agent/tier2-mcp.md` |

### Key API findings

- **`setUpDataConnector` for MCP** uses `dataSource: "custom_mcp"`, `connectorModes: ["FEDERATED"]` or `["ACTIONS"]`, `refreshInterval: "86400s"`, `params: { instance_uri, auth_uri, auth_uri_params, token_uri, client_id, client_secret, mcp_server_description, scopes }`. Discovered by sending bad payloads and reading Google's error messages.
- **`engine.widgetConfigConfigId` IS the workspace cid** for the GE app URL `vertexaisearch.cloud.google.com/home/cid/<cid>`. This trick lets you open a clean app bound to a specific engine instead of inheriting the user's cluttered default workspace.
- **New engines need feature flags enabled** via PATCH on `engine.features` (`no-code-agent-builder`, `agent-gallery`, `model-selector`, etc., ~17 flags) before the GE UI shows the "+ New agent" button.
- **Agents created via `lowCodeAgentDefinition` API stay in `state: PRIVATE`** — `state` is output-only and there's no public publish/deploy method. The visual builder's Save click is currently the only known way to flip to ENABLED.

## 5. Connector modes — important distinction

| Mode | What the agent gets | Visible in builder |
|---|---|---|
| **FEDERATED** | Treats MCP as a search corpus; GE sends natural-language queries | Knowledge section |
| **ACTIONS** | Calls each MCP tool individually with structured params | Connectors / Tools section, with each tool enumerated |

For "What's Apple's stock price" use cases, **ACTIONS** is required — precise tool invocation. The UI's "Custom MCP Server" form defaults to FEDERATED. ACTIONS only via API, and only after lifting the org policy `discoveryengine.managed.disableCustomMcpServerConnector`.

## 6. Final status

| Layer | Status |
|---|---|
| Cloudflare Worker stack | ✅ Working end-to-end |
| OAuth + JWT + service binding | ✅ Working |
| GE engine + workspace | ✅ Working — clean MC GE Agents workspace bookmarked |
| MCP data stores (FEDERATED) | ✅ Active in `mc-ge-agents` + `zorro-agentspace` |
| Agent UI / chat | ✅ Functional |
| **Tool-calling from GE (original goal)** | ❌ **Blocked** on Google-side ACTIONS init bug — see §7 |
| **Tool-calling from any other MCP client** | ✅ **Works today** — see §6a |

### 6a. The MCP servers work with non-GE MCP clients today

Even though Gemini Enterprise can't yet invoke MCP tools (Google-side init bug), the deployed Cloudflare Workers are **standard remote-MCP HTTP servers**. Any MCP client can connect — [Claude API](https://docs.claude.com/en/docs/build-with-claude/mcp) via the `mcp_servers` parameter, Claude Desktop, [Claude Code](https://docs.claude.com/en/docs/claude-code/mcp), Cursor, Cline, etc.

**Endpoints (live):**

- OAuth token: <https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/oauth/token>
- Alpha Vantage MCP: <https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/mcp>
- SEC EDGAR MCP: <https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/sec/mcp>
- Health: <https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/health>

**End-to-end proof** (verified live during build via curl + MCP JSON-RPC):

```bash
# 1. OAuth handshake
curl -X POST https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=ge-mcp-client&client_secret=<...>"
# → { "access_token": "eyJ...", "expires_in": 3600 }

# 2. tools/list returns 8 tools (4 AV + 4 SEC EDGAR) — full MCP initialize → tools/list
# 3. tools/call get_stock_quote(symbol="IBM") returns live Alpha Vantage data
```

**Claude API integration** (the same pattern works for Claude Desktop / Claude Code / Cursor / Cline):

```python
client.beta.messages.create(
    model="claude-sonnet-4-6",
    mcp_servers=[{
        "type": "url",
        "url": "https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/mcp",
        "name": "alpha-vantage",
        "authorization_token": "<JWT from /oauth/token>",
    }],
    betas=["mcp-client-2025-04-04"],
    messages=[{"role": "user", "content": "What's Apple's current stock price?"}],
)
```

A ready-to-run reference script lives at `test_mcp_with_claude.py` in the repo — set `ANTHROPIC_API_KEY` and run it to verify the chain end-to-end with Claude.

**Practical takeaway:** the Cloudflare Worker stack is **not waiting on Google** — it's a usable MCP backend right now for any non-GE client. The GE side is one cleanup command away whenever Google's init pipeline gets fixed.

## 7. Outstanding blocker — Google-side init pipeline bug

After the org admin lifted `discoveryengine.managed.disableCustomMcpServerConnector` and verified no sibling constraints are active, ACTIONS-mode connector creation now succeeds at the API surface but **fails at init pipeline** with a generic `INITIALIZATION_FAILED, code 13: pipeline failure` error.

**Confirmed via Cloudflare Worker logs:** Google never makes any HTTP request to our OAuth proxy or MCP server during init — the failure is entirely inside Google's GE backend before any external probe. Both `:setUpDataConnector` (v1) and `:setUpDataConnectorV2` exhibit identical behaviour.

This is conclusively a GE-backend bug or hidden gating, not anything in our control. **Next step: org admin opens a Google Cloud Support ticket** with the diagnostic findings.

## 8. Once Google fixes their side, recovery is one command

```bash
# Re-register both MCPs in ACTIONS mode
bash ~/.claude/skills/build-ge-agent/register-mcp-tool.sh \
  --project solutionday-cloudsummit --mode ACTIONS \
  --collection-id alphavantage-mcp \
  --display-name "Alpha Vantage MCP" \
  --mcp-url "https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/mcp" \
  --auth-url "https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/oauth/authorize" \
  --token-url "https://ge-mcp-oauth-proxy.masterconcept-hongkong.workers.dev/oauth/token" \
  --client-id ge-mcp-client \
  --client-secret <secret> \
  --description "Alpha Vantage financial-data MCP. Tools: get_stock_quote, get_company_overview, search_symbols, get_daily_time_series."
# Same for SEC EDGAR with --mcp-url .../sec/mcp
```

Then `dynamicTools` populates with the 4 tools per server, the agent picks them up, and the demo works.

---

*Generated 2026-04-29.*
