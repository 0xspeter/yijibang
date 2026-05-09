# 架構決策紀錄（ADR）

本資料夾紀錄一級棒鎖印 & 瓊瑤打字行網站（yijibang.netlify.app）建置過程中的關鍵架構決策。

每一篇 ADR 描述一個決策的「**為什麼**」，而不只是「做了什麼」——讓未來的自己或交接同事能理解選擇背後的 trade-off。

## ADR 索引

| 編號 | 主題 | 狀態 | 日期 |
|---|---|---|---|
| [0001](./0001-multi-page-seo-architecture.md) | 從單頁 linktree 重構為多頁面 SEO 站 | Accepted | 2026-05-05 |
| [0002](./0002-tailwind-cdn.md) | 採用 Tailwind CDN 而非 build 流程 | Accepted | 2026-05-05 |
| [0003](./0003-netlify-github-deployment.md) | 部署：Netlify + GitHub auto-deploy | Accepted | 2026-05-05 |
| [0004](./0004-url-structure.md) | URL 結構與目錄組織 | Accepted | 2026-05-05 |
| [0005](./0005-seo-structured-data.md) | SEO 結構化資料策略 | Accepted | 2026-05-05 |
| [0006](./0006-tracking-strategy.md) | 追蹤策略：GA4 + Google Ads + 自動 tel: 偵測 | Accepted | 2026-05-09 |
| [0007](./0007-design-system.md) | 共用設計系統 | Accepted | 2026-05-05 |

## ADR 格式

每篇 ADR 包含：

- **Status**：Proposed / Accepted / Deprecated / Superseded by ADR-xxxx
- **Context**：問題背景、為什麼需要這個決策
- **Decision**：決定怎麼做
- **Consequences**：好處、壞處、要承擔的成本
- **Alternatives**：曾考慮過但沒採用的方案，附理由

## 更新規則

- 不修改既有 ADR 內容（除了打錯字）。決策若改變，建立新 ADR 並標註舊的為 `Superseded`。
- 新增 ADR 從下一個編號開始（目前已用到 0007）。
- 標題用「動詞 + 名詞」格式（例如「採用 Tailwind CDN」而非「Tailwind CDN」）。
