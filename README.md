# site/ —— SAT Lens 社区页（GitHub Pages 用）

同一个「加入 SAT Lens 社区」页面，两处投放：

| 投放位置 | 文件 | 说明 |
| --- | --- | --- |
| 扩展内（点弹窗 / 设置 → 意见反馈里的入口） | `join.html` → `src/dashboard/JoinGroupPage.tsx` | 随扩展打包（离线也能开），但**二维码图是从在线地址现取的**，取不到才用打包快照 |
| 网页（GitHub Pages，可对外发链接） | `site/index.html` | 纯静态、零依赖、相对路径，扔到任何静态托管都能跑 |

两份内容一致（文案同源，2026-09-30 用户逐字给定）。**改文案时两边都要改**，
这是刻意的：扩展内那份要能离线、网页那份要能被任何地方引用，不共用构建。

> 关键点：扩展里的二维码是**在线热更**的（见文末「换二维码」）。
> 托管只需要做一次，之后每 7 天换图只动仓库里的 `group_qr.jpg`，**不用重新发扩展**。

---

## 已经发布好了（2026-09-30）

| 东西 | 地址 |
| --- | --- |
| 社区页（可对外发链接） | https://alan231321.github.io/sat-lens-community/ |
| **群二维码图片（换的就是它）** | https://alan231321.github.io/sat-lens-community/group_qr.jpg |
| CDN 镜像（国内兜底，扩展里配的第二条） | https://cdn.jsdelivr.net/gh/Alan231321/sat-lens-community@main/group_qr.jpg |
| 仓库 | https://github.com/Alan231321/sat-lens-community |

站点源：仓库的 `main` 分支根目录（Deploy from a branch）。改哪个文件、网站就跟着变，无需任何构建。

### 换二维码：三种做法（都不碰扩展）

1. **GitHub 网页（最快）**：打开仓库 → Add file → Upload files → 把新图拖进去（同名 `group_qr.jpg` 覆盖）→ Commit changes。约 30 秒。
2. **本地脚本**：`node scripts/gh-pages-setup.mjs --qr 新的二维码.jpg`（需要 `.gh_token`）。
3. **交给 AI**：把新图放到工作区里说一声即可。

> 扩展侧是 `fetch(..., { cache: 'no-store' })`，所以**用户刷新二维码页就是新图**（Pages 那 10 分钟缓存被绕开了）。
> jsDelivr 镜像的边缘缓存是 12 小时，只有主源（github.io）被墙时才会用到它。

---

## 怎么发布到 GitHub Pages（本文件是原始说明，实际已经按方式 0 发完了）

### 方式 0：一条命令（建仓库 + 传文件 + 开 Pages 全自动）

```bash
# 1) 建一个 Fine-grained token（建议只勾这一个仓库）
#    https://github.com/settings/personal-access-tokens/new
#    Permissions: Contents 读写 + Pages 读写（要脚本帮建仓库才需要 Administration 读写）
# 2) 把 token 写进仓库根目录的 .gh_token（已在 .gitignore 里，不会被提交）
# 3) 跑：
node scripts/gh-pages-setup.mjs                 # 首次：建仓库 + 发布 + 开 Pages，最后打印地址
node scripts/gh-pages-setup.mjs --dry-run       # 只看计划，不联网写任何东西
node scripts/gh-pages-setup.mjs --qr 新码.jpg    # 以后：只换二维码图片
```

脚本从不打印 token，只放进 Authorization 头；token 两端的 `< >` 之类的复制噪声会自动剥掉。

### 方式 A：本仓库直接发（已配好 workflow）

`.github/workflows/pages.yml` 会在 push 到 `master` / `main` 时把 **`site/` 整个目录**
发布成站点（只发 `site/`，不会把仓库里的其他东西、debug 导出、浏览器 profile 带上）。

