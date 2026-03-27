---
title: Claude Code 的 95 个技能，我按用途整理了一份完整目录
date: 2026-03-27 12:00:00
tags:
  - Claude Code
  - AI
  - 效率
  - Skills
categories:
  - 工具
---

用 Claude Code 写了一段时间代码之后，技能（Skills）越装越多，多到自己都记不清有哪些、什么时候该用哪个。上周调试一个前端样式问题，折腾了半天才想起来有个 `/clarify` 专门优化界面文案，还有个 `/harden` 做边界情况加固——早用上能省不少事。

所以花了点时间把目前环境里的所有技能过了一遍，按实际用途分了类。不是官方文档，是个人使用角度的整理，带一些使用场景的判断。

<!-- more -->

---

## 先说结论

95 个技能，分成 9 大类。日常开发高频用到的大概 15-20 个，剩下的属于"知道它在就行，需要的时候能想起来"。

最值得花时间熟悉的是前两类：**Superpowers 核心工作流**和 **CCG 多模型协作**。前者决定你怎么组织工作，后者决定你能调动多少外部算力。其他的都是锦上添花。

---

## 一、Superpowers — 核心工作流

这组技能不生产代码，它们管的是**你怎么写代码**。听起来很虚，但用过就知道差别有多大。

| 技能 | 干什么 | 什么时候用 |
|------|--------|-----------|
| `brainstorming` | 需求探索，先想清楚再动手 | 拿到一个模糊需求，不确定该怎么做 |
| `writing-plans` | 拆解实施计划 | 任务超过 3 步，需要分步骤执行 |
| `executing-plans` | 按计划分步实施 | 有了计划，开始逐步执行 |
| `test-driven-development` | TDD：先测试后实现 | 写功能或修 bug 之前 |
| `systematic-debugging` | 系统化调试 | 问题复杂，靠直觉找不到根因 |
| `dispatching-parallel-agents` | 多 Agent 并行 | 有两个以上互不依赖的子任务 |
| `verification-before-completion` | 完成前强制验证 | 准备说"搞定了"之前 |
| `finishing-a-development-branch` | 分支收尾 | 代码写完，准备合并或提 PR |
| `requesting-code-review` | 发起代码审查 | 提交后想让 AI 再检查一遍 |
| `receiving-code-review` | 处理审查反馈 | 收到 review 意见，需要逐条回应 |
| `using-git-worktrees` | 隔离工作区 | 功能开发不想污染当前分支 |
| `subagent-driven-development` | 子 Agent 驱动开发 | 当前会话内并行实施 |
| `writing-skills` | 写/改技能文件 | 想自定义一个新技能 |

这里面 `brainstorming` 和 `verification-before-completion` 是我觉得最容易被忽略但价值最高的两个。前者防止你一头扎进去写了半天发现方向不对，后者防止你以为搞定了其实还有边界没覆盖。

`systematic-debugging` 也值得单独说一下。以前遇到 bug 的习惯是加日志、猜原因、改一下试试——这种方式对简单问题够用，但遇到多层调用链的问题就容易绕圈子。这个技能强制你走"收集证据 → 形成假设 → 验证假设"的流程，虽然慢一点，但不走弯路。

---

## 二、CCG — 多模型协作

CCG 是目前环境里最复杂也最强力的一套技能体系。核心思路是：**Claude 做编排，Codex 管后端，Gemini 管前端，三个模型交叉验证**。

### 日常开发

| 技能 | 说明 |
|------|------|
| `/ccg:workflow` | 完整六阶段流程：研究 → 构思 → 计划 → 执行 → 优化 → 评审 |
| `/ccg:feat` | 智能功能开发，自动判断前端还是后端 |
| `/ccg:frontend` | 前端专项，Gemini 主导 |
| `/ccg:backend` | 后端专项，Codex 主导 |
| `/ccg:plan` | 双模型协作出计划 |
| `/ccg:execute` | 按计划执行，多模型审计 |
| `/ccg:codex-exec` | Codex 全权执行，省 Claude token |

说实话，`/ccg:workflow` 完整跑一遍比较重，适合大功能。日常小改动用 `/ccg:feat` 就够了，它会自己判断该走前端还是后端路线。

### 质量保障

| 技能 | 说明 |
|------|------|
| `/ccg:review` | 无参数直接审查 git diff，双模型交叉验证 |
| `/ccg:test` | 自动路由生成测试 |
| `/ccg:debug` | 双模型诊断定位 |
| `/ccg:optimize` | 双模型性能优化 |
| `/ccg:analyze` | 技术分析，Codex + Gemini 并行出结论 |

