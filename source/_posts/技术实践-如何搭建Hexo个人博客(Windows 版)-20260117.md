---
title: 如何搭建Hexo个人博客(Windows 版)
date: 2026-01-17 00:00:00
category: 技术实践
tags:
  - Hexo
  - 个人博客搭建
  - Windows
  - 静态博客
  - 前端工具
slug: hexo-blog-build
permalink: /2026/01/17/hexo-blog-build/
---

# 一、技术栈与选型说明

## 1. 常见个人博客搭建方式对比

目前主流的个人博客搭建方案各有优劣，核心对比如下：

|   搭建方式   |                           核心特点                           |          适用场景          |
| :----------: | :----------------------------------------------------------: | :------------------------: |
| 静态博客框架 | Hexo/Jekyll/Hugo 等，本地生成静态文件，部署简单、访问速度快、无需服务器运维 | 技术爱好者、轻量化内容输出 |
| 开源博客系统 | WordPress/Typecho 等，动态渲染，功能丰富（插件 / 评论），需服务器 + 数据库 | 内容复杂、需高频交互的博客 |
|   平台托管   | CSDN / 掘金 / GitBook 等，无需搭建，直接创作，但自定义程度低、依赖平台 | 快速输出内容、无需自主域名 |

## 2. 选择 Hexo 的核心原因

- **轻量化易上手**：基于 Node.js，命令行操作简单，新手无需掌握复杂后端知识；

- **性能优异**：生成纯静态 HTML 文件，访问速度远快于动态博客系统；

- **生态丰富**：海量免费主题（如 Next、Butterfly）和插件（评论、统计、搜索），满足个性化需求；

- **部署成本低**：可免费部署至 GitHub Pages/Gitee Pages，无需购买服务器；

- **Markdown 友好**：原生支持 Markdown 编辑，契合技术笔记 / 博客的创作习惯。

  

# 二、搭建过程（windows系统）

## 1.前置环境准备

### （1）安装 Git

