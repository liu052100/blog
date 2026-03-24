---
title: 博客自动化部署搭建记录
date: 2026-03-24 16:00:00
tags: [运维, GitHub Actions, Hexo]
categories: 技术
---

今天把博客的部署流程从手动改成了全自动，记录一下整个过程。

<!-- more -->

## 背景

之前每次写完文章都要手动 SSH 到服务器执行 `hexo generate`，比较麻烦。

## 方案

使用 GitHub Actions，在每次 `git push` 到 master 分支时自动触发：

1. SSH 连接到服务器
2. `git pull` 拉取最新代码
3. `npm install` 安装依赖
4. `hexo generate` 生成静态文件

## 核心配置

```yaml
- name: Deploy via SSH
  run: |
    ssh -i ~/.ssh/deploy_key \
      -o StrictHostKeyChecking=no \
      root@server_ip \
      'cd /var/www/hexo && git pull origin master && npx hexo generate'
```

## 效果

推送代码后约 30 秒博客自动更新，无需手动操作。
