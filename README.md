# 档案管理系统 · GitHub Pages 部署包

一个纯前端单文件应用（157 KB，零依赖、不联网也能打开）。这个包帮你一键部署到 GitHub Pages，部署完得到一个 https 网址，手机微信扫码即可查看并改档案状态。

---

## ⚠️ 最重要的一步（不做一定 404）

**仓库 → Settings → Pages → Build and deployment → Source 选「GitHub Actions」**

如果你现在遇到这个报错：

```
Error: Creating Pages deployment failed
HttpError: Not found
Error: Failed to create deployment (status: 404)
```

那 100% 是这一步没做。日志里"检出代码""上传工件"都成功了，唯独最后"创建页面部署"被拒，
就是因为仓库里还没有 Pages 站点对象。**去 Settings → Pages 把 Source 改成 GitHub Actions，再重跑一次工作流就好了。**

另外确认一下：Settings → Actions → General → Workflow permissions 选 **Read and write permissions**。

---

## 方式一：网页上传（不用装 git，推荐）

1. 打开你的 GitHub 仓库
2. 点 **Add file → Upload files**
3. 把本目录下的这些内容**全部**拖进去：
   - `public/index.html`（程序本体，名字必须是 index.html）
   - `.github/workflows/pages.yml`（自动部署配置）
   - `README.md`、`LICENSE` 等（可选）
4. 点 **Commit changes**
5. 去 **Settings → Pages → Source** 选 **GitHub Actions**（关键！）
6. 打开 **Actions** 标签页，等 1～3 分钟，绿色对勾出现即部署成功
7. 网址是 `https://你的用户名.github.io/仓库名/`

> 网页上传时 `.github` 是隐藏目录，可能不好拖。最简单是先传 `public/index.html`，
> 再在仓库里点 **Add file → Create new file**，文件名填 `.github/workflows/pages.yml`，
> 把本包的 pages.yml 内容粘进去。

## 方式二：用 git 命令行

```bash
git clone https://github.com/你的用户名/仓库名.git
cd 仓库名
# 把本包的 public/ 和 .github/ 复制进来
git add -A
git commit -m "部署档案管理系统"
git push
```

---

## 目录结构说明

```
public/index.html          站点文件，GitHub Pages 只认 index.html 当首页
.github/workflows/pages.yml  自动部署工作流
```

如果你的仓库里 `index.html` 就在根目录（不在 public/ 下），把工作流里的这一行改掉：

```yaml
      - name: 上传站点文件
        uses: actions/upload-pages-artifact@v5
        with:
          path: '.'        # ← 从 ./public 改成 '.'
```

---

## 部署完成后

打开网址 → 「设置」→「页面部署网址」填入 `https://你的用户名.github.io/仓库名/` → 保存。
之后每条档案的二维码都会带上这个网址，微信扫一扫直接打开并改状态。

程序里有「🔍 检测这个网址能不能用」，会自动识别 GitHub Pages、检查文件名和子路径。

---

## 常见问题

| 报错 | 原因 | 解决 |
|---|---|---|
| `Failed to create deployment (status: 404)` | Pages 没启用 / Source 不是 GitHub Actions | Settings → Pages → Source 选 GitHub Actions |
| `Get Pages site failed: Not Found` | 同上 | 同上 |
| `Failed to create deployment (status: 403)` | 缺 `pages: write` 或 `id-token: write` | 工作流里补权限；Settings → Actions → General 选 Read and write |
| `Invalid deployment branch... gh-pages` | Source 还停在分支模式 | 改成 GitHub Actions |
| 部署成功但访问 404 | 上传的目录里没有 index.html | 确认 `public/` 下有 index.html |
| 页面能开但样式乱 | 浏览器缓存 | Ctrl+F5 强制刷新，或换个浏览器 |
| 微信打不开 | github.io 域名偶被拦截 | 换 Gitee Pages / 腾讯云 EdgeOne，或绑定已备案域名 |

改了内容后要等 1～3 分钟才生效（Pages 有 CDN 缓存）。

## 🚨 部署成功但打开网页是 404「未找到文件」

页面显示 `404 · 未找到文件 · 该地址配置的站点则不支持包含请求的文件`。

这跟上一条不同：**工作流已经绿了、部署成功了**，但发布出去的内容里没有 index.html。
按顺序检查 4 项：

**① 网址对不对（最常见）**
项目仓库的网址必须是 `https://用户名.github.io/仓库名/`——末尾**带斜杠**，
用户名和仓库名**大小写完全一致**（GitHub 对大小写敏感）。
用户名/仓库名可在仓库页面顶部或 Settings → Pages 里查看。

**② 看 Artifact 里有没有 index.html**
Actions → 点这次运行 → 页面底部 **Artifacts** → 下载 `github-pages` → 解压看**根目录**有没有 index.html。
如果根目录是空的、或者 index.html 藏在一层子目录里，就是问题所在。

**③ index.html 被嵌套在子目录里**
比如你传的是 `github-deploy/public/index.html`，导致发布根目录多了一层目录。
→ 用本包的**智能定位工作流**：它会自动在仓库里 `find -iname 'index.html'`（忽略大小写、跳过 .git/.github），
复制到统一的 `_site/` 目录再上传，你在哪一层都不影响。

**④ 文件名必须全小写 index.html**
`Index.html`、`INDEX.html` 在 Linux 上是不同文件，GitHub Pages 只认全小写。

> 最简单的办法（不用 Actions）：把 `public/index.html` **移到仓库根目录**，
> Settings → Pages → Source 选「Deploy from a branch」→ 分支 main、目录 `/ (root)` → Save。

## 🚨 最常见的报错：404 Failed to create deployment

**根因只有一个**：仓库的 GitHub Pages 没启用，或 Source 还停在「Deploy from a branch」。
工作流本身没问题（检出、上传都成功），缺的是仓库设置。

**修复（30 秒）**：Settings → Pages → Source 选 **GitHub Actions** → Actions 页点 Re-run failed jobs。

## 目录结构与两种用法

```
github-deploy/
├── .github/workflows/pages.yml   ← GitHub Actions 工作流
├── .nojekyll                     ← 跳过 Jekyll 构建（避免被改写）
├── README.md
└── public/
    └── index.html                ← 程序本体（已是最新）
```

**方式 A：直接用分支（最简单，推荐先试这个）**
把 `public/index.html` **移到仓库根目录**，然后 Settings → Pages → Source 选
「Deploy from a branch」→ 分支 main、目录 / (root) → Save。等 1～3 分钟。

**方式 B：用 GitHub Actions（自动化）**
保持本目录结构整体上传到仓库根目录，然后**务必**先把
Settings → Pages → Source 改成 **GitHub Actions**（否则报 404），
再 Actions → Re-run failed jobs。
工作流里 `path: ./public`，只发布 public/ 目录，不会把工作流文件一起传上去。
