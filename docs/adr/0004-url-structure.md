# ADR-0004：URL 結構與目錄組織

- **狀態**：Accepted
- **日期**：2026-05-05

## Context

從單頁重構為多頁面後（[ADR-0001](./0001-multi-page-seo-architecture.md)），需要決定每個頁面的 URL 與檔案目錄結構。URL 一旦發布並被搜尋引擎索引，**改動成本極高**（要做 301 redirect、SEO 信用會短期下降），所以一開始就要設計好。

## Decision

採用 **語義化目錄分組 + trailing slash** 的結構：

```
/                                              (首頁)
/areas/{slug}/                                (區域頁)
/services/{slug}/                             (服務頁)
/blog/                                        (部落格首頁)
/blog/{slug}/                                 (文章頁)
/sitemap.xml
/robots.txt
/llms.txt
```

具體實作：每個頁面是一個 `index.html` 放在對應目錄下：

```
yijibang/
├── index.html                                          (首頁)
├── areas/
│   ├── jian/index.html
│   ├── hualien-city/index.html
│   ├── shoufeng/index.html
│   └── zhixue/index.html
├── services/
│   ├── electronic-lock/index.html
│   ├── chip-key/index.html
│   ├── rolling-code/index.html
│   ├── access-card/index.html
│   └── stamps/index.html
├── blog/
│   ├── index.html
│   └── hualien-electronic-lock-guide/index.html
├── sitemap.xml
├── robots.txt
└── og-image.jpg
```

### Slug 命名規則

- **區域**：用拼音 `jian`、`hualien-city`、`shoufeng`、`zhixue`（不用中文、不用注音）
- **服務**：用英文具體描述，連字號分隔 `electronic-lock`、`chip-key`、`rolling-code`、`access-card`、`stamps`
- **部落格文章**：英文 keyword + slug `hualien-electronic-lock-guide`

### Trailing slash

所有 URL 結尾統一帶 `/`：`/areas/jian/`，不是 `/areas/jian` 或 `/areas/jian.html`。

## Consequences

### 好處

- **語義化結構**：URL 本身就在描述「這是某類別的某個東西」，使用者一看就懂
- **SEO 友好**：搜尋引擎用目錄結構推斷主題權威（`/services/` 下所有頁面互相強化「服務」這個 topic）
- **可擴充**：未來加新區域只要 `/areas/new-slug/`、加新服務只要 `/services/new-slug/`，不影響既有 URL
- **URL 短**：純英文 slug 在 LINE / Facebook 分享時不會被 URL encode 成醜醜的 `%E5%90%89%E5%AE%89`
- **trailing slash 一致性**：避免 Netlify 把 `/areas/jian` 自動 301 redirect 到 `/areas/jian/` 造成的雙重 URL 問題

### 壞處／成本

- 每個頁面要建一個資料夾（不能直接放 `services/electronic-lock.html`）—— 12 個頁面就有 11 個資料夾
- 中文 slug 對純中文使用者比較友善，但目前選英文 slug

### 緩解策略

- canonical URL 統一為 trailing slash 版本，避免 SEO duplicate content 問題
- sitemap.xml 列出的 URL 也統一 trailing slash

## Alternatives Considered

### 扁平結構（無目錄分組）

```
/jian.html
/hualien-city.html
/electronic-lock.html
/chip-key.html
```

**否決原因**：

- 失去類別語義
- 未來若想新增「ji'an」這類接近的 slug，會混亂
- `.html` 副檔名洩露實作細節

### 中文 slug

```
/areas/吉安/
/services/電子鎖/
```

**否決原因**：

- 跨平台分享時被 percent-encode 成 `/areas/%E5%90%89%E5%AE%89/`，難看且 URL 變超長
- 部分 SEO 工具與 Schema validator 對中文 URL 支援較差
- 對英文母語者完全不友善（雖然主要客群是中文）

### Hash routing（`/#/areas/jian`）

**否決原因**：hash 後面的內容搜尋引擎不索引，等於只有首頁有 SEO 效果——違反 [ADR-0001](./0001-multi-page-seo-architecture.md) 的目的。

## References

- [sitemap.xml](../../sitemap.xml)
- [robots.txt](../../robots.txt)
