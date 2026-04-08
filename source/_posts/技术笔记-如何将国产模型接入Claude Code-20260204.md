---
title: 如何将国产模型接入Claude Code
category: 技术笔记
tags:
  - Claude Code
  - CC-Switch
  - 国产大模型
  - API接口
  
slug: claude-code
permalink: /2026/02/04/claude-code/
---

# 一、安装Claude Code

## （一）前置环境Node.js安装

- 说明：claude基于node运行，因此必须先装node

- 官方下载地址：[Node.js — Download Node.js®](https://nodejs.org/en/download)（根据需要下载相应版本）

- **重要提示**：确保你的 **Node.js 版本 ≥ 18.0.0**。若版本过低，安装 `claude-code` 时会报错。

- 手动添加环境变量

- 验证，如利用cmd输入以下指令，如若显示版本号即安装成功：

  ```
  node --version
  npm --version
  ```

## （二）claude code下载安装

目前Anthropic在中国禁止、且最近大批量封号现象发生，无法通过直接下载软件方式安装，经测试，比较顺畅的方式是直接在终端利用npm安装。打开cmd，安装过程如下：

```
# 全局安装指令
npm install -g @anthropic-ai/claude-code

# 验证
claude --version
```

如若显示版本号，说明安装成功，**如果显示claude指令无法识别，需手动配置环境变量**

## （三）关键一步（首次启动和认证）

**目前首次启动大概率会面临不可用情况（如下图所示），参考技术文档 [claude code 无法连接到 Anthropic 服务](https://docs.packyapi.com/docs/faq/CC.html#claude-code-%E6%97%A0%E6%B3%95%E8%BF%9E%E6%8E%A5%E5%88%B0-anthropic-%E6%9C%8D%E5%8A%A1)，可以通过修改用户配置文件 `.claude.json`，将 `hasCompletedOnboarding` 字段强制设为 `true`，从而跳过官方的初始引导流程：**

![首次登录显示不可用](/images/docs/首次登录显示不可用.png)

**方法一：命令行自动修改****

1. 按下键盘 `Win + R` 键，输入 `cmd` 后回车，打开命令行程序
2. 在命令行中运行以下命令后回车

```bash
powershell -Command "$f='%USERPROFILE%\.claude.json';$j=Get-Content $f|ConvertFrom-Json;$j|Add-Member -NotePropertyName 'hasCompletedOnboarding' -NotePropertyValue $true -Force;$j|ConvertTo-Json|Set-Content $f"
```

3.重启Claude终端，出现如下界面，表示已经成功绕过Anthropic原始提示不可用界面：

![绕过成功界面](/images/docs/绕过成功界面.png)

**方法二：手动修改文件**

- **文件位置**：`C:\Users\用户名\.claude.json`（Windows）或 `~/.claude.json`（macOS/Linux）。

- 操作步骤：

  1. 打开该文件（用记事本或代码编辑器）

  2. 在文件末尾的 `}` 之前，添加或确保存在以下内容（注意逗号）：

     ```
         "hasCompletedOnboarding": true
     ```

  3. 保存文件并关闭

  4. 重新启动 Claude Cde 终端



# 二、利用CC-Switch配置国产模型

## （一）下载、安装

- 下载路径：https://github.com/farion1231/cc-switch/releases
- 安装：直接安装即可

## （二）获取密钥

- **流程都一样：打开网址→注册账号→登录→创建api_key→购买额度**

  - **途径一：**使用哪家厂商模型，就去哪家注册、创建api_-_key
    - deepseek、智谱AI、Moonshot、Minimax等
  - **途径二：**直接去api聚合平台注册，获取api_key，可统一管理模型	
    - 很多，自由选择：Openrouter、Poe、OneApI、0011.ai、2233.ai、接口API、Lino API、PackyAPI、New API、灵芽API等

- **强调：要想流畅使用模型，该花钱的钱得花**！

- **注意事项：**

  - 1、**API 密钥安全**：API Key 相当于你的账户密码，**切勿泄露**或提交到公开的代码仓库中
  - 2、api接口的url网址，openai和anthropic格式有区别，目前大模型厂商提供接口有区分	

  |          | openai兼容格式                       | Anthropic格式                          |
  | :------: | ------------------------------------ | -------------------------------------- |
  | deepseek | https://api.deepseek.com/v1          | https://api.deepseek.com/anthropic     |
  |  智谱AI  | https://open.bigmodel.cn/api/paas/v4 | https://open.bigmodel.cn/api/anthropic |
  | Moonshot | https://api.moonshot.cn/v1           | https://api.moonshot.cn/anthropic      |
  | Minimax  | https://api.minimaxi.com/v1          | https://api.minimaxi.com/anthropic     |

## （三）配置

- 流程：打开CC-Switch软件 → 选中claude分组 → 添加供应商 → 填写 API Key 和 URL → 测试并启用，具体可参考技术文档[Claude Code配置 | PackyAPI 使用文档](https://docs.packyapi.com/docs/ccswitch/2-claude.html)

- 注意：具体模型供应商，按需要选择


#  三、正式使用

以上过程执行完毕后，重新打开claude终端，即可正式接入国产模型界面如下：

![绕过成功界面](/images/docs/绕过成功界面.png)

直接回车，即可进入正式交互界面：

![交互界面](/images/docs/交互界面.png)

在交互界面中，输入问题，结果如图：

![交互演示](/images/docs/交互演示.png)

