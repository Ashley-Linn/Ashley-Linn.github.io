---
title: 从零搭建个人AI信息简报系统：RSSHub + OpenClaw + DeepSeek 实战
category: 技术实践
tags:
  - RSSHub
  - OpenClaw
  - DeepSeek
  - 自动化
  - 新闻摘要
  - RSS
  - 飞书机器人
  - Obsidian
  - AI Agent
  - 工程实践
slug: rss-summary
permalink: /2026/04/13/rss-summary/
---

> 每天被海量信息淹没？时政、医药、投资、AI……想高效获取真正有用的内容，却总在刷公众号、翻网页中浪费一小时？本文记录了我从“信息焦虑”到“自动化简报”的完整折腾过程，包含三种技术方案对比、完整部署命令以及避坑指南。无论你是想快速实现，还是了解背后的技术选型，都能找到答案。

---

## 目录

1. [背景](#sec1)
2. [方案选型](#sec2)
3. [环境准备](#sec3)
4. [部署RSSHub](#sec4)
5. [攻克微信公众号](#sec5)
6. [验证RSS源的有效性](#sec6)
7. [接入OpenClaw + DeepSeek](#sec7)
8. [成本与效果](#sec8)
9. [持续优化与维护](#sec9)
10. [总结与展望](#sec10)
11. [关键卡点与避坑指南](#sec11)
12. [工程化实践与经验总结](#sec12)

---

<a id="sec1"></a>
## 一、背景：为什么我需要一个自动化简报？

作为一个关注**国内外时政、医药健康、投资、人工智能**四个领域的开发者，我每天面临两个痛点：

1. **信息源分散**：公众号、官网、RSS、社群……切来切去，注意力被撕碎。
2. **阅读效率低**：很多文章看完标题就够，却忍不住点进去浪费时间。

我的理想状态是：**每天早上收到一份简报，每条新闻配有AI生成的一句话摘要，10分钟读完，重要内容再深度阅读。**

于是，我决定动手搭建一套**自动化信息简报系统**，并设定了以下核心要求：

- 覆盖四大领域：国内外时政与社会民生、医药健康、投资财经、人工智能
- 由 AI 生成结构化摘要，10 分钟内完成精读
- 输出 Markdown 格式，支持飞书推送与 Obsidian 长期归档
- 成本低（使用DeepSeek API，月成本<2元）

---

<a id="sec2"></a>
## 二、方案选型：三种技术路径对比

在动手之前，我对比了三种主流方案：

| 对比维度         | 🐍 Python脚本 + GitHub Actions    | ⚙️ n8n（自托管）            | 🦞 OpenClaw（AI Agent）               |
| :--------------- | :------------------------------- | :------------------------- | :----------------------------------- |
| **💰 月度成本**   | **0元**（GitHub免费额度足够）    | **约25-40元**（最低配VPS） | **极低**（DeepSeek API，约1-2元/月） |
| **📦 输出质量**   | 高（代码固定，格式完全可控）     | 高（与Python同级）         | 中（Agent可能偶尔偏离格式）          |
| **🧠 专业性体现** | 扎实的工程能力                   | 架构视野、低代码编排       | 前沿探索、AI Agent                   |
| **🛠️ 调试维护**   | 需改代码、push、重新运行         | 可视化拖拽调整             | 调Prompt，反馈较慢                   |
| **⚡ 适合任务**   | 固定格式、重复推送               | 多步骤、多条件分支         | 开放式、需要推理的任务               |
| **🔄 稳定性**     | 极高（GitHub Actions SLA 99.9%） | 较高（取决于自托管服务器） | 中等（模型输出可能变化）             |

### 最终选型

考虑到我的需求是**固定格式的每日新闻推送**，且我愿意接受少量调试成本，同时希望展示前沿技术能力，我决定采用 **OpenClaw + DeepSeek** 作为唯一方案。该方案使用DeepSeek API（约￥1/百万Token），每日推送成本<0.01元。

---

<a id="sec3"></a>
## 三、环境准备：WSL2 + Docker（纯命令行，避雷桌面端）

> ⚠️ **重要避坑**：千万不要安装 Docker Desktop 的 Windows 图形界面版本！它会占用大量内存（2-4GB），导致电脑卡顿，且在 WSL2 中经常出现网络冲突。正确做法是直接在 WSL2 内部安装 Docker Engine（命令行版），轻量、稳定、无图形界面。

**在 WSL2（Ubuntu 22.04）中安装 Docker Engine：**

```bash
# 更新软件源
sudo apt update
sudo apt install -y ca-certificates curl

# 安装 Docker 官方 GPG 密钥
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 添加 Docker 仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 将当前用户加入 docker 组，避免每次 sudo
sudo usermod -aG docker $USER
newgrp docker
```

**验证安装：**

```bash
docker --version
docker compose version
```

**配置国内镜像加速（解决拉取慢）：**

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.xuanyuan.me", "https://docker.1ms.run"]
}
EOF
sudo systemctl restart docker
```

> 备注：目前国内镜像源容易down，可选择走代理。  
> 至此，无需任何图形界面，所有操作在 WSL 终端中完成，资源占用极低（Docker 守护进程仅占约 100MB 内存）。

---

<a id="sec4"></a>
## 四、部署RSSHub：本地万能RSS工厂

RSSHub能把几乎所有网站（微博、B站、公众号、知乎等）转为标准RSS源。

### 4.1 拉取并运行容器

```bash
docker pull diygod/rsshub:latest
docker run -d --name rsshub -p 1200:1200 diygod/rsshub
docker update --restart=always rsshub   # 开机自启
```

### 4.2 验证

浏览器打开 `http://localhost:1200`，看到欢迎页即成功。

### 4.3 使用示例

- 订阅“36氪”：`http://rsshub:1200/36kr/news/latest`

---

<a id="sec5"></a>
## 五、攻克微信公众号：WeWe-RSS及其他工具

大部分公众号不提供RSS，这里提供三种解决方案。

### 5.1 方案一：WeWe-RSS（自托管，最稳定）

WeWe-RSS基于微信读书接口，是目前最稳定的公众号RSS化方案。

**部署命令**：

```bash
docker run -d \
  --name wewe-rss \
  -p 4000:4000 \
  -e AUTH_CODE=your_password \
  cooderl/wewe-rss:latest
```

**配置步骤**：

1. 浏览器打开 `http://localhost:4000`，输入授权码（即 `your_password`）
2. **使用微信读书App扫码登录**（注意：不是普通微信）
3. 在后台搜索公众号名称（如“新华社”、“中国药闻”），点击订阅
4. 系统会自动生成RSS地址，复制到你的RSS阅读器即可

> ⚠️ 注意：合理控制更新频率（每小时不超过1次），不要大量订阅。

### 5.2 方案二：浏览器插件 RSSHub Radar（一键生成）

这是官方配套工具，自动识别页面RSS路由。

- 安装：Chrome/Edge扩展商店搜 **RSSHub Radar**
- 使用：打开目标页面（如公众号文章页）→ 点击插件图标 → 自动显示可用RSS路由 → 复制链接
- 设置：插件里将默认实例改为稳定国内镜像，例如 `https://rsshub.rssforever.com`

### 5.3 方案三：在线生成工具（无需部署，通用网页生成，可视化点选）

- PolitePol：https://politepol.com
- FetchRSS：https://fetchrss.com
- 用法：粘贴网页 URL → 鼠标点选标题 / 内容 → 生成 RSS。
- 适合：RSSHub 没适配、或不想拼路由的小众网站。

---

<a id="sec6"></a>
## 六、验证RSS源的有效性

获取RSS地址后，必须确认其可用。提供三种验证方式：

### 方式一：在线验证器（推荐）

- https://validator.w3.org/feed/
- https://www.feedvalidator.org/

### 方式二：直接浏览器打开

- 将RSS地址粘贴到浏览器地址栏，回车
- 正常显示XML代码（包含`<rss>`、`<channel>`、`<item>`标签）→ 有效
- 显示404或空白页 → 无效

### 方式三：Python脚本测试（适合批量验证）

```python
import feedparser

def test_rss(url):
    feed = feedparser.parse(url)
    if feed.bozo:
        print(f"❌ 无效: {feed.bozo_exception}")
    else:
        print(f"✅ 有效，共 {len(feed.entries)} 条")
        for entry in feed.entries[:2]:
            print(f"  - {entry.title}")

test_rss("https://www.qbitai.com/feed")
```

---

<a id="sec7"></a>
## 七、接入OpenClaw + DeepSeek：让AI生成每日简报

OpenClaw是一个开源的AI Agent框架，支持自然语言定义任务、定时执行、工具调用。

### 7.1 整体架构图

以下是最终实现的系统架构，清晰展示了数据流、两种工作模式以及飞书双向交互。

![[/images/docs/项目架构.png]]

> 详细说明：模式A使用 FreshRSS 聚合源（可开启白名单筛选），模式B直接抓取独立 URL 列表。成功时调用 DeepSeek 生成日报并推送；失败时自动 fallback 到 `agent-search` 联网搜索。

### 7.2 配置DeepSeek API

1. 注册 [DeepSeek](https://platform.deepseek.com/) 获取API Key
2. 在OpenClaw配置文件中添加：

```yaml
llm:
  provider: deepseek
  model: deepseek-chat
  api_key: sk-xxxxxx
```

### 7.3 编写自定义技能（`rss-summary`）

技能的核心脚本 `fetch_and_push.py` 实现了：

- 支持两种模式（统一聚合/直接抓取）
- 动态字符限制（总输入≤4000字符）
- 网络请求超时（30-60秒）与重试机制
- DeepSeek API调用失败时降级为原始列表
- 飞书单聊推送（自动拆分超长消息）
- Obsidian 归档（Markdown文件）

**查看技能是否启用**

```bash
openclaw skills list | grep rss-summary
```

### 7.4 设置定时执行

```bash
openclaw cron add \
  --name "定时RSS汇总" \
  --cron "30 7 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "运行技能 rss-summary" \
  --no-deliver
```
**完整参数说明：**

- `--name "定时RSS汇总"`：任务名（必填）
- `--cron "30 7 * * *"`：每天 07:30 执行（必填）
- `--tz "Asia/Shanghai"`：时区（强烈建议）
- `--session isolated`：独立会话（推荐）
- `--message "运行技能 rss-summary"`：指令内容
- `--no-deliver`：静默执行，不额外推送

### 7.5 测试方式

通过手动执行脚本和 OpenClaw 技能调用进行功能验证。

检查飞书是否收到日报，Obsidian目录下是否生成 `Daily News/YYYY-MM-DD.md`。

---

<a id="sec8"></a>
## 八、成本与效果

- **DeepSeek API**：每日推送约消耗5000 token，月成本 **< 0.2元**。
- **Docker容器**：占用内存约200MB，对开发机几乎无影响。
- **阅读效率提升**：日均信息阅读时间从 60min 降至 10min 内

---

<a id="sec9"></a>
## 九、持续优化与维护

- 每周检查 RSS 源有效性
- 根据推送质量调整 Prompt 与源列表
- DeepSeek 后台设置消费上限
- 定期查看 OpenClaw 运行日志
- Obsidian 知识库定期整理

---

<a id="sec10"></a>
## 十、总结与展望

通过这套系统，我实现了：

1. **信息聚合**：统一入口，不再碎片化浏览
2. **AI摘要**：快速筛选价值内容
3. **低成本自动化**：每月不到0.2元，全自动运行

**未来展望：从本地部署到在线云服务**

目前系统完全运行在我的本地 WSL2 环境中，虽然稳定，但存在两个限制：

- 需要电脑开机且 OpenClaw 网关运行才能定时推送
- 无法提供给他人使用

下一步计划将系统迁移到**云服务器**（如阿里云 ECS、腾讯云轻量应用服务器），实现：

- 7×24 小时无人值守运行
- 通过公网域名或 API 网关对外提供服务
- 支持多用户订阅（每个用户可配置自己的 RSS 源和推送渠道）

> **未来改进方向：代码健壮性与扩展性**
>
> 当前脚本为了控制 token 消耗和响应时间，采用了动态字符限制（`MAX_CHARS=4000`），这在个人使用场景下足够。但若订阅源数量激增（例如超过 50 个），或单篇文章摘要过长，可能面临以下问题：
>
> - 输入截断导致重要信息丢失
> - DeepSeek 上下文窗口（通常 32K-64K）未被充分利用
> - 降级策略过于粗糙，可能影响用户体验
>
> 未来可以从以下几方面优化：
>
> 1. **自适应截断策略**：根据文章数量和重要性动态调整每篇摘要长度，而非固定 200 字。
> 2. **分批处理**：将文章分组，分别调用 DeepSeek 生成子日报，再合并为最终日报。
> 3. **缓存机制**：对相同 URL 的文章摘要进行缓存，避免重复调用 API。
> 4. **更精细的错误处理**：区分网络超时、API 限流、认证失败等不同异常，采取不同重试策略。
> 5. **支持流式输出**：对于超长日报，可以先返回部分内容，再异步补充，提升用户体验。
>
> 这些改进将进一步提升系统的健壮性，为后续商业化 SaaS 服务打下基础。

更进一步，可以将这套能力封装成一个**SaaS 产品**，用户只需：

- 登录网页配置关注的 RSS 源
- 绑定飞书或企业微信
- 每天定时收到 AI 生成的个性化日报

这不仅是个人效率工具，也是一个完整的 AI 应用案例，具备商业化潜力。

---

<a id="sec11"></a>
## 十一、关键卡点与避坑指南

在搭建过程中，我遇到了几个最具代表性的问题，逐一攻克后系统才稳定运行。

#### 卡点一：手动执行脚本完美运行，但 OpenClaw 调用时卡死或失败

- **现象**：在终端执行 `python fetch_and_push.py` 能正常生成日报并发送飞书；但通过 OpenClaw 对话或飞书消息触发时，脚本无响应，最终 fallback 到联网搜索。
- **根本原因**：OpenClaw 调用技能时，工作目录、环境变量、网络超时、输出缓冲等与手动终端完全不同。
- **解决方案**：
  1. 脚本开头强制切换工作目录并加载 `.env`。
  2. 所有网络请求增加超时（`requests.get(url, timeout=30)`）。
  3. 每个 `print` 加上 `flush=True`。
  4. 创建包装脚本 `run.sh`，在 `SKILL.md` 中调用它。

#### 卡点二：FreshRSS 聚合源的筛选逻辑完全失效

- **现象**：设置了白名单关键词，但脚本跳过所有文章。
- **原因**：FreshRSS 输出的 RSS 条目中，`<source>` 字段为空。
- **解决方案**：放弃依赖 `source` 字段，改用文章链接的域名进行匹配。

#### 卡点三：飞书机器人无响应，日志显示 `ECONNREFUSED 127.0.0.1:7897`

- **根本原因**：WSL 环境中遗留了 HTTP_PROXY 代理环境变量，但代理服务未运行。
- **解决方案**：清除所有代理环境变量，编辑 `~/.bashrc` 删除相关行，重启 WSL。

> **教训**：代理环境变量是最大的“隐形杀手”，设置后务必确保代理服务运行，或在不使用时彻底清除。

---

<a id="sec12"></a>
## 十二、工程化实践与经验总结

### 工程化实践

1. **双模式设计**：统一聚合模式（适合大量源） + 直接抓取模式（简单可靠）。
2. **环境隔离**：Python 虚拟环境 + Docker 容器 + systemd 用户服务。
3. **错误处理与降级**：网络请求超时重试；DeepSeek 失败时降级为原始列表；飞书消息超长自动分批。
4. **测试体系**：独立功能测试（环境、模式、日报生成、Obsidian 写入）+ `pytest` 集成测试。
5. **可观测性**：脚本内大量 `print(..., flush=True)`；OpenClaw 日志；临时调试文件。
6. **配置管理**：所有敏感信息存放在 `.env`；模式开关通过环境变量控制，无需改代码。

### 经验教训

1. **永远假设自动化环境与手动环境不同**：工作目录、环境变量、PATH、代理、超时、输出缓冲——逐一显式控制。
2. **为所有网络请求设置超时**：看似简单的 RSS 抓取也可能永久挂起。
3. **不要在技能中依赖 `cd` 或复合 shell 命令**：使用包装脚本或绝对路径。
4. **大模型调用必须考虑降级**：保证核心功能可用。
5. **飞书长连接配置务必检查**：常见错误是选错“回调 URL”而非“长连接”。
6. **Obsidian 路径必须用 WSL 格式**：不能直接用 `D:\`。
7. **代理环境变量是最大元凶**：设置后务必确保服务运行。
8. **测试脚本要独立，不依赖主脚本**：避免因主脚本错误导致无法测试。
9. **技能分享需包含 `SKILL.md` + 脚本 + `.env.example` + 依赖清单**。

### 最终成果

- 一个完全可用的 OpenClaw 技能 `rss-summary`，支持两种模式、智能筛选、AI 日报生成、飞书推送、Obsidian 归档。
- 完整的技能定义文件（`SKILL.md`）、主脚本（`fetch_and_push.py`）、包装脚本（`run.sh`）、环境变量模板（`.env.example`）、依赖清单（`requirements.txt`）。
- 详细的项目文档（`README.md`）和 MIT 许可证。
- 深刻的工程经验：环境一致性、超时处理、降级策略、API 集成、日志调试。

### 个人成长

- 从“不会写代码”到独立完成一个可工程化的 AI 应用。
- 掌握了大模型 API 调用、飞书机器人开发、Docker 基本操作、Python 调试技巧。
- 培养了“问题拆解 → 假设 → 验证 → 迭代”的工程思维。

> 这个项目的代码现在看起来很普通，但当时花了整整三天才跑通。回头看，正是那三天里的每一次报错、每一次重试，让我对 API 调用、环境配置、错误处理有了体感。没有什么知识是一蹴而就的，但每迈过一个坎，下一个坎就会矮一些。

---

> **附：[GitHub 仓库：Ashley-Linn/rss-summary-openclaw](https://github.com/Ashley-Linn/rss-summary-openclaw)**

*本文基于真实技术实践，所有命令已在 Windows 11 + WSL2 + Ubuntu 22.04 环境中验证通过。*