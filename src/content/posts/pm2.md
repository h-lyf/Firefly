---
title: PM2 常用命令速查手册
published: 2026-07-05
description: PM2 是 Node.js 应用生产环境的必备工具，本指南涵盖了进程管理、监控、集群模式及最佳实践配置。
image: api
tags: [Node.js, PM2, 运维, 教程]
category: 系统管理
draft: false
---

# PM2 常用命令速查手册

PM2 (Process Manager 2) 是 Node.js 应用最常用的生产环境进程管理器。它不仅能保证应用在崩溃后自动重启，还支持零停机重启和多核负载均衡。

## 1. 进程管理
这些命令用于日常的应用启停与状态管理。

| 命令 | 描述 |
| :--- | :--- |
| `pm2 start app.js --name "name"` | 启动应用并指定名称 |
| `pm2 start npm --name "name" -- run start` | 通过 npm 脚本启动应用 |
| `pm2 list` | 查看所有进程状态 |
| `pm2 stop <name\|id>` | 停止指定进程 |
| `pm2 restart <name\|id>` | 重启指定进程 |
| `pm2 reload <name\|id>` | **零停机平滑重启** (推荐用于生产环境) |
| `pm2 delete <name\|id>` | 删除进程 |

## 2. 监控与日志
用于排查问题和查看系统负载。


*注：运行 `pm2 monit` 可实时查看应用性能。*

| 命令 | 描述 |
| :--- | :--- |
| `pm2 monit` | 打开交互式监控面板，查看 CPU/内存 |
| `pm2 logs` | 查看所有日志输出 |
| `pm2 logs <name\|id>` | 查看特定进程的日志 |
| `pm2 flush` | 清空所有日志文件 |

## 3. 高级配置与集群
| 命令 | 描述 |
| :--- | :--- |
| `pm2 start app.js -i max` | 集群模式：根据 CPU 核心数启动实例 |
| `pm2 save` | 保存当前进程列表，防止服务器重启后丢失 |
| `pm2 startup` | 生成开机自启脚本 |
