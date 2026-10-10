---
title: 从 mysql_mcp_server 换到 DBX：让 AI 直接用数据库客户端里现成的连接
date: 2026-10-10 17:00:00
tags:
  - MCP
  - DBX
  - MySQL
  - AI
  - Claude Code
categories:
  - 工具
---

> **测试环境**：Windows 11，DBX 0.6.38（2026-10-09 发布），Claude Code 2.1.296，2026-10-10。
> 这篇替代我 4 月写的《[MySQL MCP Server：让 AI 助手直接操作你的数据库](/2026/04/07/mysql-mcp-server-guide/)》，那个方案我已经不用了。

---

## 为什么换

4 月那篇用的是 `designcomputer/mysql_mcp_server`，用了一段时间，几个问题越来越明显：

| | mysql_mcp_server | DBX |
|---|---|---|
| 支持的数据库 | 只有 MySQL | MySQL、PostgreSQL、SQL Server、Oracle、Redis、MongoDB 等 90 多种 |
| 一个 MCP 能连几个库 | 一个，库名写死在环境变量里 | DBX 里存的连接都能用，按连接 + `database` 参数切换 |
| 账号密码放哪 | MCP 配置的环境变量里，明文 | 留在 DBX 自己的连接配置里，MCP 配置里只有一个令牌 |
| 权限控制 | 没有，全靠数据库账号 | DBX 里集中设：只读 / 数据读写 / 完全访问 + 允许访问的连接清单 |
| 返回量 | 读表固定 `LIMIT 100` | 默认 100 行、最多 1000 行；长文本截断后可以接着往后读 |
| 人能不能一起看 | 看不到 | 可以让 AI 把表或查询结果直接在 DBX 窗口里打开 |

最实际的一点：我本来就天天开着 DBX 当数据库客户端，连接都在里面配好了。换成它的 MCP 之后，AI 用的就是这些现成的连接，不用再为每个库单独配一个 MCP、再抄一遍密码。

---

## DBX 是什么

