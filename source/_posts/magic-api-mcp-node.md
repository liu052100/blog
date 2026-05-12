---
title: magic-api-mcp-node：让 Claude Code 直接操作 magic-api 接口
date: 2026-05-12 20:00:00
tags:
  - MCP
  - magic-api
  - Claude Code
  - AI
categories:
  - 工具
---

> **项目地址**：[liu052100/magic-api-mcp-node](https://github.com/liu052100/magic-api-mcp-node) | **版本**：v1.x | **协议**：MIT
>
> 把公司 magic-api 当成"代码托管"，让 Claude 帮你 CRUD 接口、调用部署接口、调试脚本、查数据源。

---

## 它解决什么问题

公司用 magic-api 2.1.1 做低代码 HTTP API 开发。日常痛点：

- 想批量改接口？得一个个点开 Web UI
- 想抽公共逻辑？得手写一坨 Magic Script
- 想 Code Review？没有像样的 diff 工具
- 想让 AI 帮忙？Claude 不知道 MagicScript 语法，瞎编 API

`magic-api-mcp-node` 通过 **MCP + Skill** 两件套解决：

| 组件 | 路径 | 作用 |
|------|------|------|
| **MCP 服务** | `dist/` | 让 Claude 能**操作** magic-api（25 个工具：CRUD / 调用 / 调试 / 查询 / Admin） |
| **Claude Skill** | `skill/` | 让 Claude 能**理解** magic-api（5 份领域知识：db / 内置模块 / 语法 / 配置 / 扩展） |

两个都装，威力翻倍。

---

## 25 个 MCP 工具一览

| 分类 | 工具 | 用途 |
|------|------|------|
| **资源** | `list_resource_tree` / `get_api_detail` | 接口树 + 单接口详情 |
| 资源 | `create_api` / `update_api` / `delete_api` / `move_api` / `copy_api` | 接口 CRUD + 移动复制 |
| 资源 | `create_group` / `delete_group` | 分组 CRUD |
| **函数** | `get_function_detail` / `create_function` / `update_function` | 函数文件 CRUD |
| **任务** | `get_task_detail` / `create_task` / `update_task` | 定时任务 CRUD |
| **调用** | `call_api` | 调用已部署接口（防 SSRF + 敏感 header 黑名单） |
| **调试** | `execute_script` / `test_run_api` / `get_debug_logs` | 临时跑脚本 / 调试模式 / WebSocket 日志 |
| **元数据** | `list_datasources` / `list_functions` / `list_components` / `list_tasks` / `list_api_history` | 数据源 / 函数 / 组件 / 任务 / 版本历史 |
| **Admin** | `reload_magic_api` | 触发后端重建路由表 + Quartz 调度器（改 task cron/enabled 必调） |

> 💡 **函数 / 任务 CRUD 关键约束**：必须先建对应类型分组（`group_type='function'` 或 `'task'`），不能直接放根目录 `"0"`（会报 `code=1001 找不到分组信息`）。

---

## 5 分钟接入指南

### 一、前置条件

| 软件 | 检查命令 |
|------|----------|
| Node.js ≥ 18.18 | `node -v` |
| Git | `git -v` |
| Claude Code CLI | `claude --version` |
| 能访问 magic-api 后台 | 浏览器打开 `http://<MAGIC_API_HOST>:<PORT>/magic/web/index.html` 能登录 |

> 💡 **网络要求**：你电脑必须能访问公司 magic-api 服务器（公司内网 / VPN）。浏览器能打开 = MCP 也能用。

### 二、拉取项目并编译

```cmd
cd D:\workCode
git clone https://github.com/liu052100/magic-api-mcp-node.git
cd magic-api-mcp-node
npm install
npm run build
```

完成后会生成 `dist/index.js`——这就是 MCP 入口。**记住完整路径**，例如：`D:/workCode/magic-api-mcp-node/dist/index.js`。

### 三、找到公司 magic-api 配置

向运维要 5 项（或自己看公司项目的 `application.yml`）：

```yaml
server:
  port: <PORT>                    # ← 端口
magic-api:
  web: /magic/web                 # ← UI 路径（默认）
  prefix:                         # ← 接口前缀（通常空）
  security:
    username: <YOUR_USERNAME>     # ← 登录账号
    password: <YOUR_PASSWORD>     # ← 登录密码
```

5 项分别是：服务器地址、WEB 路径、接口前缀、账号、密码。

> ⚠️ **本文中所有 `<MAGIC_API_HOST>` / `<PORT>` / `<YOUR_USERNAME>` / `<YOUR_PASSWORD>` 都需要替换成内网真实值。请勿在公开渠道（包括博客、IM 群、截图）传播这些值。**

### 四、注册到 Claude Code

#### 🪟 Windows 用户（**必须用 cmd.exe，不要 Git Bash！**）

> ⚠️ **两个大坑**：
> 1. Git Bash 会把 `/magic/web` 错误转成 `D:/dev/Git/magic/web`（MSYS 路径转换）
> 2. **不要用 `^` 续行符**！claude CLI 在 Windows 会把 `^` 当成命令名

按 `Win + R` → 输入 `cmd` → 回车，**整段一行**复制粘贴执行：

```cmd
claude mcp add magic-api -s user -e MAGIC_API_BASE_URL=http://<MAGIC_API_HOST>:<PORT> -e MAGIC_API_WEB_PATH=/magic/web -e MAGIC_API_USERNAME=<YOUR_USERNAME> -e MAGIC_API_PASSWORD=<YOUR_PASSWORD> -- node "D:/workCode/magic-api-mcp-node/dist/index.js"
```

把 5 处占位符替换好（服务器地址、端口、账号、密码、Node 项目路径），**整行不换行**回车。

#### 🍎 macOS / 🐧 Linux 用户

```bash
claude mcp add magic-api -s user \
  -e MAGIC_API_BASE_URL=http://<MAGIC_API_HOST>:<PORT> \
  -e MAGIC_API_WEB_PATH=/magic/web \
  -e MAGIC_API_USERNAME=<YOUR_USERNAME> \
  -e MAGIC_API_PASSWORD=<YOUR_PASSWORD> \
  -- node /Users/yourname/magic-api-mcp-node/dist/index.js
```

#### 备选方案：手动编辑配置

文件位置：`~/.claude.json`（macOS/Linux）或 `C:/Users/<用户名>/.claude.json`（Windows）

```json
{
  "mcpServers": {
    "magic-api": {
      "type": "stdio",
      "command": "node",
      "args": ["D:/workCode/magic-api-mcp-node/dist/index.js"],
      "env": {
        "MAGIC_API_BASE_URL": "http://<MAGIC_API_HOST>:<PORT>",
        "MAGIC_API_WEB_PATH": "/magic/web",
        "MAGIC_API_USERNAME": "<YOUR_USERNAME>",
        "MAGIC_API_PASSWORD": "<YOUR_PASSWORD>"
      }
    }
  }
}
```

### 五、重启 Claude Code 验证

#### 1. **完全退出** Claude Code

不是关 tab，是**整个进程退出**：

- **Windows**：任务栏右键 Claude Code → 退出；或 `taskkill /F /IM Claude.exe`
- **macOS**：⌘Q
- **Linux**：关闭所有窗口

⚠️ **不完全退出会导致刚改的配置被内存里旧配置覆盖！**

#### 2. 重启后输入 `/mcp`

期望看到：

```
✅ magic-api    connected    25 tools
```

#### 3. 试用功能

直接对 Claude 说：**"列出 magic-api 的资源树"** —— 应该会列出公司所有 API 分组。

### 六、安装 Skill 知识包（强烈推荐）

让 Claude 在写脚本时知道 magic-api 的各种模块、语法、配置——**写出来的代码符合规范、不瞎编 API**。

#### 🪟 Windows

```cmd
cd D:\workCode\magic-api-mcp-node
xcopy /E /I /Y skill "%USERPROFILE%\.claude\skills\magic-api"
```

#### 🍎 macOS / 🐧 Linux

```bash
cd ~/magic-api-mcp-node
mkdir -p ~/.claude/skills
cp -r skill ~/.claude/skills/magic-api
```

完全退出再重启 Claude Code，输入 `/magic-api` 应看到 skill 加载成功。

Skill 内容速览：

```
~/.claude/skills/magic-api/
├── SKILL.md                       入口（Claude 自动读）
└── references/
    ├── db-module.md               SQL / 事务 / 缓存 / LINQ / mybatis 标签
    ├── builtin-modules.md         request / response / http / log / env / magic
    ├── script-syntax.md           MagicScript 语法（类型 / lambda / 异步 / 解构）
    ├── configuration.md           application.yml 全部配置
    └── extensions.md              自定义模块 / 拦截器 / 方言 / 鉴权 / 插件
```

> 💡 跟 MCP 不一样的是，**Skill 不需要任何配置**——文件拷贝过去、重启 Claude Code 就生效。

---

## 配置参数完整列表

| 环境变量 | 必填 | 默认 | 说明 |
|----------|------|------|------|
| `MAGIC_API_BASE_URL` | ✅ | `http://localhost:9999` | magic-api 服务器地址（**含**协议+主机+端口，**不含**路径） |
| `MAGIC_API_WEB_PATH` | ⬜ | `/magic/web` | 对应 `magic-api.web` 配置 |
| `MAGIC_API_API_PREFIX` | ⬜ | _(空)_ | 对应 `magic-api.prefix`，仅 `call_api` 调部署接口时用 |
| `MAGIC_API_USERNAME` | ⬜ | _(空)_ | 安全关闭时可不填 |
| `MAGIC_API_PASSWORD` | ⬜ | _(空)_ | 同上 |
| `MAGIC_API_REQUEST_TIMEOUT` | ⬜ | `30` | 单次 HTTP 超时（秒） |
| `MAGIC_API_TREE_CACHE_TTL` | ⬜ | `10` | 资源树缓存秒数（多人写时调小） |
| `MAGIC_API_DEBUG_LOG_BUFFER` | ⬜ | `1000` | 调试日志环形 buffer 大小 |

---

## 在 Claude Code 中能做什么

| 你说 | 实际触发的工具 |
|------|---------------|
| "列出所有 API 接口" | `list_resource_tree` |
| "看下 BOM删除 这个接口的脚本" | `list_resource_tree` 找 id → `get_api_detail` |
| "把 BOM保存_更新 优化一下并加注释" | `get_api_detail` → 重写 → `update_api` |
| "调用一下 /getInspectionGoods 看返回" | `call_api` |
| "新建一个查询订单的接口" | `create_group`（可选）→ `create_api` |
| "把领料/补料/退料公共逻辑抽成函数" | `create_group(group_type='function')` → `create_function` → 老接口 × N `update_api` 改成 import 调用 |
| "看下 saveMoveBill 函数的代码" | `list_functions` 找 id → `get_function_detail` |
| "改下 queryStock 函数加个参数" | `get_function_detail` → 重写 → `update_function` |
| "新建一个每30分钟跑一次的同步任务" | `create_group(group_type='task')`（可选）→ `create_task` |
| "把 供应商同步 任务的 cron 改成每小时" | `get_task_detail` → 改 cron → `update_task` → `reload_magic_api` |
| "执行这段 Magic Script: db.select(...)" | `execute_script`（建临时接口 → 跑 → 自动删除） |
| "看接口 abc 的历史版本" | `list_api_history` |
| "本地都有哪些数据源？" | `list_datasources` |

---

## 常见问题排查（按出现频率排序）

### ❌ `/mcp` 显示 `failed` 或 `Failed to reconnect`

#### 原因 1：项目级配置覆盖了用户级（最常见！）

```cmd
type C:\Users\你的用户名\.claude.json | findstr "magic-api"
```

如果看到 `projects.<某路径>.mcpServers.magic-api`，进入对应项目目录执行：

```cmd
claude mcp remove magic-api -s project
```

#### 原因 2：Git Bash 路径转换

`.claude.json` 里 `MAGIC_API_WEB_PATH` 变成 `D:/dev/Git/magic/web` 这种 Windows 路径——是 Git Bash MSYS 路径转换搞的。**重新用 cmd.exe** 执行 `claude mcp add`。

#### 原因 3：cmd 续行符 `^` 被 claude CLI 当成命令名

`.claude.json` 里 `command` 字段是 `^`、`args` 是 `[]`、`env` 是 `{}`——**cmd 续行符没生效**。解决：

```cmd
claude mcp remove magic-api -s user
```

然后**整行不换行**重新执行 `claude mcp add` 命令。

#### 原因 4：Claude Code 重启时回写了旧配置

改 `.claude.json` 时 Claude Code 必须**完全退出**。

### ❌ `ECONNREFUSED` / 网络不通

- 浏览器能打开 `http://<MAGIC_API_HOST>:<PORT>/magic/web/index.html` 吗？
- 在公司外是否连了 VPN？
- 服务器域名/IP 写错了吗？

### ❌ `login failed` / `401`

- 账号密码对吗？复制粘贴避免敲错
- 浏览器能用相同账号登录吗？
- 密码有特殊字符（`@ # $`）吗？环境变量里直接写就行，cmd 不需要转义

### ❌ 启动慢/超时

公司服务器 magic-api 响应慢——调大 `MAGIC_API_REQUEST_TIMEOUT=60`（秒）。

---

## 安全说明

- 🔐 **`.env` 不要 commit**：`.gitignore` 已默认排除
- 🔐 **登录 token 仅内存**：不落盘、不打日志
- 🔐 **危险操作前必须确认**：`update_api` / `delete_api` / `delete_group` 都是写操作，建议先 `get_api_detail` 看一眼再决定
- 🔐 **`call_api` 拒绝绝对 URL**：只能调 magic-api 自己的接口，防 SSRF
- 🔐 **敏感 header 自动剥离**：`call_api` 出站请求自动过滤 `cookie / authorization / host / content-length / connection / magic-token`
- 🔐 **内网信息不外传**：本文中所有 `<MAGIC_API_HOST>` / `<PORT>` / `<YOUR_USERNAME>` / `<YOUR_PASSWORD>` 都是占位符。真实值请向团队内部渠道索取，**勿在博客、聊天群、截图、PR 描述中泄露**。

---

## 兼容性

| 项 | 支持 |
|----|------|
| magic-api | **2.1.1**（公司服务器实测验证）；2.0.x / 2.2.x 大概率兼容 |
| Node.js | ≥ 18.18（依赖原生 fetch） |
| 操作系统 | Windows / macOS / Linux |
| Claude 客户端 | Claude Code CLI / Claude Desktop |

---

## 反馈

仓库：[liu052100/magic-api-mcp-node](https://github.com/liu052100/magic-api-mcp-node)

任何问题先看本文 **"常见问题排查"** 章节，搞不定提 Issue 或团队内部沟通。
