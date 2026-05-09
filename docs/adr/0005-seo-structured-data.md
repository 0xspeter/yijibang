# ADR-0005：SEO 結構化資料策略

- **狀態**：Accepted
- **日期**：2026-05-05

## Context

純靠 `<title>` 與 `<meta description>` 已不足以讓搜尋引擎與 AI 服務（Google AI Overviews、ChatGPT、Perplexity 等）正確理解內容。需要結構化資料：

- 告訴搜尋引擎「這是一家鎖匠店、地址、營業時間、電話、服務項目」
- 提升 SERP 上的 Rich Snippets 機率（FAQ 直接顯示在搜尋結果）
- 讓 AI 回答「花蓮哪裡可以開鎖」類問題時，把這家店列為來源

## Decision

採用 **Schema.org JSON-LD** 結構化資料，每個頁面類型加對應 schema：

### 全站共用（首頁）

```json
[
  {"@type": "Organization", "@id": "...#organization", ...},
  {"@type": "Locksmith", "@id": "...#jianguo-store", ...},
  {"@type": "Locksmith", "@id": "...#zhongzheng-store", ...},
  {"@type": "WebSite", ...},
  {"@type": "FAQPage", ...}
]
```

### 區域頁

- `BreadcrumbList`：首頁 → 服務區域 → 該區
- `Service`：areaServed 列出該區所有村／里
- `FAQPage`：5 個在地常見問題

### 服務頁

- `BreadcrumbList`：首頁 → 服務項目 → 該服務
- `Service`：含 `AggregateOffer` 價位區間（電子鎖頁有）
- `FAQPage`：5 個服務相關問題

### 部落格

- `Blog`（首頁）含 `blogPost` 列表
- `Article`（文章頁）含 author、publisher、datePublished、dateModified

### 輔助檔案

- **sitemap.xml**：列出全部 12 個 URL，含 lastmod / changefreq / priority
- **robots.txt**：允許全部爬取，指向 sitemap
- **llms.txt**：給 AI 模型（ChatGPT、Claude 等）的 plain-text 摘要，包含商家資訊、服務項目、引用建議

### Geo / Open Graph / Twitter

每頁都有：

- `<meta name="geo.region">`、`geo.placename`、`geo.position`、`ICBM`
- Open Graph：`og:title`、`og:description`、`og:type`、`og:url`、`og:image`、`og:locale`
- Twitter Card：`twitter:card`、`twitter:title`、`twitter:description`
- Canonical URL：`<link rel="canonical">` 統一為 `https://yijibang.netlify.app/...`

## Consequences

### 好處

- **Rich Snippets 機會大**：FAQPage schema 讓搜尋結果直接顯示問答，提升 CTR
- **Local Business 整合**：`Locksmith` schema 含地址、電話、營業時間，幫助 Google Maps / Bing Local 整合
- **AI 友善**：`llms.txt` 是 emerging standard，主動提供 AI 引用所需的結構化文字
- **跨頁面實體一致性**：用 `@id` 連接（例如所有頁面引用同一個 `#organization`），Google 知道「這些都是同一家店」

### 壞處／成本

- 每頁 HTML 開頭多 1-3 KB 的 JSON-LD（不影響使用者體驗，搜尋引擎也不在意）
- Schema 寫錯會被 Google Search Console 標 warning（已用 Google Rich Results Test 驗證主要 schema）
- 商家資訊變動時要同步更新所有頁面的 schema（維護成本）

### 緩解策略

- 重要資訊（電話、地址、營業時間）只在「首頁 + footer」維護，子頁不重複放完整營業時間
- llms.txt 集中放所有結構化資訊，是「single source of truth」的試驗

## Alternatives Considered

### Microdata（itemtype / itemprop）

```html
<div itemscope itemtype="http://schema.org/Locksmith">
  <span itemprop="name">建國總店</span>
  ...
</div>
```

**否決原因**：JSON-LD 是 Google 推薦格式，與 HTML 分離維護更乾淨；Microdata 會把 schema 屬性混進 layout HTML，難維護。

### RDFa

**否決原因**：同上，且 RDFa 在繁體中文 SEO 工具中支援度比 JSON-LD 差。

### 不放結構化資料

**否決原因**：對純依賴 keyword 排名的網站可能還行，但對 local business（要被 Google Maps、Bing Local 找到）是必備項目。

## References

- [Schema.org Locksmith](https://schema.org/Locksmith)
- [Google FAQ Page guidelines](https://developers.google.com/search/docs/appearance/structured-data/faqpage)
- [llms.txt proposal](https://llmstxt.org/)
- [llms.txt](../../llms.txt)
- [sitemap.xml](../../sitemap.xml)
