---
title: Codex CLI 在 Windows 上的审批与沙箱：一份被新版本推翻的踩坑记录
date: 2026-09-23 10:30:00
tags:
  - Codex
  - AI
  - Windows
  - 沙箱
categories:
  - 工具
---

> **测试环境**：Windows 11，Codex CLI 0.148.0（2026-08）与 0.156.0（2026-09-23）对照测试。

8 月我想让 Codex 别再每跑一条命令就弹窗问我，结果折腾出一个很丧气的结论：**Windows 上 Codex 的沙箱根本起不来，想不弹窗就只能把沙箱整个关掉**。

一个月后 Codex 自动升到了 0.156，我把当时的每一条结论重测了一遍——大半已经不成立了。

这篇把两个版本放在一起对照。比结论更有用的是**验证方法**：Codex 的配置有好几处"看着生效、其实没生效"的地方，只看配置文件和 `codex doctor` 会被骗。

<!-- more -->

---

## 先分清两个旋钮

| 配置项 | 管什么 | 可选值（0.156） |
|---|---|---|
| `approval_policy` | 什么时候停下来问你 | `untrusted` / `on-failure` / `on-request` / `granular` / `never` |
| `sandbox_mode` | 命令能碰到哪些东西 | `read-only`（默认）/ `workspace-write` / `danger-full-access` |
| `windows.sandbox` | Windows 上沙箱的实现方式 | `elevated` / `unelevated` / `mxc` |

几个容易混的点：

- **两个旋钮互相独立**。"每条命令都弹窗"的根源通常是默认的 `read-only` 沙箱——只读沙箱里几乎什么都干不成，只好不停找你要权限。
- **`never` 的意思是"从不询问"，不是"自动批准"**。命令在沙箱允许的范围内直接跑，超出范围就直接失败，把错误交还给模型。
- **联网是第三个独立开关**。`workspace-write` 下默认不联网，要在 `[sandbox_workspace_write]` 里单独开 `network_access = true`。
- **`trust_level` 与审批无关**。`[projects."路径"] trust_level = "trusted"` 管的是信不信任该目录下的 `AGENTS.md` 等项目配置，配了也不会少弹一次窗。
- 命令行 `codex -a` 只列出 `on-request` 和 `never` 两个值，其余几个只能写在配置文件里；`codex exec` 干脆不接受 `-a` 参数。

---

## 0.148 时踩的坑（历史记录）

### 坑 1：沙箱启动失败，两层原因叠在一起

只要开沙箱（`read-only`、`workspace-write` 都算），执行任何 shell 命令都失败：