[t8y2/dbx](https://github.com/t8y2/dbx)：Rust + Tauri 写的开源数据库客户端（Apache-2.0），安装包 25MB 左右，有桌面端、Docker（Web 版）和命令行，自带 AI 助手和 MCP Server。更新很勤，差不多隔天一版；GitHub 下载慢的话，Release 页里给了国内的 CNB 镜像。

---

## 接入 Claude Code

两种接法，二选一。

### 方式一：桌面端自带的 HTTP 服务（我用的）

DBX 桌面端开着的时候，本机 `127.0.0.1:5225/mcp` 上就有一个 MCP 服务，用 Bearer 令牌认证。令牌在 DBX「设置 → MCP」里查看和轮换（Windows 上也存在 `%APPDATA%\com.dbx.app\mcp-http-token`）。

```bash
claude mcp add --transport http -s user dbx http://127.0.0.1:5225/mcp \
  --header "Authorization: Bearer <你的令牌>"
```

几点注意：

- **DBX 桌面端要一直开着**，关掉之后 Claude Code 这边就连不上。
- 在 DBX 里轮换令牌后，旧令牌立即失效，Claude Code 报 401。先 `claude mcp remove dbx -s user`，用新令牌再加一次，然后在 Claude Code 里 `/mcp` 重连。
- 默认只监听本机回环地址，局域网里别的电脑访问不到。

### 方式二：npm 包（stdio）

```bash
claude mcp add -s user dbx -- npx -y @dbx-app/mcp-server
```

它直接读 DBX 的本地连接存储，普通查询不需要桌面端开着（只有「在 DBX 窗口里打开表 / 展示结果」这两个工具需要）。要求 Node.js 18.18 以上；不想装 Node，也可以去 Release 页下载原生二进制，把 `command` 指向它。这种接法我没有长期用，细节以[官方说明](https://github.com/t8y2/dbx/blob/main/packages/mcp-server/README.md)为准。

---

## 能干什么

我这边接上之后有 25 个工具，按用途分：

| 用途 | 工具 |
|---|---|
| 找连接、找库、找表 | `dbx_list_connections` / `dbx_list_databases` / `dbx_list_tables` |
| 看表结构 | `dbx_describe_table`（一张表）、`dbx_get_schema_context`（多张表的紧凑结构，适合让 AI 写 SQL 前先看一眼） |
| 跑 SQL | `dbx_execute_query`（单条或多条）、`dbx_execute_batch`（脚本，可以整体放进一个事务） |
| 会话与事务 | `dbx_open_session` / `dbx_close_session`（保持 `USE`、`SET`、临时表），`dbx_begin_transaction` / `commit` / `rollback` |
| 存储过程 | `dbx_list_routines` / `dbx_get_routine_source` |
| Redis | `dbx_execute_redis_command` |
| 消息队列 | `dbx_peek_messages`（读 Kafka，不提交消费位点）/ `dbx_send_message` |
| 在 DBX 窗口里展示 | `dbx_open_table` / `dbx_execute_and_show` |
| 管理连接配置 | `dbx_add_connection` / `dbx_duplicate_connection` / `dbx_remove_connection` |
| Salesforce | `dbx_salesforce_*` 三个 |

平时说「看看订单表最近 7 天有多少条」「这两张表怎么关联」，AI 会自己先列表、看结构、再写 SQL 跑。

---

## 权限：在 DBX 里集中管

DBX「设置 → MCP」里存一份策略，每次请求都会重新读取，改完立即生效：

| 模式 | 允许的操作 |
|---|---|
| 只读 | 查询、读元数据 |
| 数据读写 | 普通插入、带有效过滤条件的更新 / 删除、普通 Redis 写入 |
| 完全访问 | 再加上大范围更新删除、DDL、`TRUNCATE`、Redis `FLUSH` 等 |

另外有一份「允许访问的连接」清单，不在清单里的连接，AI 根本看不到。`WHERE 1 = 1`、`WHERE TRUE` 这类「假过滤」会被当成高风险操作。连接本身设的只读、数据库账号自己的权限，在任何模式下都是上限。

我的建议：DBX 的策略再细，也是软件层面的一道闸。**生产库最稳的做法仍然是让 DBX 用只读账号去连**——4 月那篇里建只读账号的 SQL 照样适用：

```sql
CREATE USER 'ai_ro'@'%' IDENTIFIED BY 'your_secure_password';
GRANT SELECT ON your_database.* TO 'ai_ro'@'%';
```

---

## 实际用下来的几个坑

1. **调用时用连接 ID，别用连接名。** 名字可以重复：我有三个连接都用服务器地址命名（同一台机器上的 MySQL、SQL Server、Oracle），按名字调用会报 `AMBIGUOUS_CONNECTION` 并列出候选 ID。好在它是报错而不是随便挑一个。可以在 `CLAUDE.md` 里列一张「连接 ID ↔ 用途」的表，让 AI 照着用。
2. **默认只回 100 行，长文本只回前 200 个字符。** `max_rows` 最多 1000；单元格被截断时，结果里会标 `[chars 0..200; next cell_char_offset=200]`，用 `cell_char_limit`（最大 4000）和 `cell_char_offset` 接着往后读。查 JSON、长配置字段时，别把截断后的内容当成原文。一次发多条语句时返回的是 JSON 格式，和单条时的表格格式不一样，别奇怪。
3. **同一个连接下有多个库，用 `database` 参数切换。** MySQL 连接不传 `database` 时，表名要写全 `库名.表名`。
4. **排序规则不一致会报 1267。** 连接默认用的 `utf8mb4_0900_ai_ci` 和库的 `utf8mb4_unicode_ci` 不一样时，字符串比较或 JOIN 可能撞上 `ERROR 1267 Illegal mix of collations`，在 SQL 里显式加 `COLLATE` 就行。
5. **`EXPLAIN` 可能把整行数据带出来。** 按主键查常量时，MySQL 会把命中的那一行放进服务器消息里一起返回，我见过连密码哈希列一起带出来的。对敏感表做 `EXPLAIN` 时留个心。
6. **Windows 上别直接运行 `dbx.exe --help`**，它会拉起图形界面，而不是打印帮助。
7. **升级偶尔会出协议兼容问题。** 10-09 自动升级到 0.6.37 后，DBX 声称支持新版 MCP 协议，但返回的结果少了必填字段，Claude Code 能连上却一个工具都拿不到（报 `missing required resultType`）。我当时临时写了个转接脚本，把协议版本压回旧版顶过去；第二天 0.6.38 就修好了，直连恢复正常。遇到「连上了但没有工具」，先想想是不是刚自动升级过。

---

## 怎么选

- 只有一个 MySQL、只想让 AI 读读表：`mysql_mcp_server` 依然够用，足够轻。
- 平时本来就用数据库客户端、要连好几种数据库、想要一个集中的权限开关：DBX 省事得多。

---

## 声明

DBX 能力介绍参考了 [t8y2/dbx](https://github.com/t8y2/dbx) 官方 README（Apache-2.0）；接入方式、返回格式和踩坑部分是我在 0.6.38 上实际用出来的。
