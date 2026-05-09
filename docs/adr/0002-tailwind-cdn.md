# ADR-0002：採用 Tailwind CDN 而非 build 流程

- **狀態**：Accepted
- **日期**：2026-05-05

## Context

網站需要快速建立 12 個頁面，每頁都有複雜的 layout（hero gradient、卡片、accordion、grid 等）。手寫 CSS 維護成本高、難以保持一致性。

## Decision

每個 HTML 頁面都用 **CDN 載入 Tailwind CSS**：

```html
<script src="https://cdn.tailwindcss.com"></script>
```

加上少量自訂 CSS 變數（color tokens）與少量自訂 class（accordion、hover-lift）：

```css
:root {
  --primary: #1A365D;
  --secondary: #B8860B;
  --accent: #E53E3E;
  ...
}
```

不導入 Node.js、PostCSS、build pipeline。

## Consequences

### 好處

- **零 build 環境**：純 HTML 編輯，雙擊即可在瀏覽器預覽
- **入門門檻低**：未來交接給非工程師也能改文案
- **快速 iterate**：改一個 class 就能立即看效果，無等待 build 時間
- 與 Netlify 部署相容（無需設 build command，靜態檔直接服務）

### 壞處／成本

- **載入體積大**：CDN 版 Tailwind ~3 MB（壓縮後 ~300 KB），相比 build 後的 purged CSS（通常 < 20 KB）大很多
- **首次載入慢**：相依 CDN 速度，台灣使用者實測 ~200-500ms 額外延遲
- **production 警告**：Tailwind 官方 CDN 在 console 會顯示「Don't use this in production」warning（功能正常，僅是友善提醒）
- **CDN 服務終止風險**：若 cdn.tailwindcss.com 哪天下線，全站樣式會壞

### 緩解策略

- 若未來流量大、效能變得重要：可改成 build CSS（會建立 ADR-00xx 取代本決策）
- 把 CDN 連結改用具體版本號（例如 `https://cdn.tailwindcss.com/3.4.0`），降低破壞性更新風險

## Alternatives Considered

### Tailwind CLI build（產生 purged CSS）

**否決原因**：需要 Node.js 環境、需要 watch mode、編輯文案後要 rebuild → 違反「老店未來自行維護」目標。

### Bootstrap / Bulma / Pico.css

**否決原因**：utility-first 寫法在 HTML 內快速搭建 layout 比 component-class 更快；Tailwind 的設計語言（rounded、shadow、gradient）與我們要的「精品老店」感覺契合。

### 純手寫 CSS

**否決原因**：12 個頁面 + RWD + 一致性，手寫 CSS 維護成本太高，且容易產生「這頁的 padding 跟另一頁差 2px」的不一致。

## References

- 共用樣式定義在每個 HTML 的 `<style>` 區塊（color tokens、accordion、hover-lift）
- [ADR-0007](./0007-design-system.md) 設計系統
