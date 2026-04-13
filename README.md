# Ashley-Linn 的 Hexo 博客

> 基于 Hexo + Stellar 主题的个人博客，记录技术实践、成长印记、生活札记与状态管理。

**博客地址**：[https://ashley-linn.github.io/](https://ashley-linn.github.io/)

---

## 一、环境信息

- **系统**：Windows 11 23H2  
- **Node.js**：v20.10.0 (LTS)  
- **Git**：2.43.0.windows.1  
- **部署仓库**：[https://github.com/Ashley-Linn/Ashley-Linn.github.io.git](https://github.com/Ashley-Linn/Ashley-Linn.github.io.git)

---

## 二、完整操作步骤

### 1. 环境安装

- **Git**：下载自 [git-scm.com](https://git-scm.com/download/win)，默认路径安装。
- **Node.js**：下载 [v20.10.0](https://nodejs.org/dist/v20.10.0/node-v20.10.0-x64.msi)，安装时勾选“自动配置 PATH”。

### 2. 初始化与配置

在博客根目录打开 Git Bash，执行：

```bash
npm install -g hexo-cli
hexo init
npm install
```

### 3. 主题安装（Stellar）

bash

```
git clone https://github.com/xaoxuu/hexo-theme-stellar.git themes/stellar
```

在根目录 `_config.yml` 中修改 `theme: stellar`。

**重要**：主题配置需复制一份到根目录并重命名为 `_config.stellar.yml`，所有主题相关修改均在此文件中进行，避免直接修改主题文件夹内的配置（便于主题升级）。

------

## 三、常用命令（个人高频使用）
| 用途               | 命令                             |
| :----------------- | :------------------------------- |
| 本地预览           | `hexo clean && hexo g && hexo s` |
| 部署上线           | `hexo clean && hexo g && hexo d` |
| 新建文章           | `hexo new post "文章标题"`       |
| 新建页面（如板块） | `hexo new page "页面名称"`       |
| 手动创建分类页     | `hexo new page categories`       |
| 手动创建标签页     | `hexo new page tags`             |
| 手动创建归档页     | `hexo new page archives`         |

------

## 四、博客内容管理规范

### 文件夹结构（物理分类）

text

```
source/
├─ tech/          # 技术实践板块首页
├─ growth/        # 成长印记板块首页
├─ life/          # 生活札记板块首页
├─ manage/        # 状态管理板块首页
├─ _posts/        # 所有文章统一存放
│  ├─ tech-xxx.md
│  ├─ growth-xxx.md
│  ├─ life-xxx.md
│  ├─ manage-xxx.md
│  └─ ...
```



### 板块首页示例（`tech/index.md`）

yaml

```
---
title: 技术实践
layout: category
permalink: /tech/
---
### 技术实践
这里记录我的编程学习、技术实操、问题解决笔记。
```

其他板块（`growth/`、`life/`、`manage/`）的 `index.md` 只需替换 `title` 和 `permalink` 即可。

### 文章 Front-matter 完整格式

yaml

```
---
title: 文章标题
date: 2026-04-13 10:00:00   # 发布前手动改成最新时间，不写则使用文件创建时间
updated: 2026-04-13 15:00:00 # 可选，更新时间
category: 技术实践           # 必填，与板块名一致（技术实践/成长印记/生活札记/状态管理）
tags: [Python, 装饰器]       # 可选
slug: custom-url            # 可选，自定义链接（如 /tech/custom-url.html）
cover: /images/cover.jpg    # 可选，文章封面
excerpt: 文章摘要            # 可选
---
```

> **时间戳说明**：Hexo 默认以 `date` 字段为准。若不写 `date`，则使用文件的创建时间（可能不准确）。建议在发布前手动将 `date` 改为当前时间，确保排序正确。

### 写文档规范

- **文件名**：`[板块标识]-[主题]-YYYYMMDD.md`（如 `tech-llm-basic-20260118.md`）
- **内部结构**：用 `##`、`###` 分级，代码块注明语言。

------

## 五、踩坑记录（个人专属）

### 1. 主题配置复制与命名

- **问题**：直接修改 `themes/stellar/_config.yml` 会导致主题升级时配置丢失。
- **解决**：将主题配置文件复制到博客根目录，重命名为 `_config.stellar.yml`。Hexo 会自动合并根目录配置，优先级高于主题自带配置。

### 2. 中间导航栏重复（`index_blog` 的 `base_dir` 未设置）

- **现象**：顶部导航栏和侧边栏出现重复的链接。
- **原因**：主题配置文件（`_config.stellar.yml`）中 `index_blog` 的 `base_dir` 默认为空，导致路径解析异常。
- **解决**：设置 `base_dir: blog`，并确保 `menu` 中的链接使用相对路径。

### 3. footer 社交图标大小与本地化，以及 sitemap 配置

- **问题**：社交图标加载慢或无法显示，大小不一致；底部 sitemap（地图导航）需要正确配置。
- **解决**：
  1. **图标本地化**：将图标文件（如 `github.svg`、`zhihu.svg`）放在 `source/images/icons/` 目录下，避免依赖外部 CDN 导致加载失败。
  2. **图标大小控制**：使用 `img` 标签直接设置 `width` 和 `height` 属性（例如 32px），并添加 `style` 保证对齐：

yaml

```
icon: '<img src="/images/icons/github.svg" width="32" height="32" style="margin-right:8px; vertical-align:middle; pointer-events: none;">'
```

​	3.**底部 sitemap（地图导航）**：在 `_config.stellar.yml` 中配置 `footer.sitemap`，定义导航列（如“博客”、“内容板块”等），每列包含链接列表。示例：

yaml

```
footer:
  sitemap:
    - title: 博客
      items:
        - '[近期发布](/)'
        - '[分类](/categories/)'
        - '[标签](/tags/)'
        - '[归档](/archives/)'
    - title: 内容板块
      items:
        - '[技术实践](/tech/)'
        - '[成长印记](/growth/)'
        - '[生活札记](/life/)'
        - '[状态管理](/manage/)'
    # 可按需添加更多列
```

### 4. 板块主题页（`index.md`）与最近文章链接（`slug`）

- **问题**：板块首页需要手动创建，且自定义文章链接（slug）不生效。
- **解决**：
  - 手动创建 `source/[板块名]/index.md`，设置 `layout: category` 和 `permalink`。
  - 在文章 Front-matter 中使用 `slug` 字段自定义链接，如 `slug: /tech/custom-url.html`。

### 5. 分类、标签、归档页手动创建及路径配置

- **问题**：默认不生成这些页面，导航链接指向 404。
- **解决**：执行 `hexo new page categories` 等命令创建，并在生成的 `index.md` 中设置 `layout: categories/tags/archives`。主题配置中导航链接应为 `/categories/`、`/tags/`、`/archives/`。

### 6. 时间戳与实际发布时间不符

- **原因**：Hexo 以 Front-matter 中的 `date` 字段为准，若不写则使用文件创建时间。
- **解决**：发布前手动将 `date` 改为当前时间，确保文章按正确顺序排列。

### 7. 博客接入 Git 管理修改记录

- **做法**：将整个博客项目初始化为 Git 仓库，每次修改后提交并推送。可通过 `git log` 获取最后更新时间，在主题中显示“最近更新”。

bash

```
git init
git add .
git commit -m "初始提交"
git remote add origin https://github.com/Ashley-Linn/Ashley-Linn.github.io.git
git push -u origin main
```

------

## 六、下一步完善（可选功能）

以下功能可按需逐步添加，提升博客体验：

- **专栏（系列文章）**：在 `_config.stellar.yml` 中定义 `series`，文章 Front-matter 中添加 `series: 系列名`。
- **评论系统**：推荐 Giscus（基于 GitHub Discussions）或 Waline（自托管）。
- **网站统计**：接入 Google Analytics 或 Umami。
- **自定义域名与 Cloudflare CDN**：购买域名，配置 CNAME 并开启 Cloudflare 代理，加速访问。

------

## 七、部署说明（源码与成品分离）

本博客采用 **源码与成品分离** 的部署策略：

- **源码分支**：`master`（存放 Hexo 源文件、主题配置、文章等）
- **成品分支**：`gh-pages`（存放 Hexo 生成的静态页面，由 GitHub Pages 自动发布）

### 部署流程

1. **本地生成静态文件**

   bash

   ```
   hexo clean && hexo g
   ```

2. **将静态文件推送到 `gh-pages` 分支**
   使用 `hexo-deployer-git` 插件，配置 `_config.yml`：

   yaml

   ```
   deploy:
     type: git
     repo: https://github.com/Ashley-Linn/Ashley-Linn.github.io.git
     branch: gh-pages
   ```

   然后执行部署命令：

   bash

   ```
   hexo d
   ```
  
3. **将源码推送到 `master` 分支**

   bash

   ```
   git add .
   git commit -m "更新源码"
   git push origin master
   ```

### GitHub Pages 设置

- 仓库 Settings → Pages → Build and deployment → Source 选择 **Deploy from a branch**
- Branch 选择 `gh-pages`，文件夹选择 `/ (root)`
- 保存后，访问 `https://ashley-linn.github.io/` 即可看到博客

### 注意事项

- 不要将生成的 `public/` 目录提交到 `master` 分支（已在 `.gitignore` 中忽略）。
- 每次修改文章或主题后，需要**先执行 `hexo g` 生成静态文件，再执行 `hexo d` 部署到 `gh-pages`**，同时将源码变更推送到 `master`。
- 如果使用 GitHub Actions 自动化部署，可跳过本地 `hexo d`，但需要配置工作流文件。当前采用手动部署方式。

## 八、参考资料

- [Hexo 官方文档](https://hexo.io/zh-cn/)
- [Stellar 主题文档](https://xaoxuu.com/wiki/stellar/)
- [GitHub Pages 部署](https://docs.github.com/zh/pages)
- [Giscus 评论系统](https://giscus.app/zh-CN)
- [Cloudflare CDN 免费加速](https://www.cloudflare.com/zh-cn/)