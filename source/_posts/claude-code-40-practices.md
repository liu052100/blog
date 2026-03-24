---
title: 我好几个月都把 Claude Code 用错了：这 40 个实践彻底改变了我的工作流
date: 2026-03-24 12:00:00
tags:
  - Claude Code
  - AI
  - 效率
categories:
  - 工具
---

> 原文作者：Suryansh Tiwari | 来源：X / Twitter | 原始时间：2026-03-22

大多数人把 Claude Code 当成"自动补全"。而前 1% 的人，把它当成"操作系统"来用。

这就是"帮我写个函数"与"我睡觉时它替我交付完整功能"之间的差别。

**普通用户和高手之间的差距，不在技能，而在配置；而这个差距每周足以值回 4-6 个小时。**

---

## 设置与基础配置

### 01 — 改变一切的别名

每个认真使用 Claude Code 的人，第一件事通常都是设置 `cc` 别名。把下面这行加到你的 `~/.zshrc` 或 `~/.bashrc` 里：

```bash
alias cc='claude --dangerously-skip-permissions'
```

然后执行 `source ~/.zshrc`。以后你输入 `cc`，而不是 `claude`，从而跳过每次操作都要确认权限的提示。前提是：你必须清楚自己到底授权了什么——它之所以快，是因为它在"信任你"。

### 02 — 加一条实时状态栏

在 Claude Code 里运行 `/statusline`。它会生成一个 shell 脚本，在每次对话轮次结束后，把实时信息显示在终端底部，比如当前目录、Git 分支、上下文使用量等。它就像是给你的会话装了一个 HUD（抬头显示器）。

### 03 — 把上下文窗口扩展到 100 万 token

Sonnet 4.6 和 Opus 4.6 支持 100 万 token 上下文。你可以在会话中途用 `/model opus[1m]` 切换。作者建议先从 50 万开始，逐步往上加，在上下文压缩（compaction）开始明显吃掉输出质量之前，找到最适合你的甜蜜点。

### 04 — 只设一次输出风格，以后就别再管了

运行 `/config` 选择输出风格：Explanatory、Concise 或 Technical。你也可以在 `~/.claude/outputstyles/` 里自己创建完全自定义的风格。别再每次都手动修 Claude 的语气了，最好的做法是从一开始就把"说话方式"配置好。

### 05 — 用手机远程控制 Claude Code

运行 `claude remote-control`，然后从 claude.ai 或手机 App 连接。你可以在电脑上发起一个耗时很长的重构任务，然后去泡咖啡，坐在沙发上用手机看它的进度。

---

## 工作流与提速技巧

### 06 — ! 前缀：让 bash 命令输出直接进入上下文

输入 `!git status` 或 `!npm test`，命令输出会直接进入 Claude 的上下文里，它就能立刻基于这些结果继续行动，不需要你复制粘贴来回传递。对调试循环来说，这个功能非常强。

### 07 — Esc 停止，Esc+Esc 回滚

`Esc` 可以中途打断 Claude 当前的动作。`Esc+Esc`（或 `/rewind`）会打开一个菜单，让你恢复代码、恢复对话，或者两者一起恢复。这就像给那些你只有 40% 把握的想法准备了一个"撤销键"——放心大胆地用。

### 08 — Ctrl+S：把提示词草稿先存起来

当你正在写一段很长的提示词，中途突然又想插个简短问题时，按 `Ctrl+S` 就能把当前草稿暂存。你先把临时问题问完，拿到答案后，原来的草稿会自动恢复。

### 09 — Ctrl+B：把长任务丢到后台

当 Claude 正在跑测试套件或者构建流程时，按 `Ctrl+B`，它就会继续在后台工作，而你可以前台继续聊天、继续规划下一步。并行生产力，真正意义上的"它在干活，你在指挥"。

### 10 — /btw：侧边提问层

`/btw` 会弹出一个覆盖层，专门用来问小问题，比如"为什么你选这个方案？"之类。这样你的主线对话不会被这些顺手问题污染，但你的好奇心还是能得到即时回答。