1. 在 GitHub 上建一个仓库（**公开**仓库才能免费用 Pages；私有仓库需要 GitHub Pro/Team）；
2. 本地加远程并推送：

   ```bash
   git remote add origin https://github.com/<你的用户名>/<仓库名>.git
   git push -u origin master
   ```

   > ⚠️ 本仓库 `.git` 有 **1.2 GB**（`plans/real-test-profile*/` 里存了整个 Chrome profile 的缓存，
   > 已在历史里）。直接推会非常慢甚至失败。**推荐方式 B。**

3. 仓库 → Settings → Pages → Source 选 **GitHub Actions**；
4. Actions 里跑一次 `Deploy community site to GitHub Pages`，站点地址是
   `https://<你的用户名>.github.io/<仓库名>/`。

### 方式 B：单独一个小仓库（推荐）

`site/` 是自包含的（`index.html` + `group_qr.jpg`），可以独立成一个几百 KB 的仓库：

```bash
# 在空目录里
cp -r <本仓库>/site/* ./            # 把 site 里的东西放到仓库根
git init && git add -A && git commit -m "feat: SAT Lens 社区页"
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

发布方式二选一：

- **Settings → Pages → Source = Deploy from a branch**，分支选 `main`、目录选 `/ (root)`；
- 或者把 `.github/workflows/pages.yml` 一起放进这个仓库，Source 选 GitHub Actions。

用单独的仓库时，workflow 里的 `path: site` 要改成 `path: .`。

### 本地预览

```bash
# 直接双击打开也行（相对路径，file:// 下同样能显示二维码和保存）
npx serve site
```

---

## 换二维码（7 天一次，**改网页就行，不用重发扩展**）

微信群二维码 **7 天就过期**（图片上那行「该二维码 7 天内…有效」是微信生成的）。
2026-09-30 起，扩展不再「打印」二维码，而是每次打开页面**从网址现取**：

```
扩展（join.html / 反馈页缩略图）
   ├─ 1. 在线地址 GROUP_QR_REMOTE_URLS[0]   ← 你换的就是这里（GitHub Pages）
   ├─ 2. 在线镜像 GROUP_QR_REMOTE_URLS[1]   ← 可选，github.io 国内不稳时的兜底
   └─ 3. 打包快照 src/dashboard/assets/group_qr.jpg  ← 离线兜底（可以一直是旧的）
```

所以日常换码只有一步 —— **把仓库里的 `group_qr.jpg` 换掉**，三种做法任选：

| 做法 | 操作 | 耗时 |
| --- | --- | --- |
| GitHub 网页（推荐） | 打开仓库 → 点 `group_qr.jpg` → 右上「…」→ Upload / 直接把新图拖进仓库 → Commit | ~30 秒 |
| 本地脚本 | `node scripts/gh-pages-setup.mjs --qr 新的二维码.jpg`（需要 `.gh_token`） | ~5 秒 |
| 让我来做 | 把新图给我，`.gh_token` 在位我就直接换 | — |

> ⚠️ 图片文件名**必须还是 `group_qr.jpg`**（URL 不变），只是内容换掉。
> GitHub Pages 的 `Cache-Control: max-age=600` 约 10 分钟；扩展侧用的是
> `fetch(..., { cache: 'no-store' })`，所以**用户刷新二维码页就是最新的**。

### 什么时候才需要重新构建扩展
只有这三种情况，其余一律不用：
1. 换**托管地址**（换仓库 / 换域名）→ 改 `src/shared/group_invite.ts` 的 `GROUP_QR_REMOTE_URLS`；
2. 想刷新**离线兜底快照**（`src/dashboard/assets/group_qr.jpg`）—— 不刷也行，只是断网用户看到旧码；
3. 想让老用户**再收到一次**邀请弹窗 → 把 `GROUP_INVITE_VERSION` 改成新值
   （例如 `sat-lens-2026-10` → `sat-lens-2026-11`）。「只弹一次」是按这个版本号记的，
   不改的话老用户永远不会再被邀请，但仍能自己走「设置 → 意见反馈」看到最新码。

> 建议：每换 2~3 次码顺手刷一次「离线兜底快照」，这样断网/被墙的用户也不会扫到太旧的码。
