---
title: Playwright MCP：让 Claude Code 直接操控浏览器
date: 2026-04-09 20:00:00
tags:
  - MCP
  - Playwright
  - Claude Code
  - AI
  - 浏览器自动化
  - 测试
categories:
  - 工具
---

> **官方包**：[@playwright/mcp](https://www.npmjs.com/package/@playwright/mcp) | **维护者**：Microsoft | **协议**：Apache-2.0

Playwright MCP 是微软官方发布的 MCP Server，让 Claude Code、Claude Desktop、Cursor、VS Code 等 AI 工具能够**直接控制真实浏览器**——导航页面、点击按钮、填写表单、截图、读取网络请求，全部通过自然语言驱动。

与截图识别方案不同，Playwright MCP 基于**无障碍树（Accessibility Tree）**进行交互，Token 消耗更低、响应更快、操作更精准。

<!-- more -->

---

## 工作原理

整体架构非常简洁：

```
Claude Code（AI 推理层）
    │
    │  MCP 协议（JSON-RPC）
    ↓
Playwright MCP Server（本地进程）
    │
    │  CDP / Browser Protocol
    ↓
真实浏览器（Chromium / Firefox / WebKit）
```

**关键流程：**

1. Claude Code 通过 MCP 协议向 Playwright Server 发送指令（如 `browser_navigate`）
2. Server 调用 Playwright API 操控浏览器执行动作
3. Server 返回**无障碍树快照**（而非截图）给 Claude Code
4. Claude 基于结构化文本理解页面状态，决定下一步操作

无障碍树快照长这样：

```
- heading "Dashboard" [level=1]
- navigation "Main"
  - link "Settings" [href="/settings"]
  - link "Profile" [href="/profile"]
- button "Save Changes" [disabled]
- textbox "Search" [focused]
```

这比截图方案高效得多——纯文本、低 Token、结构化，AI 能精准定位元素。

---

## 安装与配置

### 前置条件

- Node.js 18+
- Claude Code 已安装（`npm install -g @anthropic-ai/claude-code`）

### 一行注册

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

这条命令会将 Playwright MCP Server 注册到 Claude Code 的 MCP 配置中。下次启动 `claude` 时自动可用。

### 安装浏览器二进制（首次需要）

```bash
npx playwright install
```

> Linux/Docker 环境还需要 `npx playwright install-deps` 安装系统依赖。

### 作用域控制

```bash
# 个人全局（所有项目可用）
claude mcp add --scope user playwright npx @playwright/mcp@latest

# 项目级（写入 .mcp.json，可提交到版本控制）
claude mcp add --scope project playwright npx @playwright/mcp@latest
```

团队协作建议锁定版本号，避免 beta 版本带来不一致：

```bash
claude mcp add playwright npx @playwright/mcp@0.0.28
```

### 其他客户端配置

如果你使用 Claude Desktop 或 Cursor，在对应配置文件中添加：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

| 客户端 | 配置文件路径 |
|--------|-------------|
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Cursor | `Cursor Settings → MCP → Add new MCP Server` |
| VS Code | `code --add-mcp '{"name":"playwright","command":"npx","args":["@playwright/mcp@latest"]}'` |

---

## 可用工具一览

注册成功后，Claude Code 会获得以下浏览器操控能力：

### 导航与页面

| 工具 | 功能 |
|------|------|
| `browser_navigate` | 导航到指定 URL |
| `browser_navigate_back` | 浏览器后退 |
| `browser_snapshot` | 获取无障碍树快照 |
| `browser_take_screenshot` | 页面截图 |
| `browser_wait_for` | 等待特定条件 |
| `browser_resize` | 调整浏览器窗口大小 |
| `browser_close` | 关闭浏览器 |

### 交互操作

| 工具 | 功能 |
|------|------|
| `browser_click` | 点击元素 |
| `browser_type` | 输入文本 |
| `browser_hover` | 鼠标悬停 |
| `browser_drag` | 拖拽元素 |
| `browser_press_key` | 按键操作 |
| `browser_select_option` | 下拉选择 |
| `browser_file_upload` | 文件上传 |
| `browser_handle_dialog` | 处理弹窗（alert/confirm） |
| `browser_fill_form` | 表单填充 |

### 调试与信息

| 工具 | 功能 |
|------|------|
| `browser_console_messages` | 读取控制台日志 |
| `browser_network_requests` | 查看网络请求 |
| `browser_evaluate` | 执行 JavaScript |
| `browser_generate_playwright_test` | 生成 Playwright 测试代码 |

### 标签页管理

| 工具 | 功能 |
|------|------|
| `browser_tabs` | 列出所有标签页 |
| `browser_tab_new` | 新建标签页 |
| `browser_tab_select` | 切换标签页 |
| `browser_tab_close` | 关闭标签页 |

---

## 实战场景

### 场景一：自然语言驱动 Web 测试

在 Claude Code 中直接说：

```
用 Playwright MCP 打开浏览器，访问 https://example.com，
点击"Login"链接，在用户名输入框填入 admin，密码填入 test123，
点击提交按钮，然后截图给我看结果。
```

Claude 会自动编排工具调用链：

```
browser_navigate → browser_snapshot → browser_click →
browser_type × 2 → browser_click → browser_take_screenshot
```

全程无需手写任何 Playwright 代码。

### 场景二：前端 Bug 复现与调试

```
打开浏览器访问 http://localhost:3000/dashboard，
查看控制台有没有报错，检查网络请求中是否有失败的 API 调用，
把结果汇总给我。
```

Claude 会调用 `browser_console_messages` 和 `browser_network_requests`，直接汇总错误信息，省去手动打开 DevTools 的步骤。

### 场景三：生成 E2E 测试代码

```
访问 http://localhost:3000，完成一次完整的用户注册流程，
然后用 browser_generate_playwright_test 生成对应的测试代码。
```

Playwright MCP 可以基于你实际操作的流程，自动生成标准的 Playwright 测试脚本。

### 场景四：多页面数据采集

```
打开浏览器，依次访问以下 3 个产品页面，提取每个页面的标题、价格和描述：
- https://shop.example.com/product/1
- https://shop.example.com/product/2
- https://shop.example.com/product/3
```

Claude 会循环调用导航和快照工具，从无障碍树中精准提取结构化数据。

### 场景五：响应式设计验证

```
打开 http://localhost:3000，分别用以下尺寸截图：
- 1920x1080（桌面）
- 768x1024（平板）
- 375x667（手机）
对比三个截图，告诉我有没有布局问题。
```

通过 `browser_resize` + `browser_take_screenshot` 组合，快速验证响应式布局。

---

## 无障碍树 vs 截图：为什么选无障碍树？

| 维度 | 无障碍树（Playwright MCP） | 截图方案 |
|------|--------------------------|---------|
| Token 消耗 | 低（纯文本） | 高（图片编码） |
| 响应速度 | 快 | 慢（需要 OCR/视觉模型） |
| 操作精度 | 高（精确元素引用） | 中（坐标可能偏移） |
| 动态内容 | 支持（实时快照） | 需要频繁截图 |
| 调试能力 | 强（含 ARIA 属性） | 弱（纯视觉） |

简单来说，无障碍树方案就像给 AI 一份**结构化的页面蓝图**，而截图方案相当于给 AI 一张**照片让它猜**。

---

## 常见问题

### Q: 首次使用时 Claude 没调用 Playwright MCP？

首次交互建议明确提到 "用 Playwright MCP"，否则 Claude 可能会尝试用 Bash 直接运行 Playwright CLI。建立上下文后，后续对话会自动使用 MCP 工具。

### Q: 浏览器窗口没有弹出？

默认模式下 Playwright 使用 headless 浏览器。如果需要看到浏览器窗口，可以在启动参数中添加 `--headed`：

```bash
claude mcp add playwright npx @playwright/mcp@latest --headed
```

### Q: CI 环境如何使用？

CI 环境通常没有显示器，确保使用 headless 模式（默认行为），并安装系统依赖：

```bash
npx playwright install --with-deps chromium
```

### Q: 和直接写 Playwright 脚本相比有什么优势？

| 场景 | 推荐方式 |
|------|---------|
| 快速验证、探索性测试、原型验证 | Playwright MCP + 自然语言 |
| 稳定的回归测试套件 | 标准 Playwright 测试脚本 |
| Bug 复现和调试 | Playwright MCP |
| CI/CD 集成 | 标准 Playwright 测试脚本 |

两者互补，MCP 适合**探索和验证**，脚本适合**固化和回归**。

---

## 总结

Playwright MCP 把浏览器变成了 Claude Code 的"眼睛和手"：

- **零代码操控浏览器**——自然语言描述，AI 自动编排
- **无障碍树驱动**——比截图方案更快、更准、更省 Token
- **一行命令安装**——`claude mcp add playwright npx @playwright/mcp@latest`
- **完整的工具集**——导航、点击、输入、截图、调试、生成测试代码
- **多场景适用**——Web 测试、前端调试、数据采集、响应式验证

如果你还在手动打开浏览器 → 打开 DevTools → 复制粘贴错误信息给 AI，不妨试试让 Claude Code 直接接管浏览器。

---

## 参考

- [Playwright MCP 官方文档](https://playwright.dev/docs/getting-started-mcp)
- [Builder.io - How to Use Playwright MCP Server with Claude Code](https://www.builder.io/blog/playwright-mcp-server-claude-code)
- [Simon Willison - Using Playwright MCP with Claude Code](https://til.simonwillison.net/claude-code/playwright-mcp-claude-code)
