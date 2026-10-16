# 陆山君的博客

基于 [Hugo](https://gohugo.io/) + [Reimu](https://github.com/D-Sketon/hugo-theme-reimu) 主题的静态博客，
通过 GitHub Actions 自动构建并发布到 GitHub Pages。

- 线上地址：<https://lushanjun1127.github.io/blog/>
- 仓库地址：<https://github.com/lushanjun1127/blog>

## 快速开始

### 环境要求

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Hugo | 0.168.0 **extended** | **必须是 extended 版**，主题用 SCSS，标准版无法构建 |
| Git | 任意较新版本 | 主题以 submodule 形式引入，克隆时需带子模块 |

检查 Hugo 是否为 extended 版：

```bash
hugo version
# 期望输出包含 +extended，例如：
# hugo v0.168.0-...+extended windows/amd64
```

若输出中没有 `+extended`，说明装的是标准版，需要重装：

```bash
# Windows
winget install --id Hugo.Hugo.Extended --accept-source-agreements --accept-package-agreements

# macOS
brew install hugo

# 或从官方下载 extended 版本
# https://github.com/gohugoio/hugo/releases
```

### 克隆与运行

```bash
# 克隆时一并拉取主题子模块（重要）
git clone --recurse-submodules https://github.com/lushanjun1127/blog.git
cd blog

# 启动本地开发服务器，访问 http://localhost:1313/
hugo server --bind 127.0.0.1 --port 1313 --baseURL http://localhost:1313/
```

已经克隆过但忘了带子模块，用这条命令补上：

```bash
git submodule update --init --recursive
```

开发服务器支持热重载，修改内容或配置后浏览器会自动刷新。

## 目录结构

```text
.
├── .github/workflows/hugo.yml   # CI：质量门禁 → 构建 → 部署
├── .markdownlint.json           # Markdown 规范检查规则
├── archetypes/                  # 新建文章的模板
├── config/_default/params.yml   # 主题配置（菜单、作者、评论等）
├── content/                     # 文章与页面（Markdown）
│   ├── post/                    # 文章目录
│   ├── archives/_index.md       # 归档页（不可删除）
│   └── about.md                 # 关于页
├── data/                        # 主题数据
│   ├── covers.yml               # 随机封面图列表
│   ├── friends.yml              # 友链列表
│   └── vendor.yml               # 第三方库 CDN 配置
├── themes/                      # 主题（git submodule，勿直接修改）
│   ├── reimu/                   # 当前启用的主题
│   └── PaperMod/                # 备用主题，未启用
└── hugo.toml                    # 站点主配置
```

## 日常写作

新建一篇文章：

```bash
hugo new post/my-article.md --kind post
```

会在 `content/post/` 下生成带完整 front matter 的草稿。写完后把 `draft = true` 改成 `false`，
本地预览确认无误再提交。

写作规范详见 [docs/内容写作规范.md](docs/内容写作规范.md)。

## 发布流程

**推送即发布。** 向 `main` 分支推送后，CI 自动完成质量检查、构建和部署，约 1 分钟上线。

```bash
git add .
git commit -m "feat: 新增文章《xxx》"
git push origin main
```

发布状态在 [Actions 页面](https://github.com/lushanjun1127/blog/actions) 查看。
质量检查不通过会阻断部署，需要先修复。

提交信息格式、分支策略等详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 文档索引

| 文档 | 内容 |
| --- | --- |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Git 提交规范、分支策略、协作流程 |
| [docs/内容写作规范.md](docs/内容写作规范.md) | Markdown 写作规范、front matter 字段说明 |
| [docs/配置手册.md](docs/配置手册.md) | 站点配置、主题参数、常用定制方法 |
| [docs/运维手册.md](docs/运维手册.md) | CI/CD 说明、故障排查、主题升级 |

## 常见问题

**构建报错 `TOCSS: failed to transform`？**
Hugo 装成了标准版。装 extended 版即可，见上文「环境要求」。

**主题样式不生效、页面错乱？**
子模块没拉取。执行 `git submodule update --init --recursive`。

**推送后线上没更新？**
先看 Actions 是否失败。质量门禁不通过会阻断部署。

**本地正常但线上 404？**
检查 `hugo.toml` 的 `baseURL`，线上是子路径 `/blog/`，本地预览用的是根路径。

## 许可

内容版权归作者所有；站点配置与文档可自由参考。
