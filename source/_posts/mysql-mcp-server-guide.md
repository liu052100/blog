---
title: MySQL MCP Server：让 AI 助手直接操作你的数据库
date: 2026-04-07 20:00:00
tags:
  - MCP
  - MySQL
  - AI
  - Claude Code
categories:
  - 工具
---

> **项目地址**：[designcomputer/mysql_mcp_server](https://github.com/designcomputer/mysql_mcp_server) | **作者**：Dana K. Williams | **版本**：0.2.2 | **协议**：MIT

MCP（Model Context Protocol）是 Anthropic 推出的一套标准协议，让 AI 助手能够通过结构化接口与外部工具交互。`mysql_mcp_server` 是一个基于 MCP 的 MySQL 连接器，装上之后，Claude Desktop、VSCode Copilot、Claude Code 等 AI 工具就能直接查询和操作你的 MySQL 数据库。

---

## 它能做什么

- **列出数据库中的所有表**（作为 MCP Resource 暴露）
- **读取表内容**（默认 LIMIT 100 行）
- **执行任意 SQL 查询**（SELECT / INSERT / UPDATE / DELETE / SHOW / DESCRIBE 等）
- 完整的错误处理和日志记录

核心代码只有一个文件（`server.py`，约 200 行），非常轻量。底层依赖 `mysql-connector-python` 连接数据库，通过 MCP SDK 的 stdio 模式与 AI 客户端通信。

---

## 安装

### pip 安装

```bash
pip install mysql-mcp-server
```

### Smithery 一键安装（Claude Desktop）

```bash
npx -y @smithery/cli install mysql-mcp-server --client claude
```

---

## 配置

### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `MYSQL_HOST` | 数据库主机 | `localhost` |
| `MYSQL_PORT` | 端口 | `3306` |
| `MYSQL_USER` | 用户名 | （必填） |
| `MYSQL_PASSWORD` | 密码 | （必填） |
| `MYSQL_DATABASE` | 数据库名 | （必填） |
| `MYSQL_CHARSET` | 字符集 | `utf8mb4` |
| `MYSQL_COLLATION` | 排序规则 | `utf8mb4_unicode_ci` |
| `MYSQL_SQL_MODE` | SQL 模式 | `TRADITIONAL` |

### Claude Desktop 配置

在 `claude_desktop_config.json` 中添加：

```json
{
  "mcpServers": {
    "mysql": {
      "command": "uv",
      "args": [
        "--directory",
        "path/to/mysql_mcp_server",
        "run",
        "mysql_mcp_server"
      ],
      "env": {
        "MYSQL_HOST": "localhost",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "your_username",
        "MYSQL_PASSWORD": "your_password",
        "MYSQL_DATABASE": "your_database"
      }
    }
  }
}
```

### VSCode 配置

在 `mcp.json` 中添加：

```json
{
  "servers": {
    "mysql": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "--from",
        "mysql-mcp-server",
        "mysql_mcp_server"
      ],
      "env": {
        "MYSQL_HOST": "localhost",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "your_username",
        "MYSQL_PASSWORD": "your_password",
        "MYSQL_DATABASE": "your_database"
      }
    }
  }
}
```

### Claude Code 配置

在 `~/.claude/settings.json` 中添加，或使用命令行：

```bash
claude mcp add mysql \
  -e MYSQL_HOST=localhost \
  -e MYSQL_PORT=3306 \
  -e MYSQL_USER=your_username \
  -e MYSQL_PASSWORD=your_password \
  -e MYSQL_DATABASE=your_database \
  -- uvx --from mysql-mcp-server mysql_mcp_server
```

> **Windows 用户注意**：如果 stdio 模式不工作，将 `command` 改为 `"cmd"`，`args` 前面加 `["/c", ...]`。

---

## 实现原理

整个项目只有一个核心文件 `src/mysql_mcp_server/server.py`，注册了三个 MCP 能力：

| MCP 接口 | 功能 | 实现 |
|----------|------|------|
| `list_resources()` | 列出所有表 | `SHOW TABLES` → 返回 `Resource` 列表 |
| `read_resource(uri)` | 读取表数据 | `SELECT * FROM {table} LIMIT 100` → CSV 格式 |
| `call_tool("execute_sql")` | 执行 SQL | 直接执行传入的 SQL，返回结果或影响行数 |

**连接方式**：每次请求都新建连接（`with connect(**config)`），没有连接池。对于低频 AI 交互场景足够，高频场景下可能是瓶颈。

**SQL 注入风险**：`read_resource` 中的表名是直接拼接的（`f"SELECT * FROM {table}"`），但由于输入来源是 MCP 协议而非用户直接输入，风险可控。`execute_sql` 工具则是完全开放的，传什么执行什么——安全性完全依赖于数据库用户权限。

---

## 安全最佳实践

这一点**非常重要**。既然 AI 可以执行任意 SQL，那数据库用户的权限控制就是唯一的安全屏障。

### 创建专用只读用户（推荐）

```sql
-- 创建专用用户
CREATE USER 'mcp_user'@'localhost' IDENTIFIED BY 'your_secure_password';

-- 只给 SELECT 权限（最安全）
GRANT SELECT ON your_database.* TO 'mcp_user'@'localhost';

-- 如果需要写入
GRANT SELECT, INSERT, UPDATE, DELETE ON your_database.* TO 'mcp_user'@'localhost';

FLUSH PRIVILEGES;
```

### 限制查询频率

```sql
ALTER USER 'mcp_user'@'localhost'
WITH MAX_QUERIES_PER_HOUR 1000
MAX_UPDATES_PER_HOUR 100;
```

### 列级权限控制

```sql
-- 只允许访问特定列（隐藏敏感数据）
GRANT SELECT (id, name, email) ON your_database.users TO 'mcp_user'@'localhost';
```

### 安全检查清单

- [ ] 绝不使用 root 账号
- [ ] 专用用户 + 最小权限
- [ ] 生产环境建议只读
- [ ] 密钥通过环境变量传递，不要硬编码
- [ ] 定期审计查询日志
- [ ] 考虑用 VIEW 替代直接表访问

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 0.2.2 | 2025-04-18 | 修复 `SHOW INDEX`、`SHOW CREATE TABLE`、`DESCRIBE` 等返回结果集的命令处理 |
| 0.2.1 | 2025-02-15 | 新增 `MYSQL_PORT` 环境变量支持，重构数据库配置为运行时加载 |
| 0.2.0 | 2025-01-20 | 首次发布，支持 SQL 查询、列表和读取表 |

---

## 个人评价

**优点**：
- 轻量简洁，核心代码不到 200 行，没有过度设计
- 安装配置简单，pip 一行搞定
- 支持 Claude Desktop / VSCode / Claude Code 等主流客户端
- MIT 开源，PyPI 可直接安装

**局限**：
- 没有连接池，每次请求新建连接
- 没有查询白名单或 SQL 过滤机制，安全性完全依赖数据库权限
- `read_resource` 中表名直接拼接，虽然 MCP 协议下风险低但不够严谨
- 不支持多数据库切换，一个实例只能连一个库
- 没有查询超时控制

**适用场景**：
- 开发环境中让 AI 助手快速探索数据库结构和数据
- 数据分析：自然语言 → SQL → 结果，AI 闭环
- 快速原型验证，不想写 CRUD 代码时让 AI 代劳

**不建议用于**：
- 生产环境直接操作（除非严格只读 + 权限隔离）
- 高频自动化任务（无连接池）

---

## 声明

本文基于 [designcomputer/mysql_mcp_server](https://github.com/designcomputer/mysql_mcp_server) 开源项目（MIT 协议）的源码和文档整理。项目作者为 Dana K. Williams。