- 下载地址：[Git 官方下载页](https://git-scm.com/download/win)

- 安装步骤：

  - 双击安装包，默认路径一路点击「Next」（新手无需修改）；

  - 验证：Win+R 输入 `cmd`，执行 `git --version`，显示版本号（如 `git version 2.43.0.windows.1`）即成功。

### （2）安装 Node.js

- 下载地址：[Node.js 官方下载页](https://nodejs.org/zh-cn/download/)

- 安装步骤：

  - 选择「Windows 安装包 (.msi)」（优先 LTS 长期支持版，稳定性更高）；

  - 双击安装包，勾选「Accept the license agreement」，默认路径一路「Next」；

  - 验证：cmd 中执行 `node -v` 和 `npm -v`，均显示版本号即成功。

### （3）配置全局环境变量

​       Git 和 Node.js 安装时**默认勾选自动配置全局环境变量**，但偶尔会出现配置失败的情况 —— 核心目的是让 `git`/`node`/`npm`/`hexo` 命令在**任意目录**（比如你的博客根目录 `D:\myblog`）都能执行，而非仅在安装目录可用。

**① 检查全局变量是否生效**

   打开 cmd 窗口（无需切换到任何目录），直接执行以下命令：

```
git --version
node -v
npm -v
```

- 若均显示版本号：说明全局环境变量已配置成功，可跳过下一步；
- 若提示「'git' 不是内部或外部命令，也不是可运行的程序」：说明全局变量未配置，需手动添加。

**② 手动配置全局环境变量**

**a.找到安装路径（默认路径参考）：**

- Git：`C:\Program Files\Git\bin`（若自定义安装，需找 `git.exe` 所在目录）；
- Node.js：`C:\Program Files\nodejs\`（`node.exe`/`npm.cmd` 所在目录）。

**b.打开环境变量配置界面：**

​	  右键「此电脑」→「属性」→「高级系统设置」→「环境变量」；

**c.添加全局路径：**

- 在「系统变量」中找到 `Path` 变量，点击「编辑」；
- 点击「新建」，分别粘贴上述 Git 和 Node.js 的安装路径；
- 点击「确定」保存（需逐层确认，避免配置不生效）。

**d.验证配置：**

​       关闭所有已打开的 cmd 窗口（关键！旧窗口不会加载新配置），重新打开 cmd，执行以上验证命令 ，若显示版本号，说明全局环境变量已手动配置成功。

## 2. 安装 Hexo 并初始化博客

### （1）安装 Hexo 核心包

- 新建博客根目录（如 `D:\myblog`）；
- 打开 cmd 窗口，执行 `cd D:\myblog` 切换至根目录；
- 安装 Hexo 脚手架（全局）：


```
npm install -g hexo-cli
```

- 验证：执行 `hexo -v`，显示 Hexo 版本号即安装成功。

### （2）初始化 Hexo 博客

在博客根目录的 cmd 窗口执行：

```
hexo init  # 初始化博客目录结构
npm install  # 安装项目依赖包
```

初始化完成后，核心目录 / 文件说明：

- `source/_posts`：存放 Markdown 格式的博客文章；
- `_config.yml`：博客核心配置文件（主题、部署、站点信息均在此修改）；
- `themes`：存放博客主题文件（可自定义样式）。

### （3）本地启动博客预览

```
hexo clean  # 清理缓存（首次启动可省略，修改配置后建议执行）
hexo g      # 生成静态文件（generate 缩写）
hexo s      # 启动本地服务器（server 缩写）
```

执行后终端显示 `Hexo is running at http://localhost:4000/`，浏览器访问该地址即可看到默认博客页面。



# 三、自定义博客（基础配置）

## 1. 修改博客基本信息

打开根目录 `_config.yml`（建议用 VS Code 编辑，编码设为 UTF-8），修改核心配置（**冒号后必须加空格**）：

```
# 站点基础信息
title: 你的博客标题       # 例：「XX的技术笔记」
subtitle: 博客副标题     # 例：「记录编程与成长」
description: 博客描述     # 用于搜索引擎优化（SEO）
author: 你的名字         # 例：「张三」
language: zh-CN          # 语言设置为中文
timezone: Asia/Shanghai  # 时区设置为上海

# 网址配置（部署时需同步修改）
url: https://你的用户名.github.io # 部署到GitHub Pages后填写
root: /
permalink: :year/:month/:day/:title/  # 文章链接格式
permalink_defaults:
```

## 2. 更换博客主题（以 Next 为例）

Next 是 Hexo 生态最热门的主题之一，轻量化且可定制性强：

**a.下载 Next 主题：**在 Git Bash（或 cmd）中执行

```
git clone https://github.com/next-theme/hexo-theme-next.git themes/next
```

**b.启用主题：**修改根目录 _config.yml中 theme 配置：

```
theme: next  # 替换默认的 landscape
```

**c.重启预览：**执行 以下指令，刷新浏览器即可看到新主题样式。

```
hexo clean && hexo g && hexo s
```



# 四、发布第一篇博客文章

## 1. 新建文章

在 Git Bash/cmd 中执行：

```
hexo new "我的第一篇Hexo博客"  # 生成Markdown文章文件
```

执行后会在 `source/_posts` 目录下自动生成 `我的第一篇Hexo博客.md` 文件，文件开头默认包含一段 `Front-matter` 配置（这是 Hexo 识别文章属性的核心）。

## 2. Front-matter 介绍

`Front-matter` 是 Hexo 特有的「文章元数据配置区」，位于 Markdown 文件最顶部，以 `---` 分隔，采用 YAML 格式编写。Hexo 会优先解析这部分内容，用于定义文章的标题、日期、分类、标签等属性，**无需手动写 HTML 或配置其他文件**。

### （1）Front-matter 核心属性说明

|    属性名    |                             作用                             |                示例                 |
| :----------: | :----------------------------------------------------------: | :---------------------------------: |
|   `title`    |                 文章标题（优先级高于文件名）                 |     `title: 我的第一篇Hexo博客`     |
|    `date`    | 文章发布时间（格式：YYYY-MM-DD HH:MM:SS），默认是文件创建时间 |     `date: 2026-01-17 10:00:00`     |
|    `tags`    |            文章标签（支持多个，用于内容分类检索）            | `tags: [Hexo, 博客搭建]` 或分行写法 |
| `categories` | 文章分类（支持多级，如「教程 / 前端」），分类比标签更具层级性 |         `categories: 教程`          |
| `permalink`  |     自定义文章访问链接（覆盖站点默认的 permalink 规则）      |   `permalink: /2026/hexo-guide/`    |
| `published`  |     是否发布（`false` 则仅本地可见，不生成到静态文件中）     |          `published: true`          |
|  `comments`  |            是否开启评论功能（需主题 / 插件支持）             |          `comments: true`           |

### （2）写法注意事项

- 必须以 `---` 开头和结尾，且位于文件最顶部；

- 采用 YAML 格式，**冒号后必须加空格**（如 `title: 标题` 而非 `title:标题`）；

- 多值属性（如 tags/categories）支持两种写法：

  ```
  # 单行写法（适合少量标签）
  tags: [Hexo, 博客搭建, Windows]
  
  # 分行写法（适合多标签/多级分类，更易读）
  tags:
    - Hexo
    - 博客搭建
    - Windows
  categories:
    - 技术教程
    - 工具使用
  ```

### （3） 编辑文章内容

在 `source/_posts` 目录下找到 `我的第一篇Hexo博客.md`，用 Markdown 编辑器（Typora/VS Code）编辑，完整示例如下：

```
---
title: 我的第一篇Hexo博客
date: 2026-01-17 10:00:00
tags:
  - Hexo
  - 博客搭建
categories:
  - 教程
published: true
comments: true
---

# 欢迎来到我的Hexo博客
这是我用Hexo搭建的第一篇博客，记录搭建过程与心得～
```

### （4） 预览文章

执行 `hexo clean && hexo g && hexo s`，访问 `localhost:4000` 即可查看发布的文章，Hexo 会根据 `Front-matter` 中的配置，自动展示文章标题、分类、标签等信息。



# 五、部署博客到 GitHub Pages

## 1. 准备 GitHub 仓库

- 登录 GitHub 账号，新建仓库，仓库名必须为：`你的用户名.github.io`（全小写，如 `zhangsan.github.io`）；
- 复制仓库的 HTTPS 地址（如 `https://github.com/zhangsan/zhangsan.github.io.git`）。

## 2. 安装 Hexo 部署插件

在博客根目录的 Git Bash/cmd 中执行：

```
npm install hexo-deployer-git --save
```

## 3. 配置部署信息

修改根目录 `_config.yml` 末尾的部署配置：

```
deploy:
  type: git
  repo: https://github.com/你的用户名/你的用户名.github.io.git  # 替换为你的仓库地址
  branch: main  # GitHub 新仓库默认分支为 main（老仓库可能是 master）
```

## 4. 部署博客

执行部署命令：

```
hexo clean && hexo g && hexo d   # d 是 deploy 缩写
```

首次执行会弹出 GitHub 登录窗口，授权后即可完成部署。部署完成后访问 `https://你的用户名.github.io`（等待 1-2 分钟生效），即可看到全网可访问的博客。