### 11 — Ctrl+G：在它写代码之前先改它的计划

当 Claude 先给出一个计划时，按 `Ctrl+G` 会把这个计划直接打开到你的编辑器里。你可以修改步骤、重定策略，然后再让它执行。这样就能避免它在错误实现方向上白白跑 20 分钟。

### 12 — 语音输入往往比打字更容易给出高质量提示词

运行 `/voice` 启用按住说话。当你开口说一个需求时，你会自然带出更多背景、限制条件和细微语义；而打字时，你往往只会写最短版本。

---

## 上下文与提示词管理

### 13 — 不相关任务之间一定要 /clear

一个干净的新会话，几乎总比一个混乱地聊了 3 小时的旧会话更好。堆积起来的上下文，会在你没察觉时不断稀释你的指令。`/clear` 是阻止这种"慢性变笨"的最简单手段。

### 14 — 连续改了两次还不对，就重开

如果你已经纠正 Claude 两次，它还是偏了，不要再做第三次纠偏。直接 `/clear`，然后写一个全新的提示词，把你刚刚学到的东西一次性讲清楚。

### 15 — 别再"描述 bug"了，直接贴原始数据

不要用自己的话转述 bug。直接把错误日志、CI 输出、Slack 讨论串贴进去，然后说一句"修它"。

```bash
cat error.log | claude "explain this error and suggest a fix"
```

### 16 — 只要涉及架构，就用 Plan Mode

`Shift+Tab` 可进入 Plan Mode。只要是多文件修改、结构性变更、或者设计层面的任务，都值得先用它。

### 17 — 明确告诉 Claude 该看哪些文件

直接用 `@src/auth/middleware.ts` 这样的方式引用文件。Claude 能自动解析它。这样可以省掉它在整个代码库里四处搜索的 token 开销。

### 18 — 探索陌生代码时，模糊提示反而有用

像"你会怎么改进这个文件？"这样的宽泛问题，能给 Claude 留出空间，让它主动暴露那些你甚至还没想到要问的不一致和异味。

### 19 — 使用 /compact 时，要告诉它保留什么

如果你在做上下文压缩，别只敲一个 `/compact` 就结束。要加一句像"focus on the API changes"这样的保留指令。

### 20 — 在提示词里加上 "ultrathink"

在 Opus 4.6 上，你可以在提示词里直接加上 `ultrathink`。这样 Claude 会根据问题真实复杂度，动态分配推理预算。

---

## 自动化、工具与 MCP

### 21 — 给 Claude 一个"自检"的办法

在提示词里直接包含测试命令，例如："重构认证模块。跑测试套件。在宣布完成前修掉失败项。"这一句会直接闭合反馈回路。作者说，这能带来 2-3 倍的质量提升。

### 22 — 安装一个 LSP 插件

LSP 插件能在 Claude 编辑完文件后自动给出诊断信息。这样它能在你发现之前，就先把类型错误、语法问题之类的东西修掉。

```bash
/plugin install typescript-lsp@claude-plugins-official
```

### 23 — 能用 CLI，就尽量别用 MCP 服务器

CLI 工具通常比 MCP 服务器更省上下文。比如，让 Claude 用 `gh` 管 PR，用 `sentry-cli --help` 排线上问题。

### 24 — 最值得先装的 MCP 服务器

作者推荐四个高信噪比起点：Playwright（验证 UI）、PostgreSQL/MySQL（查 schema）、Slack（直接读缺陷讨论串）、Figma（设计转代码）。

### 25 — 用 /loop 做周期性后台检查

`/loop 5m check if deploy succeeded` 这类命令，会在会话保持开启时，按周期去检查某件事是否完成。

### 26 — 用 /permissions 把安全命令放进白名单

别再为 `npm run lint` 这种低风险命令一遍遍点确认了。把你信任的命令加入白名单，工作流就不会被琐碎确认反复打断。

---

## 精通 CLAUDE.md 与规则系统

### 27 — 先跑 /init，然后把结果砍掉一半

