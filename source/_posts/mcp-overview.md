---
title: Claude Code MCP 服务与 CCG 工作流总览
date: 2026-03-24 18:00:00
tags: [Claude Code, MCP, AI, CCG, 工具]
categories: 工具
---

记录目前 Claude Code 环境中配置的所有 MCP 服务及 CCG 多模型协作工作流，方便查阅。

<!-- more -->

## 远程服务 / AI 模型

| MCP | 工具 | 用途 |
|-----|------|------|
| **codex** | `mcp__codex__codex` | OpenAI Codex，代码生成、架构分析、PR Review |
| **gemini** | `mcp__gemini__gemini` | Google Gemini，前端分析、交叉验证 |
| **sequential-thinking** | `mcp__sequential-thinking__sequentialthinking` | 结构化多步推理，复杂问题拆解 |

## 开发工具

| MCP | 工具 | 用途 |
|-----|------|------|
| **github** | `mcp__github__*` | 仓库/Issue/PR/文件/Actions 全套 GitHub 操作 |
| **ssh-mpc-server** | `mcp__ssh-mpc-server__execute-command` 等 | SSH 连接服务器，执行命令、上传下载文件 |
| **mysql-mes** | `mcp__mysql-mes__mysql_query` | 直连 MySQL 执行查询 |
| **auggie-mcp** | `mcp__auggie-mcp__codebase-retrieval` | 代码库语义检索 |

## 文档 / 知识

| MCP | 工具 | 用途 |
|-----|------|------|
| **context7** | `resolve-library-id` + `query-docs` | 查询库/框架最新官方文档和代码示例 |
| **tavily** | `search` / `extract` / `crawl` / `research` | 网络搜索、网页内容提取、深度调研 |

## 设计 / 图表

| MCP | 工具 | 用途 |
|-----|------|------|
| **drawio-mcp** | `open_drawio_xml/csv/mermaid` | 生成架构图、流程图、ER 图 |
| **xmind** | `generate-mind-map` / `read-mind-map` | 生成/读取思维导图 |
| **openpencil** | `design_skeleton` / `design_content` 等 | UI 设计稿生成和编辑 |

## uTools 工具集

| 功能分类 | 工具 | 用途 |
|---------|------|------|
| **文件搜索** | `utools_everythingfind_search_files` | 本地文件极速搜索（替代 Glob/find）|
| **PDF** | `pdf_convert` / `pdf_merge` / `pdf_split` / `pdf_to_img` 等 | PDF 转换、合并、拆分、提取图片 |
| **图片** | `image_compress` / `image_resize` / `image_convert` / `image_remove_bg` 等 | 图片处理全套 |
| **视频** | `video_clip` / `video_compress` / `video_convert` / `video_extract_audio` 等 | 视频剪辑、压缩、转码 |
| **屏幕** | `screen_capture` / `start_screen_record` | 截图、录屏 |
| **剪贴板** | `clipboard_copy` / `clipboard_history_get` | 剪贴板读写和历史 |
| **二维码** | `qrcode_generator` / `qrcode_decoder` | 生成/解码二维码 |
| **图床上传** | `utools_zcne9mnz_upload_images` | 图片上传到配置的存储源 |

## 上下文管理

| MCP | 工具 | 用途 |
|-----|------|------|
| **context-mode** | `batch_execute` / `execute` / `execute_file` / `search` / `fetch_and_index` | 大输出沙箱执行，避免上下文污染；替代 Bash 大命令、Read 分析、WebFetch |

---

## CCG 多模型协作工作流

> `npx ccg-workflow` 安装，Claude 作编排，Codex 负责后端，Gemini 负责前端，外部模型只返回 patch，Claude 审核后应用。

### 开发工作流

| 命令 | 说明 | 路由 |
|------|------|------|
| `/ccg:workflow` | 完整 6 阶段开发流程 | Codex + Gemini |
| `/ccg:plan` | 多模型协作规划 | Codex + Gemini |
| `/ccg:execute` | 多模型协作执行 | Codex + Gemini + Claude |
| `/ccg:codex-exec` | Codex 全权执行（省 Claude token） | Codex + 多模型审核 |
| `/ccg:feat` | 智能功能开发，自动路由 | 自动 |
| `/ccg:frontend` | 前端专项 | Gemini |
| `/ccg:backend` | 后端专项 | Codex |

### 分析 / 质量

| 命令 | 说明 |
|------|------|
| `/ccg:analyze` | 技术分析（双模型交叉） |
| `/ccg:debug` | 问题诊断 + 修复 |
| `/ccg:optimize` | 性能优化 |
| `/ccg:test` | 测试生成（自动路由） |
| `/ccg:review` | 代码审查（自动 git diff） |

### Spec 规范驱动（OPSX）

| 命令 | 说明 |
|------|------|
| `/ccg:spec-init` | 初始化 OPSX 环境 |
| `/ccg:spec-research` | 需求 → 约束集 |
| `/ccg:spec-plan` | 约束 → 零决策可执行计划 |
| `/ccg:spec-impl` | 执行计划 + 归档 |
| `/ccg:spec-review` | 双模型交叉审查 |

### Agent Teams 并行（v1.7.60+）

| 命令 | 说明 |
|------|------|
| `/ccg:team-research` | 需求 → 约束（并行探索） |
| `/ccg:team-plan` | 约束 → 并行实施计划 |
| `/ccg:team-exec` | 派生 Builder 并行写代码 |
| `/ccg:team-review` | 双模型交叉审查 |

> 需在 `settings.json` 开启：`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

### Git 工具

| 命令 | 说明 |
|------|------|
| `/ccg:commit` | 智能提交（Conventional Commit 格式） |
| `/ccg:rollback` | 交互式回滚 |
| `/ccg:clean-branches` | 清理已合并分支 |
| `/ccg:worktree` | Worktree 管理 |

### 项目初始化

| 命令 | 说明 |
|------|------|
| `/ccg:init` | 初始化项目 CLAUDE.md |
| `/ccg:context` | 上下文管理（初始化/记录/压缩/历史） |

---

## 常用场景速查

| 场景 | 用哪个 MCP |
|------|-----------|
| 搜索本地文件 | uTools / everythingfind |
| 查库文档 | context7 |
| 搜索网络资料 | tavily |
| 操作服务器 | ssh-mpc-server |
| 操作 GitHub 仓库 | github |
| 查询数据库 | mysql-mes |
| 画架构图 | drawio-mcp |
| 大命令输出处理 | context-mode |
