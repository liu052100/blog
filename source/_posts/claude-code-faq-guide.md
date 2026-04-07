---
title: Claude Code 终极版 FAQ 指南：Windows 安装与常见问题全解
date: 2026-04-07 18:00:00
tags:
  - Claude Code
  - AI
  - Windows
  - FAQ
categories:
  - 工具
---

> **原文作者**：哈雷彗星 (Haleclipse) | **来源**：[LINUX DO - 《Claude Code 终极版 FAQ 指南》](https://linux.do/t/topic/803265) | **原始发布日期**：2026-03-11
>
> 本文基于原帖内容整理转载，已保留原始引用。原帖持续更新中，建议同时关注原帖获取最新信息。

Claude Code 的更新发版速度很快，官方文档的内容存在滞后性。这篇 FAQ 实时跟进一些"隐晦"的问题，记录正确的解决方案，解决从基础到高级的疑惑。

---

## 安装部分

> 部分软件包存在个人偏好。如你明白其作用和意义，按需选择；如不明白，请悉数按序执行安装。目的是正确、方便、快乐地使用。
>
> —— Haleclipse

### 准备工作

从代理软件中获取 Proxy 端口，在 PowerShell 中执行：

```powershell
$env:HTTP_PROXY="http://127.0.0.1:Port"
$env:HTTPS_PROXY="http://127.0.0.1:Port"
```

以保证后续软件包的安装拉取正常进行。

### 1. Windows Terminal

从 [Microsoft Store](https://apps.microsoft.com/detail/9N0DX20HK701) 安装 Windows Terminal，安装后设定默认打开系统内置 PowerShell 并给予默认管理员权限。后续即可右键在所处文件夹内直接打开终端。

### 2. WinGet

```powershell
$progressPreference = 'silentlyContinue'
Install-PackageProvider -Name NuGet -Force | Out-Null
Install-Module -Name Microsoft.WinGet.Client -Force -Repository PSGallery | Out-Null
Write-Host "Using Repair-WinGetPackageManager cmdlet to bootstrap WinGet..."
Repair-WinGetPackageManager -AllUsers
```

### 3. PowerShell 7

```powershell
winget install Microsoft.PowerShell
```

安装后切换默认终端为 PowerShell 7，同样修改其默认配置为管理员启动。全部关闭后打开新终端窗口继续安装。

### 4. Notepad 4（可选，替代系统记事本）

```powershell
winget install zufuliu.notepad4
```

安装后可通过"系统集成设置"替换系统记事本和快捷打开方式。

### 5. Git for Windows

```powershell
winget install --id Git.Git -e --source winget
```

### 6. fnm（Node 版本管理）

```powershell
winget install Schniz.fnm
```

安装成功后关闭全部窗口，重新打开终端，继续安装 Node：

```powershell
fnm install lts/krypton
fnm use lts/krypton
```

#### 配置 fnm 环境

```powershell
notepad $profile
```

如果提示文件不存在，先新建：

```powershell
New-Item -Path $Profile -Type File -Force
```

在 Profile 中添加以下内容后保存：

```powershell
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression
```

关闭全部窗口，重新打开终端后：

```powershell
fnm use lts/krypton
npm config set registry https://registry.npmmirror.com
npm install -g @anthropic-ai/claude-code
```

### 美化相关（可选）

#### 等宽字体

推荐 [Maple Mono](https://github.com/subframe7536/maple-font)（支持中文的等宽字体），下载 `MapleMonoNormal-NF-CN.zip` 安装后，在 PowerShell 外观设置中选择 `Maple Mono Normal NF CN`。

#### Oh My Posh

```powershell
winget install JanDeDobbeleer.OhMyPosh --source winget --scope user --force
```

在 `$profile` 中新增：

```powershell
oh-my-posh init pwsh --eval | Invoke-Expression
```

主题样式参考：[Oh My Posh Themes](https://ohmyposh.dev/docs/themes)

---

## FAQ

### Q: Windows Plan Mode 切换问题

**A**：仅在 Node v24.2.0 / v22.17.0 以上版本，`Shift+Tab` 才能正确工作。在不支持的 Node 版本上会自动回退绑定到 `Alt+M`。v2.0.31 使用 bun 构建后全端统一为 `Shift+Tab`。

> 参考：[Windows 下 claude code 如何切换到 plan 模式？](https://linux.do/t/topic/794815/4) —— Haleclipse

### Q: Windows 图片粘贴方法

**A**：

- **macOS**：v1.0.61 增加了 `Cmd+V` 图片粘贴支持
- **Windows**：v1.0.93 增加了 `Alt+V` 图片粘贴支持（原先只能接收图片文件，不支持剪贴板图片数据）

> 参考：[win 下 claude code 怎么上传图片？](https://linux.do/t/topic/796754/28) —— Haleclipse

### Q: macOS iTerm2 换行正常，但 VSCode 内嵌终端多出一个反斜杠？

**A**：历史兼容性问题。`keybindings.json` 中的绑定内容 `"\\\\\\r\\n"` 会在换行后多输出一个 `\`。将其改为 `"\u001B\u000A"` 即可与 `Option+Enter` 行为一致。

### Q: Windows 触发错误 `Error: cannot open _claude_fs_right:`

**A**：暂时卸载 VSCode 的 VSIX 扩展程序，并关闭 IDE Auto Connect 功能。该扩展在编辑时将目标文件路径传递给 VSCode 进行协同，目前路径转换存在问题。

- v1.0.65 已修正链接 IDE 稳定性问题

### Q: Windows 下 MCP Server (Stdio) 全部无法使用？

**A**：将 MCP 配置改成 `command: "cmd"`，然后 `args: ["/c", "npx"]`。

原理是 Windows 下直接调用 `npx` 可能存在路径解析问题，通过 `cmd /c` 包装可以解决。

示例（SSE 类型直接添加）：

```bash
claude mcp add --transport sse context7 https://mcp.context7.com/sse
```

> 参考：[关于 Windows 原生运行 claude code 的问题](https://linux.do/t/topic/784087/4) —— Haleclipse

### Q: 红色 Offline 字样是做什么用的？遥测相关会不会用于检测封号？

**A**：`Offline` 只用于连接状态检测，与封号无关。遥测信息仅包含元数据。

> 参考：[windows claude code 显示 offline，但是不影响使用](https://linux.do/t/topic/807647/10) —— Haleclipse

### Q: 官网 / 中转站 / CCR 类自定义重定向 API 接入有何区别？

**A**：

#### 1. 官网直连

设定 `HTTP_PROXY` / `HTTPS_PROXY` 环境变量后正常登录即可。

#### 2. 中转站

使用其提供的 API 端点地址和 Key：

- `ANTHROPIC_BASE_URL`：API 端点（需是 Anthropic 形式接口）
- `ANTHROPIC_API_KEY`：标准 Anthropic 接口填此项
- `ANTHROPIC_AUTH_TOKEN`：非标准转换接口大概率需要此项

> **注意**：`ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 二选一，不可共存。

**接入示例**：

- **月之暗面 K2**（标准 Anthropic 形式）：
  ```json
  {
    "ANTHROPIC_BASE_URL": "https://api.moonshot.cn/anthropic",
    "ANTHROPIC_API_KEY": "你的Key"
  }
  ```

#### 3. 自定义兼容接口

同中转站配置，Key 处可按提供者要求处理（可置空）。

**重要提示**：

- 以上配置均只需在 `~/.claude/settings.json` 的 `env` 节点下编辑，**无须操作系统环境变量**
- `ANTHROPIC_API_KEY` 的优先级高于官网登录态，只要存在该 `env`，CC 将不会使用官网登录态
- v1.0.61 已添加 `--settings` 参数，可指定加载不同的 `settings.json` 配置文件
- 在设置中开启 `Verbose Output` 可恢复详情输出并显示上下文窗口实际值

---

## SubAgent 参考

原帖推荐了一个 SubAgent 集合网站供参考：[subagents.cc](http://subagents.cc/)

---

## 声明

本文内容整理自 LINUX DO 社区用户 **[哈雷彗星 (Haleclipse)](https://linux.do/u/haleclipse)** 的原创帖子，原帖地址：[https://linux.do/t/topic/803265](https://linux.do/t/topic/803265)。原帖标注为"施工中"且持续更新，本文仅为某一时间点的快照。如有出入，以原帖为准。感谢原作者的无私分享。
