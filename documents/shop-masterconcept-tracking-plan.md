---
title: "`shop.masterconcept.ai` 轉換追蹤規劃"
date: 2026-05-15
---

# `shop.masterconcept.ai` 轉換追蹤規劃

**目標：** 在 Master Concept 網路商店追蹤按鈕點擊與關鍵用戶行為，將數據送到 Google Ads（及未來其他廣告平台），用於轉換優化與再行銷。

---

## 🟢 專案目前進度（Last updated: 2026-05-15）

| 項目 | 狀態 | ID / 備註 |
|---|---|---|
| GTM container（Web） | ✅ 已建立 | `GTM-MBBXJM2F` |
| GA4 Property — Production | ✅ 已建立 | `G-S42GRZGW62` |
| GA4 Property — Development | ✅ 已建立 | `G-HYZDFDW7EW` |
| GTM snippet 安裝至 Next.js | ⏳ 待工程師埋入 | 見 [Phase 1](#phase-1--基礎建置半天) |
| dataLayer events 埋點 | ⏳ 待開發 | 見 [Phase 2](#phase-2--event-埋點工程師-12-天) |
| Google Ads conversion tag | ⏳ 未開始 | 見 [Phase 3](#phase-3--接通-google-ads約-30-分鐘) |
| Server-side conversion（付款 webhook） | ⏳ 未開始 | 見 [Section 7](#7-server-side-conversion-tracking付款流程離站) |

**Dev / Prod 環境切換建議：**
在 GTM 內用一個 **Lookup Table variable**，輸入 = `{{Page Hostname}}`：
- `shop.masterconcept.ai` → `G-S42GRZGW62`（Prod）
- 其他（如 `staging.*`、`localhost`） → `G-HYZDFDW7EW`（Dev）

GA4 Configuration tag 的 Measurement ID 欄位填這個變數，這樣同一份 GTM container 就能同時服務 Prod 與 Dev，不會把測試數據污染到正式報表。

---

## 1. 現況稽核 — 目前裝了什麼？

我抓取 `https://shop.masterconcept.ai/` 的真實 HTML（會 307 跳轉到 `/zh-TW/google-workspace`），並 grep 整份原始碼掃描追蹤碼特徵。

| Tag / 工具 | 狀態 |
|---|---|
| Google Tag Manager (`GTM-XXXX`) | ❌ 未安裝 |
| Google Analytics 4 (`G-XXXXXXX`) | ❌ 未安裝 |
| Universal Analytics (`UA-XXXXXXX`) | ❌ 未安裝 |
| Google Ads tag (`AW-XXXX`) | ❌ 未安裝 |
| Meta / Facebook Pixel | ❌ 未安裝 |
| Hotjar / Microsoft Clarity | ❌ 未安裝 |
| Freshchat 客服 widget | ✅ 已安裝（網站上唯一的第三方腳本） |

**結論：** 網站目前完全沒有任何分析、轉換或再行銷追蹤碼 — 是張白紙，從零開始建置不會與舊 tag 衝突。

### 技術棧重點（影響安裝方式）

- 網站是 **Next.js App Router**，資源走 `/_next/static/...` chunk。
- 啟用了**嚴格 CSP 並帶 per-request `nonce`**（每個 `<script>` 都有 `nonce="N2IyMmRkZjQt..."`）。任何追蹤碼都必須帶相同 nonce，否則會被瀏覽器擋掉。
- 網站為多語系（`/zh-TW/`，預期還有 `/en/` 等）— 要確認 tag 在所有語系都會觸發。

### 原始碼中找到的轉換點 CTA

直接從 HTML 撈出來：

| 按鈕文字 (zh-TW) | 含義 | 建議 GA4 event |
|---|---|---|
| `立即購買` | Buy now | `begin_checkout` |
| `加入購買` | Add to cart | `add_to_cart` |
| `立即綁定` / `立即註冊並綁定` | 註冊綁定帳號 | `sign_up` |
| `購買方案` / `購買產品` | 查看產品/方案 | `view_item` |
| `聯絡我們` | Contact us | `generate_lead` |
| 訂閱完成頁 | 購買完成 | `purchase`（帶 `value` + `currency`） |

---

## 2. 建議架構 — GTM ➜ GA4 ➜ Google Ads

```
[Next.js shop]  ── push events ──►  window.dataLayer
                                          │
                                          ▼
                                     [GTM container]
                                          │
                ┌─────────────────────────┼─────────────────────────┐
                ▼                         ▼                         ▼
            [GA4 property]        [Google Ads conversion]      [Meta Pixel / 其他]
            （數據衡量）         （競價訊號 + 再行銷）            （未來擴充）
```

### 為什麼用 GTM（而不是直接貼 `gtag.js`）

1. **一次安裝，多 tag 共用。** 之後要加 Facebook Pixel、LinkedIn Insight、TikTok Pixel，零代碼改動，GTM 內新增 tag 即可。
2. **非工程師也能調整。** 行銷團隊可在 GTM UI 直接改觸發條件，不用走 Next.js 部署。
3. **內建 Preview / Debug 模式。** 可在正式環境測試，不污染真實數據。
4. **單一數據源。** 所有事件走同一個 `dataLayer`，GA4、Google Ads、Meta 看到的資料一致。

### 為什麼 GA4 **與** Google Ads conversion 都要裝

| 層級 | 用途 |
|---|---|
| **GA4** | 全漏斗衡量 — sessions、來源、用戶旅程、留存。報表與分析的基礎。 |
| **Google Ads conversion tag** | 專門餵 Google Ads 競價系統。比對 `gclid`（廣告點擊識別），支援 **Enhanced Conversions**（hash email/phone）— 在 iOS / cookie 限制下仍能維持高歸因率。 |

**簡單做法：** 把 GA4 事件標記為 conversion，再 *import* 到 Google Ads。
**最佳做法：** 同時貼一個獨立的 Google Ads conversion tag (`AW-XXXX/abc...`) 給 `purchase` 事件用，並開啟 Enhanced Conversions。匹配率更高、回報更即時。

---

## 3. 實作計劃

### Phase 1 — 基礎建置（半天）

1. ✅ **已完成** — GA4 property 已建立：
   - Production: `G-S42GRZGW62`
   - Development: `G-HYZDFDW7EW`
2. ✅ **已完成** — GTM container 已建立：`GTM-MBBXJM2F`
3. ⏳ **待工程師執行** — 把 GTM snippet 加到 Next.js root layout。因為有 CSP nonce，必須用 `next/script` 並顯式傳入 nonce：

   {% raw %}
   ```tsx
   // app/[lng]/layout.tsx (head)
   <Script
     id="gtm-init"
     strategy="afterInteractive"
     nonce={nonce}
     dangerouslySetInnerHTML={{
       __html: `(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
         new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
         j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
         'https://www.googletagmanager.com/gtm.js?id='+i+dl;j.nonce='${nonce}';
         f.parentNode.insertBefore(j,f);})(window,document,'script','dataLayer','GTM-MBBXJM2F');`
     }}
   />
   ```
   {% endraw %}
   同時在 `<body>` 開頭加上 `<noscript>` iframe，給關閉 JS 的用戶使用：

   ```html
   <noscript>
     <iframe src="https://www.googletagmanager.com/ns.html?id=GTM-MBBXJM2F"
             height="0" width="0" style="display:none;visibility:hidden"></iframe>
   </noscript>
   ```

4. ⏳ **待 GTM 內設定** — 在 GTM 新增以下兩個元件：

   **a. Lookup Table Variable** (名稱建議：`GA4 - Measurement ID`)
   - Input Variable: `{{Page Hostname}}`
   - Default Value: `G-HYZDFDW7EW` (Dev)
   - Rows:
     | Input | Output |
     |---|---|
     | `shop.masterconcept.ai` | `G-S42GRZGW62` |

   **b. GA4 Configuration tag**
   - Measurement ID: `{{GA4 - Measurement ID}}`
   - Trigger: All Pages

**完成條件：** GA4 Realtime 報表（Prod property）能看到正式環境的 pageview，且 Dev property 看到測試環境的流量。

### Phase 2 — Event 埋點（工程師 1–2 天）

工程師在每個關鍵動作 push `dataLayer` event。建議遵循 GA4 e-commerce 標準 schema — GA4、Google Ads、Meta 都看得懂同一份 payload。

**Add-to-cart 範例：**
```js
window.dataLayer.push({ ecommerce: null });   // 清空前一筆 payload
window.dataLayer.push({
  event: 'add_to_cart',
  ecommerce: {
    currency: 'TWD',
    value: 2400,
    items: [{
      item_id: 'gws-business-starter',
      item_name: 'Google Workspace Business Starter',
      item_category: 'Google Workspace',
      price: 480,
      quantity: 5,
    }],
  },
});
```

**Purchase 範例（訂單完成頁觸發）：**
```js
window.dataLayer.push({ ecommerce: null });
window.dataLayer.push({
  event: 'purchase',
  ecommerce: {
    transaction_id: 'MC-2026-00123',
    currency: 'TWD',
    value: 12000,
    items: [/* 同上格式 */],
  },
  // 給 Enhanced Conversions 用 — hashing 在 GTM 內完成：
  user_data: {
    email: 'buyer@example.com',
    phone_number: '+886912345678',
  },
});
```

在 GTM 內，每個 event name 對應一個 **GA4 Event tag**，trigger = "Custom Event equals `add_to_cart`" / `purchase` / 等等。

### Phase 3 — 接通 Google Ads（約 30 分鐘）

1. Google Ads → **Tools → Conversions**，建立 conversion action（例如 *"Workspace Subscription Purchase"*），拿到類似 `AW-1234567890/AbC-DeFgHi` 的 tag。
2. GTM 內新增 **Google Ads Conversion Tracking** tag，填入 ID + label。Trigger = `purchase` event。從 dataLayer 取 `value` 與 `transaction_id` 傳入。
3. 在同一個 Google Ads conversion action 內啟用 **Enhanced Conversions**。GTM 內把 hashed `user_data.email` / `phone_number` 對應上去。
4. 新增 **Google Ads Remarketing** tag，trigger = All Pages，用於累積再行銷 audience。
5. （建議）在 GA4 Admin → Product Links 把 Google Ads 與 GA4 串接，這樣 GA4 audiences/conversions 可直接 import 到 Ads。

### Phase 4 — 驗證 Checklist

- [ ] **GTM Preview mode**：逐一點擊所有 CTA，確認每個 event 都觸發、payload 正確。
- [ ] **GA4 DebugView**：每個事件秒級進來，參數齊全。
- [ ] **GA4 Realtime → Conversions**：把 `purchase`、`add_to_cart`、`generate_lead`、`sign_up` 標為 conversion。
- [ ] **Google Ads → Conversions**：狀態顯示 *"Recording conversions"*（首筆真實轉換後最多 24 小時）。
- [ ] **Tag Assistant** (`tagassistant.google.com`)：通過所有檢查，無重複觸發。
- [ ] **Consent**：確認 cookie consent banner / GDPR 狀況（若預期有歐盟流量）— 考慮導入 Google Consent Mode v2。

---

## 4. Event Map — 快速對照表

| 用戶動作 | dataLayer `event` | GA4 conversion? | Google Ads conversion? | 備註 |
|---|---|---|---|---|
| 頁面瀏覽 | `page_view`（自動） | No | No | 用於 audience 建構 |
| 查看產品 / 方案 | `view_item` | No | No | 漏斗訊號 |
| 點 "加入購買" | `add_to_cart` | Yes（micro） | No | 對 Smart Bidding 有幫助 |
| Begin checkout | `begin_checkout` | Yes（micro） | Optional | |
| 點 "聯絡我們" 表單送出 | `generate_lead` | Yes | Yes（lead conv.） | B2B lead 通常高價值 |
| 帳號註冊 / 綁定 | `sign_up` | Yes | Optional | |
| **訂閱確認完成** | **`purchase`** | **Yes（primary）** | **Yes（primary）** | 必帶 `value` + `transaction_id` |

---

## 5. 工作量與分工

| Phase | Owner | 時程 |
|---|---|---|
| 1. GA4 + GTM 開戶、安裝 snippet | Dev + Marketing | 0.5 天 |
| 2. 每個 CTA 加 dataLayer event | Frontend dev | 1–2 天 |
| 3. Google Ads conversion + remarketing | Marketing（GTM UI 內） | 0.5 天 |
| 4. QA 驗證 | Marketing + Dev | 0.5 天 |
| **總計** | | **約 3 天 elapsed** |

---

## 6. 開工前要確認的事

1. GA4 與 Google Ads 帳號由誰管理？（Master Concept 行銷團隊？）
2. 網站是否已有 **cookie consent / GDPR 政策**？若有，第一天就要規劃 **Consent Mode v2**。
3. 要不要趁這次一起裝 **Meta Pixel** 與 **LinkedIn Insight Tag**？放進同一個 GTM container，邊際成本極低，未來廣告渠道直接打開。
4. 訂單完成頁的確切 URL pattern 是？需要這個來建立可靠的 `purchase` trigger。
