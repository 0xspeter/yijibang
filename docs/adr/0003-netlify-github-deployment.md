# ADR-0003：部署架構：Netlify + GitHub auto-deploy

- **狀態**：Accepted
- **日期**：2026-05-05

## Context

舊版網站本來就是部署在 Netlify（`yijibang.netlify.app`），透過 GitHub auto-deploy。但前 GitHub 帳號 `page5888` 被停權後：

- GitHub source 失效，Netlify 無法 fetch 更新
- 網站卡在 2026-03-23 的舊版本

需要重新建立部署管線，並決定要不要繼續用 Netlify。

## Decision

繼續用 **Netlify + GitHub auto-deploy**，但 source 改連到新帳號 `0xspeter/yijibang`：

```
本地端 (D:\網頁製作\yijibang\)
    ↓ git push
GitHub (github.com/0xspeter/yijibang)
    ↓ webhook (Netlify GitHub App)
Netlify build
    ↓ deploy
yijibang.netlify.app  (live)
```

具體設定：

1. 安裝 Netlify GitHub App 到 0xspeter 帳號（取代失效的 page5888 連結）
2. 在 Netlify Project configuration → Build & deploy 把 repo 改連到 `0xspeter/yijibang`
3. Branch to deploy: `main`
4. Build command: 留空（靜態 HTML，不需 build）
5. Publish directory: 留空（serve from root）

每次 `git push origin main` → Netlify 自動部署，30-60 秒上線。

## Consequences

### 好處

- **零維運**：靜態檔由 Netlify CDN 全球加速
- **HTTPS 自動**：免費 SSL 證書自動續簽
- **保留原 URL**：`yijibang.netlify.app` 不變，維持已建立的 SEO 累積
- **Atomic deploys**：每次部署是原子操作，可一鍵回滾
- **PR previews**（未啟用但可用）：未來若做 PR 流程，每個 PR 自動產生預覽 URL

### 壞處／成本

- 綁定 Netlify 平台。若 Netlify 政策改變或漲價，要遷移
- 自訂網域沒接（仍在 `*.netlify.app`）→ SEO 信用度略低於有自家網域
- 免費方案有頻寬上限（100 GB/月），超過會被暫停或要付費

### 緩解策略

- HTML/CSS 純靜態，未來要遷到 Vercel / Cloudflare Pages / 自家 VPS / GitHub Pages 都很簡單，沒有 lock-in
- 流量目前遠低於 100 GB/月，免費方案綽綽有餘
- 若要接自家網域：到 Netlify Domain settings 加入並改 DNS，30 分鐘完成

## Alternatives Considered

### GitHub Pages

**否決原因**：

1. URL 變成 `0xspeter.github.io/yijibang/` 或要設自家網域
2. HTML 內所有 canonical URL、OG URL 都寫死 `yijibang.netlify.app`，遷移要改 12 個檔
3. 已建立的 SEO 累積（部分被 Google 索引）會因換 URL 失效，恢復要等 1-3 個月

### Vercel

**否決原因**：功能與 Netlify 接近，但已有 Netlify 帳號與 project，遷移無實質好處。

### Cloudflare Pages

**否決原因**：CDN 速度更快但介面較陽春；同上，無遷移誘因。

### 自架 VPS（DigitalOcean / Linode）

**否決原因**：對純靜態網站是過度工程；要自己處理 HTTPS、防火牆、安全更新；月費高於 Netlify 免費方案。

## References

- 部署 trigger：`git push origin main`
- Netlify project URL：https://app.netlify.com/projects/yijibang
- GitHub repo：https://github.com/0xspeter/yijibang
