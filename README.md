# ⚡ pi-config

## 📦 概述

本仓库集中管理 pi Agent 的个性化配置、扩展插件、MCP 服务以及开发行为准则，旨在为 AI 编码助手提供**一致的行为准则、高效的工具链和规范的工作流**。

## 🗂️ 目录结构

```text
.
├── AGENTS.md               # 全局开发准则（交互规范、编码原则、工具路由、Agent 协作）
├── settings.json           # pi 核心设置（主题、依赖包、重试策略、默认工具集，带详细注释）
├── mcp.json                # MCP 外部服务配置（Chrome 调试、代码检索、全网搜索）
├── models.json.example     # 模型渠道配置模板（支持 OpenAI / Claude / 中转站，带详细注释）
├── pi-cc-extensions.json   # 生产力扩展套件配置
├── extensions/             # 本地扩展配置目录
│   └── pi-rtk-optimizer/   # RTK 命令输出优化与压缩配置
├── .gitignore              # 敏感信息与缓存忽略规则
└── README.md
```

## 🧩 扩展包说明 (Packages)

`settings.json` 中的 `packages` 字段声明了智能体启用的扩展包，涵盖代码审查、极速检索、多 Agent 协作与状态展示：

| 包名 | 类别 | 核心功能说明 | 包含工具 / 触发模式 |
| :--- | :--- | :--- | :--- |
| `pi-mcp-adapter` | 协议适配 | 接入 Model Context Protocol (MCP)，连接外部服务 | MCP 工具自动分发 |
| `pi-web-access` | 网络检索 | 提供网页检索与内容深度提取能力 | `web_search`、`fetch_content` |
| `pi-subagents` | 多 Agent 协作 | 支持多智能体分工委派，并内置专家顾问团辩论仲裁 | `subagent`、`council-mode` |
| `@dietrichgebert/ponytail` | 编码准则 | 强制推行极简编码哲学，防止过度工程与无用抽象 | `ponytail`、`ponytail-review` |
| `@ff-labs/pi-fff` | 极速检索 | 基于 Rust 的高性能文件模糊查找与内容搜索 | `fffind`、`ffgrep` |
| `pi-rtk-optimizer` | Token 优化 | 重写终端命令并压缩长输出，避免上下文快速溢出 | 命令过滤器自动生效 |
| `@narumitw/pi-statusline` | 界面增强 | 终端底部状态栏，实时显示路径、耗时、Token 与上下文 | TUI 自绘状态条 |
| `@narumitw/pi-plan-mode` | 交互控制 | 任务分步规划模式，支持向用户进行结构化提问确认 | `plan_mode_question`、`/plan` |
| `pi-cc-extensions` | 生产力套件 | 辅助增强配置与上下文检查 | 生产力辅助工具 |

## 🔌 MCP 服务说明 (mcp.json)

通过 Model Context Protocol (MCP) 接入外部工具，让 AI 拥有突破本地文件限制的能力：

| 服务名称 | 连接方式 | 什么时候触发（大白话） | 赋予的核心能力 |
| :--- | :--- | :--- | :--- |
| `chrome-devtools` | 本地进程 (`npx`) | **做前端或测试交互时**：AI 自己拉起无头浏览器看渲染效果、查报错 | 网页截图验证、执行页面 JS |
| `searchcode` | 远程服务 (HTTP) | **参考外部开源项目时**：不用把别人的大仓库 clone 到本地，直接搜代码 | 检索 GitHub 开源库、提取函数声明 |
| `tavily-remote-mcp` | 远程服务 (HTTP) | **查最新技术/报错时**：突破大模型知识库截止时间，查实时全网资料 | 深度全网调研、网页深度抓取 |

## 📜 全局行为准则（AGENTS.md）

`AGENTS.md` 是注入给 AI 的最高行为准则，确保 AI 表现稳定、可靠且受控：

- **交互规范**：
  - 强制全中文交互。
  - **改动确认制**：修改任何代码前，必须先提供具体方案与改动范围，等待用户明确确认后再落盘执行。
  - 沟通代码优先，日常解释精简（≤ 3 行）。
