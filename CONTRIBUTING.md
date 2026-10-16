# 协作规范

本项目是静态博客，协作模式比常规软件项目简单，但仍有需要统一的规范，
以保证历史可追溯、发布可回滚、多人协作不冲突。

## 一、分支策略

采用简化的分支模型，只有两类长期分支：

| 分支 | 用途 | 部署 |
| --- | --- | --- |
| `main` | 唯一长期分支，始终可发布 | 推送即自动部署 |
| `feat/*`、`fix/*`、`docs/*` | 临时分支，用完即删 | 不部署 |

### 何时直接推到 main

改动小、风险低、且你能立刻验证时，直接推到 `main`：

- 新增或修改文章
- 修错别字
- 调整主题参数

### 何时开临时分支

以下情况先开分支，走 Pull Request：

- 更换主题或大改站点配置
- 调整 CI 流水线
- 涉及结构性改动（目录调整、字段重命名）

流程：

```bash
git switch -c feat/switch-theme
# ... 改动 ...
git push -u origin feat/switch-theme
# 在 GitHub 上开 PR，CI 会自动跑质量门禁
```

PR 的描述里写清楚：**改了什么、为什么改、怎么验证**。

## 二、提交信息规范

采用 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/) 格式：

```text
<类型>: <简短描述>

[可选的正文]

[可选的脚注]
```

### 类型

| 类型 | 用途 | 示例 |
| --- | --- | --- |
| `feat` | 新增内容或功能 | `feat: 新增文章《Hugo 构建优化》` |
| `fix` | 修复问题 | `fix: 修正 CI 缓存 key 未包含配置文件的问题` |
| `docs` | 只改文档 | `docs: 补充主题升级步骤` |
| `style` | 只改格式，不影响行为 | `style: 统一 Markdown 列表缩进` |
| `refactor` | 重构，不改变对外行为 | `refactor: 重组 content 目录结构` |
| `chore` | 构建、依赖、配置变动 | `chore: 升级 Hugo 至 0.168.0` |
| `ci` | 只改 CI 配置 | `ci: 增加构建后死链检查` |

### 要点

- 描述用**中文**，简洁说明做了什么，不要写「update」「修改」这类无信息量的词
- 描述末尾**不加句号**
- 一次提交只做一件事，不要把「换主题」和「改错别字」混在一起
- 正文每行不超过 72 字符，便于 `git log` 阅读

### 反例

```text
# 差：无信息量
update
修改了一下
fix bug

# 好：一眼看出影响范围
feat: 新增文章《Reimu 主题上手记录》
fix: 修正归档页在无文章时报错的问题
ci: 缓存 Hugo resources 目录，构建耗时从 60s 降至 25s
```

## 三、提交前检查清单

每次推送前，确认以下四项：

- [ ] 本地 `hugo server` 预览正常，页面无错乱
- [ ] 草稿已把 `draft` 改为 `false`（不想发布的保持 `true`）
- [ ] 文章 front matter 的 `categories` 和 `tags` 已填
- [ ] 提交信息符合上面的规范

推送后到 [Actions](https://github.com/lushanjun1127/blog/actions) 确认流水线通过。
若失败，优先看 **Quality gate** 环节的日志。

## 四、内容与配置的边界

这条最容易踩坑，务必分清：

| 改动对象 | 可否直接改 | 说明 |
| --- | --- | --- |
| `content/` 下的 Markdown | ✅ 直接改 | 这是你的内容 |
| `config/_default/params.yml` | ✅ 直接改 | 主题参数，站点级配置 |
| `data/` 下的 yml | ✅ 直接改 | 友链、封面、CDN |
| `hugo.toml` | ✅ 直接改 | 站点主配置 |
| `themes/` 下的文件 | ❌ **禁止直接改** | 是 submodule，改了会在升级时丢失 |

`themes/` 是外部依赖。要覆盖主题的某个文件（模板、样式、静态资源），
做法是**在项目根的对应目录放同名文件**，Hugo 会优先使用项目根的文件：

```text
# 例如要覆盖主题的 layouts/partials/head.html
layouts/partials/head.html        # 项目根，优先级高
themes/reimu/layouts/partials/head.html   # 主题内，被覆盖
```

这样主题升级时你的定制不会丢。

## 五、回滚

线上出问题时，用 `git revert` 撤销（保留历史，优于 `reset`）：

```bash
git log --oneline -10          # 找到要撤销的提交
git revert <commit-hash>       # 生成一个反向提交
git push origin main           # 推送后自动重新部署
```

紧急情况下也可以在 GitHub Actions 页面找到上一个成功的运行，
用 `workflow_dispatch` 手动触发一次部署。

## 六、不要提交的东西

以下内容已在 `.gitignore` 中排除，请勿强制添加：

| 路径 | 原因 |
| --- | --- |
| `public/` | 构建产物，CI 自己生成 |
| `resources/` | Hugo 的 SCSS 编译缓存，体积大且可重建 |
| `.hugo_build.lock` | 构建锁文件，无意义 |
| `.workbuddy-ai/` | 本地工具产生的目录 |

若不慎提交了 `public/` 或 `resources/`，用以下命令移除：

```bash
git rm -r --cached public resources
git commit -m "chore: 停止追踪构建产物"
```
