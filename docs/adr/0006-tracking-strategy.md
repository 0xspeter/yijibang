# ADR-0006：追蹤策略：GA4 + Google Ads + 自動 tel: 偵測

- **狀態**：Accepted
- **日期**：2026-05-09

## Context

業主未來會跑 Google Ads 廣告，需要知道「廣告花了多少錢、買到多少通電話諮詢」。鎖匠服務 80% 以上的成交透過電話發生（緊急開鎖時間敏感，使用者直接撥打不會走表單），所以「**點擊電話按鈕**」是最重要的轉換指標。

需要決定：

1. 流量分析工具
2. Google Ads 轉換代碼整合
3. 電話點擊事件如何追蹤

## Decision

### 雙標籤並存

每個頁面都載入兩個 gtag config：

```javascript
gtag('config', 'G-BDXBNDRJTR');     // GA4：流量分析、使用者行為
gtag('config', 'AW-980254441');     // Google Ads：轉換追蹤
```

兩者共用一份 `gtag.js` library，不會重複載入。

### 自動偵測式 tel: 連結追蹤

在每個頁面的 gtag script block 內加上：

```javascript
document.addEventListener('DOMContentLoaded', function() {
  document.querySelectorAll('a[href^="tel:"]').forEach(function(link) {
    link.addEventListener('click', function() {
      gtag('event', 'conversion', {
        'send_to': 'AW-980254441/-wdwCI_y_akcEOn9tdMD'
      });
    });
  });
});
```

任何 `<a href="tel:...">` 連結被點擊時，自動向 Google Ads 回報一筆「點擊電話號碼」轉換。

同時保留 Google 官方推薦的手動觸發函數（給未來特殊按鈕用）：

```javascript
function gtag_report_conversion(url) { ... }
```

## Consequences

### 好處

- **零 onboarding 成本**：未來在任何頁面新增 `<a href="tel:...">`，自動納入追蹤，不需記得改設定
- **DRY**：12 個頁面 + 多個電話按鈕（首頁 6+ 個、子頁 2-3 個），不必各別寫 `onclick`
- **HTML 乾淨**：`<a href="tel:...">` 維持純 HTML 語意，沒有混入分析代碼
- **GA4 + Google Ads 並存**：同時拿到流量分析（GA4：哪個頁面熱門、跳出率）與廣告 ROI（Google Ads：CPA、ROAS）

### 壞處／成本

- 每頁多 ~600 bytes JS（極小）
- 依賴 `DOMContentLoaded` 事件 → 若使用者在 DOMContentLoaded 觸發前就點擊（極罕見），第一次點擊不會追蹤到（但通常會 retry）
- 若日後改用 SPA 框架（[ADR-0001](./0001-multi-page-seo-architecture.md) 已否決），此自動偵測在 client-side route 切換後不會自動 re-bind

### 緩解策略

- 改 SPA 時要改用 MutationObserver 或框架的 router event hook
- 目前是純 HTML，不會發生這問題

### 隱私／合規

- GA4 + Google Ads 均使用 cookie 與 client identifier，技術上落入 GDPR 範圍
- 目標客群在台灣，受《個資法》規範但無 GDPR 強制；目前未加 cookie consent banner
- 若未來服務歐洲使用者，要加 consent banner（建議用 Cookiebot 或 Klaro）

## Alternatives Considered

### 手動在每個 `<a href="tel:...">` 加 `onclick`

```html
<a href="tel:038569085" onclick="return gtag_report_conversion('tel:038569085');">☎</a>
```

**否決原因**：違反 DRY，未來新增頁面要記得加；維護心智成本高。

### 用 Google Tag Manager

**否決原因**：

- 多一層工具與帳號要維護
- 對小型靜態網站殺雞用牛刀
- 直接 gtag 已可完成所有需求

### 只裝 GA4，不裝 Google Ads conversion

**否決原因**：未來跑廣告時，廣告花費與成效對應不上來，無法做 CPA / ROAS 計算。

### 只裝 Google Ads，不裝 GA4

**否決原因**：缺流量分析（哪些頁面熱、跳出率、來源），無法優化內容。

## 已驗證

實機測試（Chrome DevTools Network 面板）：

- 點擊 `tel:038569085` → 發出 `googleads.g.doubleclick.net/pagead/980254441/?random=...` 請求
- 重導鏈 `302 → 302 → 200` 正常
- 包含 conversion label `-wdwCI_y_akcEOn9tdMD`

## References

- GA4 Measurement ID：`G-BDXBNDRJTR`
- Google Ads Customer ID：`AW-980254441`
- Conversion label：`-wdwCI_y_akcEOn9tdMD`
- 觸發事件：`gtag('event', 'conversion', { 'send_to': 'AW-980254441/-wdwCI_y_akcEOn9tdMD' })`