`/ccg:review` 是我用得最多的一个。写完代码直接跑，不用给参数，它自己读 git diff 然后两个模型分别审查。经常能发现自己漏掉的边界情况。

### OpenSpec 规格驱动

这套流程适合需求比较正式的场景——先把规格写清楚，再按规格实施：

| 技能 | 阶段 |
|------|------|
| `/ccg:spec-init` | 初始化 OPSX 环境 |
| `/ccg:spec-research` | 需求 → 约束集 |
| `/ccg:spec-plan` | 约束 → 零决策可执行计划 |
| `/ccg:spec-impl` | 按规格执行 + 归档 |
| `/ccg:spec-review` | 双模型交叉审查 |

### Agent Teams 并行

这是 CCG 里最新也最激进的能力——派多个 Agent 同时写代码：

| 技能 | 说明 |
|------|------|
| `/ccg:team-research` | 并行探索代码库 |
| `/ccg:team-plan` | 产出并行实施计划 |
| `/ccg:team-exec` | 派 Builder 并行写代码 |
| `/ccg:team-review` | 双模型交叉审查产出 |

实际体验下来，适合模块边界清晰、互不依赖的任务。如果几个文件之间有复杂依赖，并行写容易冲突。

### Git 工具

| 技能 | 说明 |
|------|------|
| `/ccg:commit` | 分析改动自动生成 Conventional Commit |
| `/ccg:rollback` | 交互式回滚，支持 reset/revert |
| `/ccg:clean-branches` | 清理已合并分支，默认 dry-run |
| `/ccg:worktree` | Worktree 管理 |
| `/ccg:init` | 初始化项目 CLAUDE.md |
| `/ccg:context` | 上下文管理、决策日志 |
| `/ccg:enhance` | Prompt 增强，模糊需求 → 结构化描述 |

### 质量门禁

这几个不带 `ccg:` 前缀，直接用名字调用：

| 技能 | 触发条件 |
|------|---------|
| `verify-security` | 改了认证、授权、加密相关代码 |
| `verify-quality` | 单次改动超过 30 行 |
| `verify-change` | 需要检查改动影响和文档同步 |
| `verify-module` | 新建了一个模块 |
| `gen-docs` | 新模块需要文档骨架 |

---

## 三、UI/UX 设计技能

这一组数量最多，有 19 个，覆盖了从创建到打磨的整个 UI 生命周期。

### 创建

- **`frontend-design`** — 生成高质量前端界面，刻意避免"一看就是 AI 生成的"那种通用感
- **`document-skills:web-artifacts-builder`** — 复杂多组件构件，React + Tailwind + shadcn/ui
- **`teach-impeccable`** — 一次性设置，告诉 AI 你项目的设计上下文，后续所有设计技能都会参考

### 调优（加强 / 削弱）

做出来的东西太平？用 **`bolder`** 加强冲击力。太花哨？用 **`quieter`** 降低强度。颜色太单调？**`colorize`**。字体层级不对？**`typeset`**。布局间距不舒服？**`arrange`**。

这几个技能的思路很像修图：不是从零开始，而是在已有基础上调整。一般的使用路径是：

```
frontend-design（创建）→ critique（评估）→ bolder/quieter/colorize/typeset/arrange（调优）→ polish（最终打磨）
```

### 体验增强

| 技能 | 说明 |
|------|------|
| `clarify` | 改善界面文案，让用户看得懂 |
| `onboard` | 引导流程和空状态设计 |
| `delight` | 加点让人开心的小细节 |
| `animate` | 有意义的动画和微交互 |
| `harden` | 加固：错误处理、i18n、溢出、边界 |
| `adapt` | 跨屏幕和平台适配 |
| `optimize` | 性能优化：加载、渲染、包体积 |
| `overdrive` | 突破常规的技术实现 |

### 设计系统

| 技能 | 说明 |
|------|------|
| `normalize` | 规范化到设计系统 |
| `extract` | 提取可复用组件和设计令牌 |
| `distill` | 去除不必要的复杂性 |
| `audit` | 全面审计：无障碍、性能、主题、响应式 |
| `critique` | UX 视角评估，给出可操作反馈 |

### 主题和品牌

- **`document-skills:theme-factory`** — 10 种预设主题 + 自定义生成
- **`document-skills:brand-guidelines`** — Anthropic 品牌色彩和排版

---

## 四、文档和文件处理

这组很直观，按文件格式来：

