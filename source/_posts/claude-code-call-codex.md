---
title: 在 Claude Code 里调用 Codex：从 CCG、MCP 到 mcp-server 被删之后
date: 2026-09-23 10:40:00
tags:
  - Claude Code
  - Codex
  - MCP
  - AI
categories:
  - 工具
---

> **测试环境**：Windows 11，Claude Code + Codex CLI 0.156.0，2026-09-23。

让 Claude Code 干活时顺手问一下 Codex，是我用得最多的"第二意见"：交叉审查方案、让另一个模型独立排查同一个 bug、把体力活分出去省 token。

这件事我前后换了三种接法。最近一次是被迫换的——今天发现 Claude Code 里 codex 那一项连不上了，一查才知道：**Codex 0.154.0 把 `codex mcp-server` 这个子命令删了**。自动更新之后，原来的接法就这么悄无声息地断了。

<!-- more -->

---

## 第一阶段：CCG 工作流

3 月写过一篇[《Claude Code MCP 服务与 CCG 工作流总览》](/2026/03/24/mcp-overview/)，当时用的是 [ccg-workflow](https://github.com/fengshao1227/ccg-workflow)：`npx ccg-workflow` 安装，提供一整套 `/ccg:*` 命令，Claude 负责编排，Codex 和 Gemini 分别处理后端和前端。

后来我把它整个卸掉了，换成了更轻的接法。**那篇文章里 CCG 的部分已经过时**，以本文为准。

---

## 第二阶段：把 Codex 接成 MCP 服务

Codex CLI 以前自带 `codex mcp-server` 子命令，可以作为 MCP 服务运行。接进 Claude Code 只要一条命令：

```bash
claude mcp add codex -s user -- codex mcp-server
```

它提供两个工具：

| 工具 | 作用 |
|---|---|
| `codex` | 开一个新会话。参数有 `prompt`、`cwd`、`model`、`sandbox`、`approval-policy`、`config` 等，返回会话 ID（`threadId`） |
| `codex-reply` | 带着 `threadId` 继续同一个会话，上下文延续 |

当时用下来的几个要点（0.148 实测）：

- 它读 `~/.codex/config.toml`，审批和沙箱设置与命令行一致（相关的坑见[上一篇](/2026/09/23/codex-windows-approval-sandbox/)）。
- 改 `config.toml` 下次调用就生效；但**新加的 MCP 服务要重启 Claude Code 会话**才会出现在工具列表里。
- 一次调用 30~60 秒起步，思考档位越高越慢，要给足超时。
- 别问 Codex 它是什么模型——它的系统提示词里写死了一句自我介绍，它复述的是那句话，不是实际配置的型号。以 `config.toml` 和会话记录为准。

---

## 第三阶段：0.154 删掉了 mcp-server

### 症状

- Claude Code 里 `claude mcp list` 显示 codex `✘ Failed to connect`，错误是 `CONNECTION_CLOSED`。
- 手动运行 `codex mcp-server`，报错：

```
Error: stdin is not a terminal
```

第二条很有迷惑性：报错说的是终端，跟 MCP 八竿子打不着。原因是 `mcp-server` 已经不是子命令了，Codex 把它当成了**一句提示词**，去启动交互界面，而交互界面需要终端。

### 确认

Codex 自动更新会把历次版本保留在 `~/.codex/packages/standalone/releases/` 下。逐个版本看 `--help`：

| 版本 | 有没有 `mcp-server` |
|---|---|
| 0.148.0 / 0.151.0 / 0.153.2 / 0.153.4 | ✅ 有 |
| 0.154.0 / 0.155.0 / 0.155.1 / 0.156.0 | ❌ 没有 |

对应上游的 PR [#42993《Remove the deprecated `codex mcp-server` command》](https://github.com/openai/codex/pull/42993)，2026-09-05 合并。0.154.0 的发布说明里，只在 Chores（杂项）下写了一行：

> The deprecated `codex mcp-server` entry point is no longer available.

没有给替代方案。

---

## 现在怎么接

### 方案 A（推荐）：让 Claude Code 直接调 `codex exec`

Claude Code 本来就能跑命令，直接用 Codex 的非交互模式即可。这是 Codex 正式支持的入口，跟着升级走，不用担心哪天又被删。

**开新会话：**

```bash
codex exec --json --skip-git-repo-check -s workspace-write \
  -o answer.md "审查 src/ 下的改动，列出风险点" < /dev/null
```

- `--json`：输出事件流。第一条 `thread.started` 里有 `thread_id`，后面有 `command_execution`、`mcp_tool_call`、`agent_message` 等事件；
- `-o`：把最终回答写进文件，Claude Code 直接读文件，不用从事件流里抠；
- `-s`：按任务收紧或放开沙箱；
- `-C <目录>`：指定工作目录。

**续聊**（对应原来的 `codex-reply`）：

```bash
codex exec resume <thread_id> "针对第 2 个风险点给出修改方案" < /dev/null
# 或者接最近一次会话
codex exec resume --last "..." < /dev/null
```

实测：第一次让它记住一个暗号，第二次用 `resume` 追问，答对了，上下文确实延续。

关于结尾的 `< /dev/null`：0.148 在 Claude Code 的 Bash 工具里不加它，会卡在 `Reading additional input from stdin...` 一直等；0.156 实测不加也能正常结束。加上没有副作用，建议保留。

**和 MCP 方式对比：**

| | MCP（已失效） | `codex exec` |
|---|---|---|
| 调用方式 | 工具调用 | Bash 命令 |
| 续聊 | `codex-reply` + threadId | `exec resume` + thread_id |
| 过程可见性 | 只返回最终文本 | `--json` 能看到每条命令和退出码 |
| 跟随升级 | 入口已删除 | 正式支持的非交互入口 |

### 方案 B（临时）：指向本机残留的旧版

旧版本还留在 `releases` 目录里。把 MCP 指向 0.153.4 的程序，握手和工具列表都正常（`codex`、`codex-reply` 两个工具都在）：

```bash
claude mcp remove codex -s user
claude mcp add codex -s user -- "C:\Users\<你的用户名>\.codex\packages\standalone\releases\0.153.4-x86_64-pc-windows-msvc\bin\codex.exe" mcp-server
```

代价是永远停在 0.153.4，之后的新模型和修复都用不上；哪天旧版本目录被清理，又会再断一次。只适合过渡。（这个方案我只验证到握手和工具列表，没有做完整的任务调用。）

---

## 教训

- **"deprecated"（已弃用）之后就是删除**，而删除可能只在发布说明的杂项里占一行。
- **自动更新 + 你依赖的入口 = 无声断链**。它不会打断你干活，只是工具列表里少了一项，很容易被忽略。升级后跑一次 `claude mcp list`，看看有没有 `✘`。
- **报错信息不一定指向真因**。`stdin is not a terminal` 看着像终端问题，实际是子命令没了。先用 `codex --help` 确认子命令还在不在。

---

## 参考

- [openai/codex PR #42993：Remove the deprecated `codex mcp-server` command](https://github.com/openai/codex/pull/42993)
- [openai/codex Release rust-v0.154.0](https://github.com/openai/codex/releases/tag/rust-v0.154.0)
- 旧文：[Claude Code MCP 服务与 CCG 工作流总览](/2026/03/24/mcp-overview/)