`/init` 会生成一个入门版的 CLAUDE.md。但别照单全收。每一行没必要的说明，都会变成额外的 token 负担，并悄悄挤占别处的注意力。

### 28 — 判断一条规则是否该留的试金石

对 CLAUDE.md 里的每一行都问自己一句："**如果没有这一条，Claude 会犯错吗？**" 如果答案是否定的，就删掉。你大概只有 150-200 条指令预算，再多，遵循度就会明显下降。

### 29 — 每犯一次错，就让规则文件变聪明一点

当 Claude 犯错时，直接说一句："Update CLAUDE.md so this doesn't happen again." 这样你的规则文件就不再是死文档，而会随着每次出错逐渐进化成一个活系统。

### 30 — 用 .claude/rules/ 写条件加载规则

给规则加上 paths frontmatter，只在相关范围内加载。比如，TypeScript 规则只对 `.ts` 文件生效，数据库规则只在 `/db` 目录里生效。

### 31 — 用 @imports 让 CLAUDE.md 保持轻量

与其把一大坨 Git 指南直接贴进 CLAUDE.md，不如写成 `@docs/git-instructions.md` 这种引用。Claude 只在需要时去读，基础上下文会更干净。

### 32 — Skills：按需加载知识，而不是长期塞满上下文

`.claude/skills/` 里的 Skills，本质上就是一种"按需加载的知识库"。它们能扩展 Claude 的能力，但不会把你的基线上下文撑爆。

### 33 — CLAUDE.md 用来"建议"，Hooks 用来"强制"

Claude 对 CLAUDE.md 的遵循率大概是 80% 左右。真正不能商量的东西——比如格式要求、安全限制、代码规范——应该交给 Hooks。因为 Hooks 每次都会执行，不靠"记得遵守"。

### 34 — 用 PostToolUse hook 自动格式化

把下面这段配置加到 `.claude/settings.json` 中，这样每次编辑结束后就会自动跑 Prettier：

```json
"hooks": {
  "PostToolUse": [{
    "matcher": "Edit|Write",
    "hooks": [{
      "type": "command",
      "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
    }]
  }]
}
```

### 35 — 用 PreToolUse hook 拦截破坏性命令

在 Bash 工具执行前先拦截它，阻止类似 `rm -rf` 或 `DROP TABLE` 的操作。这种 hook 是让你敢于给 Claude 更多自主权的"安全绳"。

---

## 高阶玩法：Worktree、Agents 与隔离环境

### 36 — --worktree：并行分支开发

`claude --worktree feature-auth` 会创建一个隔离的工作副本。你可以同时开 3 个并行会话，让它们分别处理不同功能，而互不踩文件。吞吐量会直接倍增。

### 37 — 用 subagents 保持主上下文清爽

例如你可以说："Use subagents to figure out the payment flow." 这样 Claude 会启动一个独立实例，自己去读文件、理解支付流程，然后只把摘要带回来。

### 38 — 为重复性任务创建自定义 subagents

用 `/agents` 把常用角色存到 `.claude/agents/` 中，比如快速搜索 agent、严格的 TypeScript reviewer、专门写文档的 agent。久而久之，你等于有了一支随叫随到的个人 AI 小队。

### 39 — Agent teams：让多个子代理并行协作

启用 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` 后，一个 team lead 可以把任务拆给 3-5 个 subagents 并行完成。对大规模研究、跨模块重构这种任务，它带来的不是"小优化"，而是速度层级的变化。

### 40 — /sandbox：让它在隔离环境里尽情试错

`/sandbox` 会通过 Seatbelt 或 bubblewrap 提供操作系统级隔离。你可以放心让 Claude 在一个实验性重构上"放开跑"，而不用担心真的破坏你的系统。最后只要看 diff，把你喜欢的部分合并回来就行。

---

**作者最后的结论很直接：大多数工程师在优化代码；而最快的那批人，在优化自己的 AI 工作流。这两件事带来的复利，并不是同一等级。**

> 40 条技巧，零废话，全是信号。把它存下来。那个以后用更少摩擦交付更快的你，会感谢现在的自己。