- **工程准则（Ponytail Core）**：
  - **YAGNI & 原生优先**：不写未明确要求的抽象、工厂与单次接口；能用标准库解决的严禁引入新依赖。
  - **根因修复**：修 Bug 前必须 grep 所有调用链（callers），在公共路由做最小防御，禁止打表面补丁。
  - **零测试 & 编译必过**：严禁生成冗余测试文件，带类型的语言必须确保类型与编译检查通过。
- **工具路由规则**：
  - 文件路径检索优先用 `fffind`；代码内容检索优先用 `ffgrep`。
  - 单文件多处改动必须在单次 `edit` 中合并完成。
- **Subagent 协作**：
  - 复杂大型任务必须主动提议使用 `subagent`，经用户确认后方可启动。
  - 多 Agent 必须启用 `worktree` 隔离，保持单写入者机制。

## ⚙️ 核心配置详解

### settings.json
| 配置项 | 说明 | 当前值 |
| :--- | :--- | :--- |
| `theme` | 终端界面主题 | `"dark"` |
| `defaultTools` | 默认开放给 AI 的基础工具集 | `["read", "bash", "edit", "write"]` |
| `packages` | 启动时自动加载的扩展包清单 | 见上方「扩展包说明」 |
| `retry` | API 报错或超时后的自动重试策略 | 最多重试 8 次，基础延迟 10 秒 |
| `defaultProvider` | 默认模型提供商 | `"localhost-gemini"` |
| `defaultModel` | 默认启动模型 | `"gemini-3.8-flash-high"` |

### models.json
用于定义自定义 AI 渠道与模型映射。真实文件已通过 `.gitignore` 保护，请参考 `models.json.example` 进行配置。

### pi-cc-extensions.json
用于微调终端交互与代码 Diff 渲染体验：

| 配置项 | 默认值 | 作用说明 |
| :--- | :--- | :--- |
| `diffViewMode` | `"auto"` | 代码差异展示模式（`auto` / `unified` 单栏 / `split` 分栏） |
| `diffIndicatorMode` | `"bars"` | 差异状态指示器样式（`bars` 彩色条 / `symbols` 符号） |
| `diffSplitMinWidth` | `120` | 触发双栏分栏展示的最小终端宽度 |
| `editDiffCollapsedLines` | `24` | 修改代码时折叠未改动区域的行数阈值 |
| `writeDiffCollapsedLines` | `0` | 新建文件时折叠代码的行数阈值（`0` 表示全量展开） |
| `useSummaryTitlesAsThinkingTitle` | `true` | 是否使用自动生成的摘要作为思考过程的折叠标题 |
| `enableContextCommand` | `true` | 是否开启 `/context` 上下文分析命令 |
| `enableSubagentAutocomplete` | `true` | 是否启用子智能体指令的自动补全 |
| `scrollStepLines` | `3` | 终端滚动步长（行数） |


## 🚀 快速开始

### 1. 同步配置到本地
```bash
git clone https://github.com/nafsifan/pi-config.git ~/.pi/agent
cd ~/.pi/agent
```

### 2. 安装全部扩展依赖
```bash
pi install \
  pi-mcp-adapter \
  pi-web-access \
  pi-subagents \
  @dietrichgebert/ponytail \
  @narumitw/pi-statusline \
  @narumitw/pi-plan-mode \
  @ff-labs/pi-fff \
  pi-cc-extensions \
  pi-rtk-optimizer
```

### 3. 配置模型凭据
```bash
cp models.json.example models.json
# 编辑 models.json 填入你自己的 API Key 与接口地址
```

### 4. 启动使用
```bash
pi
```

---

## 🪟 Windows 用户适配贴士

- **配置路径**：Windows 对应的目录为 `C:\Users\你的用户名\.pi\agent`。
- **默认终端工具**：
  - 若系统已安装 **Git for Windows**，可直接沿用默认的 `"bash"`。
  - 若使用系统自带终端，请在 `settings.json` 中将 `"defaultTools"` 里的 `"bash"` 改为 `"powershell"`。