1. **`CreateProcessAsUserW failed: 5`（拒绝访问）**。沙箱会用"受限令牌"启动 PowerShell，而我机器上的 PowerShell 7 是**微软商店版**，装在 `C:\Program Files\WindowsApps\` 下，这个目录的权限不认受限令牌。
2. **`orchestrator_helper_launch_failed: setup helper: program not found`**。PATH 指向的 Codex 安装目录只有 `bin\`，缺了本该与它同级的 `codex-resources\`，沙箱辅助程序找不到。

### 坑 2："能用，只是烦"其实是假象

默认的 `on-request` 下每条命令都弹窗，但**你点"同意"，等于让这条命令跳出沙箱执行**——恰好绕过了坏掉的沙箱，所以一切看起来正常。

一改成 `never`，命令被强制留在沙箱里，立刻全线失败。

### 坑 3：`never` 会直接拒绝 MCP 工具

报错原文：

```
MCP tool call requires approval, but approval policy is never
```

对 shell 命令，`never` 是"在沙箱里直接跑"；对 MCP 调用，却是"拿不到批准 → 拒绝"。只有在 `danger-full-access` 下 MCP 才恢复正常。

### 坑 4：网上教程里的 `--full-auto` 已经没了

0.148 里它已被移除，`--help` 里看不到，但大量教程还在教。

### 当时的无奈之选

```toml
approval_policy = "never"
sandbox_mode = "danger-full-access"
```

能用，但等于让 Codex 在整台机器上随意读写。

---

## 0.156 复测：大半结论被推翻

2026-09-23 在 0.156.0 上，用同一台机器、同一份配置重测。每一项都让 Codex **真实执行命令或调用工具**，并去文件系统上核对结果：

| 测试项 | 0.148 | 0.156 |
|---|---|---|
| `read-only` 下执行 `echo` | ❌ 失败 | ✅ 成功 |
| `read-only` 下写文件 | — | ✅ 被拒（PermissionDenied），文件没有生成 |
| `workspace-write` 写工作目录内 | ❌ 失败 | ✅ 成功 |
| `workspace-write` 写工作目录外 | — | ✅ 被拒，文件没有生成 |
| `never` + `workspace-write` 调 MCP 工具 | ❌ 被拒 | ✅ 成功 |
| `--full-auto` | 已移除 | 仍然不存在 |
| 配置里有无法识别的字段 | 完全静默 | 启动时提示 `Codex is ignoring N unrecognized configuration settings` |

几点说明：

- PowerShell 仍然是商店版（这次是 7.6.6），但沙箱已经能正常用它启动了。
- 我没有逐个版本去定位是哪一版修好的，只能确定 **0.148 不行、0.156 可以**。
- **沙箱"能跑命令"不等于"在拦东西"**，所以表里专门测了越权写入——两次都被拦下，文件系统上也确认没有文件生成。

所以在 0.156 上，不必再为了不弹窗而关掉沙箱：

```toml
approval_policy = "never"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
writable_roots = ["D:\\your\\projects"]
network_access = true   # 需要联网才开；本次复测没有专门测联网
```

另外，0.156 的 `codex exec` 多了一个 `--approve-for-me` 参数，帮助里的说明是"把审批请求交给自动审查，使用 `workspace-write` 沙箱"。这个我还没实测。

---

## 比结论更值钱的：验证方法

结论会过期，验证方法不会。这次复测十几分钟就做完了，靠的是下面几招。

### 1. 探针必须真跑 shell、真调 MCP

0.148 那次，我第一轮验证只让 Codex 回一句话：启动横幅正常、严格校验通过、`codex doctor` 通过，三重绿灯——实际上它连一条命令都执行不了。

**纯聊天任务碰不到沙箱，验证不出任何东西。**

### 2. 还要测"该拦的有没有拦"

只测"能跑"，分不清是沙箱在正常工作，还是沙箱被悄悄绕过了。至少补一条越权写入，再去文件系统上看文件到底有没有生成。

### 3. 用 `--json` 看事件，别听模型怎么说

`codex exec --json` 会逐条输出事件：`command_execution` 带真实的命令和 `exit_code`，`mcp_tool_call` 带 `server`、`tool`、`status`。这比模型的自述可靠得多。

```bash
codex exec --json --skip-git-repo-check -s workspace-write \
  "用 shell 执行 echo probe-ok，然后只回复输出" < /dev/null
```

### 4. 用非法值"诈"出合法取值

文档跟不上版本时，故意填一个非法值，报错信息会把合法值全列出来。它在配置解析阶段就失败，不会真的发请求：

```bash
codex exec -c 'windows.sandbox="__bogus__"' "x" < /dev/null
# unknown variant `__bogus__`, expected one of `elevated`, `unelevated`, `mxc`

codex exec -c 'approval_policy="__bogus__"' "x" < /dev/null
# expected one of `untrusted`, `on-failure`, `on-request`, `granular`, `never`
```

`mxc` 就是这次诈出来的新值，0.148 时只有前两个。

### 5. `--strict-config` 查拼错和过时的字段

TOML 里写错的字段默认会被忽略。`codex exec --strict-config` 会在第一个无法识别的字段处报错，并给出行号。我的配置一查就是 8 个：

- 1 个已经废弃的字段 `disable_response_storage`；
- 7 个 `type = "stdio"`——从 Claude Code 的 MCP 配置照抄过来的。**Codex 的 `[mcp_servers.xxx]` 没有 `type` 字段**，写了 `command` 就是 stdio。

注意两点：`--strict-config` 一次只报第一个，要逐个修掉再跑；`codex doctor` 不接受这个参数。

---

## 总结

- **0.156 上 Windows 沙箱已经能用**，`never` + `workspace-write` 可以做到"不弹窗、有边界"，不必再 `danger-full-access`。
- `never` 是"不问"，不是"全批"；联网是另一个开关；`trust_level` 与审批无关。
- 验证要**真执行、测越权、看事件**；用非法值诈枚举，用 `--strict-config` 查字段。
- 这类工具迭代很快，**任何"XX 不能用"的结论都要带上版本号**，升级后重测一遍。

---

## 参考

- [openai/codex](https://github.com/openai/codex)
- Codex CLI 自带帮助：`codex --help`、`codex exec --help`