| 技能 | 格式 | 能做什么 |
|------|------|---------|
| `document-skills:pdf` | PDF | 读取、合并、拆分、水印、加密、OCR |
| `document-skills:docx` | Word | 创建、编辑、格式化、目录、页眉页脚 |
| `document-skills:pptx` | PPT | 幻灯片创建编辑、模板、批注 |
| `document-skills:xlsx` | Excel/CSV | 表格创建编辑、图表、数据清洗 |
| `document-skills:canvas-design` | PNG/PDF | 视觉设计作品 |
| `document-skills:algorithmic-art` | p5.js | 生成艺术、粒子系统、流场 |
| `document-skills:slack-gif-creator` | GIF | Slack 用的动画 GIF |

实际使用频率最高的是 PDF 和 Excel。PDF 合并拆分、Excel 数据清洗这些，以前得开专门的工具，现在一句话搞定。

---

## 五、协作和沟通

| 技能 | 说明 |
|------|------|
| `document-skills:doc-coauthoring` | 结构化文档协作：提案、技术规格、决策文档 |
| `document-skills:internal-comms` | 内部沟通：状态报告、FAQ、事件报告 |

---

## 六、开发辅助工具

| 技能 | 说明 | 使用频率 |
|------|------|---------|
| `simplify` | 审查代码的复用性和效率 | 中 |
| `sql-optimization` | SQL 调优、索引策略、执行计划 | 高（后端项目） |
| `ruoyi-dev` | 按 RuoYi 标准生成 CRUD 代码 | 高（本项目专用） |
| `claude-api` | 构建 Claude API 应用 | 低 |
| `loop` | 定时循环执行命令 | 低 |
| `update-config` | 配置 Claude Code 设置 | 偶尔 |
| `document-skills:mcp-builder` | 创建 MCP Server | 偶尔 |
| `document-skills:skill-creator` | 创建和测试技能 | 偶尔 |
| `document-skills:webapp-testing` | Playwright 测试 Web 应用 | 中 |
| `dev-browser:dev-browser` | 浏览器自动化 | 中 |

`sql-optimization` 对后端开发者来说几乎是必备的。丢一条慢查询进去，它会分析执行计划、建议索引、给出优化后的写法。比自己盯着 EXPLAIN 输出琢磨快很多。

`ruoyi-dev` 是针对当前项目的，按 RuoYi-Vue 的约定生成 Entity、Mapper、Service、Controller 全套，省了大量重复劳动。

---

## 七、Context Mode

这组解决一个很具体的问题：**大量输出会撑爆上下文窗口**。

| 技能 | 说明 |
|------|------|
| `context-mode:context-mode` | 用沙箱处理大输出，替代 Bash/cat |
| `context-mode:doctor` | 诊断运行环境 |
| `context-mode:stats` | 查看上下文节省统计 |
| `context-mode:upgrade` | 更新到最新版 |

装了之后基本不用管，它会自动拦截可能产生大输出的命令，在沙箱里处理完只返回摘要。对长会话的稳定性提升很明显。

---

## 八、GudaSpec

另一套规格驱动开发框架，和 CCG 的 OpenSpec 类似但独立：

| 技能 | 说明 |
|------|------|
| `gudaspec:init` | 初始化环境 |
| `gudaspec:research` | 需求转约束集 |
| `gudaspec:plan` | 生成实施计划 |
| `gudaspec:implementation` | 按规格执行 |

---

## 九、运维

| 技能 | 说明 |
|------|------|
| `myserver` | 服务器信息、博客运维、SSH、部署流程 |

这个是自定义的，记录了服务器上所有服务的地址、账号、管理命令。写博客、管服务器的时候直接调用，不用每次翻笔记。

---

## 实际使用建议

**不要试图记住所有技能。** 95 个全记住既不现实也没必要。

我的做法是分三层：

1. **肌肉记忆层**（5-8 个）：每天都用，不用想就能打出来。对我来说是 `brainstorming`、`ccg:feat`、`ccg:review`、`ccg:commit`、`sql-optimization`、`ruoyi-dev`。

2. **知道在哪层**（10-15 个）：不是每天用，但遇到特定场景知道有这么个东西。比如 `systematic-debugging`、`harden`、`clarify`、`verify-security`。

3. **目录查阅层**（其余）：存在这篇文章里，需要的时候翻一下。

技能本身在不断更新。隔一段时间值得重新过一遍，看看有没有新加的、或者原来用不好现在变好用了的。

---

工具再多，最终还是服务于写出好代码这件事。技能是手段不是目的，别让工具链的复杂度超过了问题本身的复杂度。
