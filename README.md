# tvbox-Beauty

TVBox 配置仓库，通过 Cloudflare Pages 对外提供：**https://tvbox-beauty.pages.dev/**

Fork 自 [cluntop/tvbox](https://github.com/cluntop/tvbox)（雨呢）。上游每小时合并一次（见 `.github/workflows/tvbox-beauty-auto-sync.yml`，合并版；丢弃式同步已删除）。

## 6 套配置怎么选

| 配置 | 站点数 | spider | 说明 |
|---|---|---|---|
| [box.json](https://tvbox-beauty.pages.dev/box.json) | 100 | ./jar/xs.jar | 主配置，最全 |
| [aa.json](https://tvbox-beauty.pages.dev/aa.json) | 29 | ./jar/pro.jar | 备用配置 |
| [fun.json](https://tvbox-beauty.pages.dev/fun.json) | 87 | ./pg.jar | 娱乐向 |
| [jsm.json](https://tvbox-beauty.pages.dev/jsm.json) | 79 | ./pg.jar | **本机专用**：29 条直播 + 5 个站点走 `127.0.0.1:10079/35456` 本地代理，只在跑着该代理的设备上能用 |
| [wv.json](https://tvbox-beauty.pages.dev/wv.json) | 15 | ./jar/WvSpider.jar | WebView 爬虫 |
| [test.json](https://tvbox-beauty.pages.dev/test.json) | 9 | ./jar/yt.jar | 测试用 |

TVBox 里订阅任一配置地址即可；`/tv` 会重定向到 `box.json`。

## 目录

- `js/` — JS 爬虫（drpy / WvSpider）。`js/s/` 是 drpy 规则 JSON，`js/ss/` 是 Python 爬虫合集
- `py/` — Python 爬虫。注意：`douban.py` / `guazi.py` / `ytb.py` 依赖的 `base.spider` 模块不在仓库里，由 TVBox/FongMi 运行时提供
- `jar/` / `lib/` — spider jar 与数据文件（含直播 m3u）
- `tools/` — 辅助脚本（`sync-pg.py`：Telegram 同步 PG 资源，需 `PHONE` / `API_ID` / `API_HASH` / `GIT_TOKEN` 环境变量）
- `web/` — 见下

## 已知情况（2026-11-18 整理后）

- **本机专用地址**：`jsm.json` 大量使用 `127.0.0.1` 本地代理，`aa.json` 有 6 条 `localhost:5757/5759` 解析和 1 条 `127.0.0.1:9978` 直播；`box.json` 的弹幕字段已移除。这些只在作者本机能用，公开配置里不要指望它们。
- **两套 aa.json**：根 `aa.json`（29 站）是现行版；`js/aa.json`（37 站）是旧变体，未被任何顶层配置引用，留作参考，不要长期两套并存。
- **继承的 App 签名密钥**：`py/shortfilm.py` 里有上游带来的短剧 App 签名密钥（掌阅短剧等），是公开 App 的共享密钥，不是本仓库持有人的凭证，见 [SECURITY.md](SECURITY.md)。只有 App 方能轮换；轮换后对应源失效。
- **红果**：`hongguo.py`（旧版，App-API + 解密 + 下载）和 `hongguo_fixed.py`（新版，网页版，搜索修复）能力不等价，不要直接替换；迁移前先在 `test.json` 上实测同一批剧集的播放。
- **死文件**：90 个 py、58 个 js、61 个 `js/s/` 规则未被任何顶层配置引用，已核对过的重复/空文件已删除；其余暂时保留（jar 内部引用无法完全排除），删前请先全库检索。
- **`js/xiaosa/`** 是第三方整包（小骚），由 `zip.yml` 同步；该 workflow 已暂停（下载无校验），恢复前需重写。
- **`.md5` 文件**：97 个校验 sidecar，git 本身已校验完整性，留作历史；`md5.yml` 已改为只提交 `.md5` 变更。
- **`nostr.html`**：FongMi 的 WebHome 推荐页（Nostr + TMDB），放根目录仅为历史原因。

## 同步与回滚

- 上游同步：每小时合并 `cluntop/tvbox` 的 main（`tvbox-beauty-auto-sync.yml`）；冲突时 workflow 失败并中止合并，等人工处理。
- 备份分支：`backup/2026-11-18-before-improvement`（2026-11-18 改善前的 main）。不满意时从该分支恢复文件即可。
- 部署：推到 `main` 即自动部署到 Pages；每天 08:00 / 20:00（北京时间）还会各触发一次重新部署。

## 健康检查

`.github/workflows/health-check.yml` 每天检查一次各配置里的 `sites[].api` / `parses[].url` / `lives[].url` 是否可达，只出报告、不自动删。失效清单见 Actions 运行摘要。
