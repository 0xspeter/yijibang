# ADR-0007：共用設計系統

- **狀態**：Accepted
- **日期**：2026-05-05

## Context

12 個頁面需保持視覺一致性。沒有 build 流程（[ADR-0002](./0002-tailwind-cdn.md)）也沒有 component framework，每頁都是獨立 HTML 檔。需要建立一套設計 token 與重複使用的 layout pattern，避免「這頁的 button 圓角是 8px、那頁是 12px」的不一致。

## Decision

### 設計 Token（每個頁面都複製這套 CSS variables）

```css
:root {
  --primary:    #1A365D;   /* navy 深藍：標題、按鈕主色、footer 背景 */
  --secondary:  #B8860B;   /* gold 金色：強調、品牌色、CTA hover */
  --accent:     #E53E3E;   /* red 紅色：緊急按鈕（電話、緊急救援）*/
  --background: #F7FAFC;   /* off-white 米白：section 交替底色 */
  --foreground: #2D3748;   /* near-black：內文 */
  --muted:      #718096;   /* gray：次要文字、說明 */
}
```

### 字型

```html
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700&family=Noto+Serif+TC:wght@700;900&display=swap" rel="stylesheet">
```

- **Noto Sans TC**：內文（300/400/500/700）
- **Noto Serif TC**：標題 h1-h4（700/900）

### 共用 Layout Pattern

每個頁面（除首頁外）有一致結構：

```
[Fixed Header with Nav]
[Hero Section (hero-gradient)]
  - 麵包屑導航
  - 徽章 / Tag
  - h1 標題
  - 描述文字
  - 雙 CTA 按鈕（電話 + LINE）
[Why us / About Section]
  - 5 段落式內容
[Service Areas / Tables Section]
  - 3 column grid
[FAQ Accordion Section]
  - 5 個問題
[Related Services Section]
  - 4 個 link cards
[Footer]
  - 雙店資訊
  - 連結列表
  - 版權
```

### 共用組件 CSS

```css
/* Hero gradient（深藍漸層） */
.hero-gradient {
  background: linear-gradient(135deg, var(--primary) 0%, #2A4365 100%);
}

/* Accordion（FAQ 展開） */
.accordion-content { max-height: 0; overflow: hidden; transition: max-height 0.3s ease-out; }
.accordion-item.active .accordion-content { max-height: 600px; }
.accordion-item.active .accordion-icon { transform: rotate(180deg); }

/* Hover lift（卡片懸浮效果） */
.hover-lift { transition: transform 0.25s ease, box-shadow 0.25s ease; }
.hover-lift:hover { transform: translateY(-4px); box-shadow: 0 12px 28px rgba(0,0,0,0.08); }
```

### Nav 統一（6 項）

每個頁面的 desktop nav 一致：

```
關於我們 | 服務項目 | 服務區域 | 常見問題 | 部落格 | [立即聯繫]
```

- 在首頁：anchor links 指向 `#about`、`#services`、`#areas`、`#faq`、`#contact`
- 在子頁：anchor links 指向 `/#about`、`/#services` 等（回首頁的 anchor）
- `部落格` 連結指向 `/blog/`（無論在哪個頁面）

### Footer 統一

- 雙店資訊（建國 + 中正）
- LINE 連結（`@hig7952z`）
- 版權聲明

## Consequences

### 好處

- **快速複製新頁面**：拿任一既有頁面當樣板，改 meta + body 就行
- **一致的使用者體驗**：使用者在頁面間切換不會有「跑到另一個網站」的錯覺
- **品牌一致**：navy + gold 的「精品老店」感貫穿全站

### 壞處／成本

- **CSS 重複**：每個頁面都有同一份 `<style>` 區塊，更新時要同步 12 處
- **沒有真正的 component**：button 樣式 hardcode 在 HTML 內，不像 React/Vue 可以 extract 成 `<Button />`

### 緩解策略

- 風格更新時用 grep + 批次 Edit 工具同步（已在 [ADR-0006](./0006-tracking-strategy.md) 改 GA4 時驗證可行）
- 若未來想 extract component，可改用 Astro / Eleventy（純靜態 SSG），保留 HTML output 不變

## Alternatives Considered

### 用 Web Components

```html
<my-header></my-header>
<my-footer></my-footer>
```

**否決原因**：每個 component 要寫 JS class，定義 shadow DOM；對純 HTML 編輯而言反而複雜化。

### Server-side include（SSI）

```html
<!--#include virtual="/_includes/header.html" -->
```

**否決原因**：Netlify 預設不啟用 SSI；要設定 `_redirects` 或 build 流程。

### iframe 共用 header / footer

**否決原因**：iframe 對 SEO 不友好（搜尋引擎不會合併內容）；無法 styling 跨域；2026 年沒人這樣做了。

### 改用 SSG（Astro / Eleventy / Hugo）

**否決原因**：詳見 [ADR-0001](./0001-multi-page-seo-architecture.md) 與 [ADR-0002](./0002-tailwind-cdn.md)。簡單來說：本案無 build 流程是刻意選擇，便於未來非工程師維護。

## References

- 設計 token 參考：[index.html](../../index.html) 的 `<style>` 區塊
- Tailwind 部分：[ADR-0002](./0002-tailwind-cdn.md)
