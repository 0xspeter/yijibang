# ADR-0001：從單頁 linktree 重構為多頁面 SEO 站

- **狀態**：Accepted
- **日期**：2026-05-05

## Context

舊版（page5888 帳號被停權前）的 yijibang.netlify.app 是一個 **單頁面 linktree 風格** 的手機優先網站：

- 所有內容塞在 `index.html` 一個檔案
- 沒有 `/services/`、`/areas/` 子頁
- 服務描述、FAQ、聯絡資訊全部疊在同一頁
- canonical URL 只有一個（首頁）

問題：

1. **SEO 競爭力差**：當使用者搜尋「花蓮電子鎖安裝」、「吉安開鎖」、「肚臍章」這類具體 keyword 時，搜尋引擎只能比對到「整個首頁」，而不是某個專屬頁面。整個首頁因為包山包海，相關性分數被稀釋。
2. **無法針對地點做 local SEO**：吉安、花蓮市、壽豐、志學（東華）都是各自獨立的搜尋意圖，但單頁無法為每個區域寫深度內容。
3. **內容組織困難**：使用者想深入了解「電子鎖怎麼挑」、「滾碼遙控故障」這類話題時，沒有專屬空間放詳細資訊。

## Decision

重構為 **多頁面結構**，建立 12 個頁面：

```
/                                                  (首頁)
/areas/jian/                                      (吉安)
/areas/hualien-city/                              (花蓮市)
/areas/shoufeng/                                  (壽豐)
/areas/zhixue/                                    (志學/東華)
/services/electronic-lock/                        (電子鎖安裝)
/services/chip-key/                               (汽機車晶片鑰匙)
/services/rolling-code/                          (滾碼鐵捲門遙控)
/services/access-card/                           (社區門禁卡複製)
/services/stamps/                                (刻印章)
/blog/                                            (部落格首頁)
/blog/hualien-electronic-lock-guide/             (電子鎖選購指南文章)
```

每個頁面為一個獨立 keyword target：

- 區域頁針對「{地區} + 開鎖／鎖匠／換鎖」搜尋意圖
- 服務頁針對「花蓮 + {服務名}」搜尋意圖
- 部落格文章針對 long-tail informational 搜尋意圖

每頁有自己的 `<title>`、`<meta description>`、canonical URL、結構化資料。

## Consequences

### 好處

- 每個關鍵字有專屬入口頁，搜尋引擎相關性分數提升
- Local SEO：每個區域頁可針對該區獨特需求寫內容（吉安透天厝、花蓮市老公寓、壽豐海邊民宿、志學學生套房）
- 部落格可累積 long-tail 流量，非緊急訪客也能進站
- 內部連結網路（首頁 ↔ 區域頁 ↔ 服務頁）強化 SEO 主題權威

### 壞處／成本

- 維護成本提高：12 個頁面要維護，nav / footer / GA 改動要同步 12 處
- 首次撰寫成本：每個頁面要寫專屬 body 與 FAQ，不能複製貼上
- HTML 重複：header、footer、style、JSON-LD 在每頁都要寫一份（沒有 build 流程做共用）

### 緩解策略

- 用一致的 nav / footer / style 樣板（見 [ADR-0007](./0007-design-system.md)）
- 自動偵測式追蹤（見 [ADR-0006](./0006-tracking-strategy.md)），減少新增頁面時的 onboarding 成本
- sitemap.xml 集中管理 URL 清單

## Alternatives Considered

### 維持單頁，加 anchor links

```
/#electronic-lock
/#chip-key
/#about
```

**否決原因**：anchor 不是獨立 URL，搜尋引擎仍視為同一頁面；無法為各 anchor 寫獨立 meta description；分享 LINE/FB 預覽圖只能是同一張。

### 用 SPA 框架（React / Next.js / Vue）

**否決原因**：客戶網站只有靜態內容，沒有互動需求；引入框架增加複雜度與部署成本；50 年老店未來主要由非工程師維護，純 HTML 比 SPA 友善許多。

### 用 GitHub Pages 而非 Netlify

**否決原因**：見 [ADR-0003](./0003-netlify-github-deployment.md)。

## References

- [sitemap.xml](../../sitemap.xml) — 完整 URL 清單
