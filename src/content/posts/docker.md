---
title: Docker 与 Docker Compose 生产运维完整手册
published: 2026-07-05
description: 涵盖 Docker 容器生命周期、网络管理、持久化存储、镜像优化及 Docker Compose 多容器编排的生产级运维指南。
image: api
tags: [Docker, Compose, 运维, 容器化, 完整手册]
category: 系统管理
draft: false
---

# Docker 与 Docker Compose 运维手册

本手册汇集了从基础入门到生产环境排查的全部核心命令，助你高效管理容器化应用。

---

## 一、 Docker 基础容器管理

### 1. 生命周期控制
| 命令 | 描述 |
| :--- | :--- |
| `docker run -d --name <name> <image>` | 后台启动容器 |
| `docker stop / start / restart <id>` | 启停与重启容器 |
| `docker rm -f <id>` | 强制删除容器 |
| `docker update --restart=always <id>` | 修改运行中容器的重启策略 |

### 2. 交互与调试

| 命令 | 描述 |
| :--- | :--- |
| `docker exec -it <id> sh` | 进入容器内部终端 |
| `docker logs -f --tail 100 <id>` | 实时跟踪最后 100 行日志 |
| `docker inspect <id>` | 查看容器的详细 JSON 配置信息 |
| `docker top <id>` | 查看容器内运行的进程树 |
| `docker diff <id>` | 查看容器内文件系统的修改历史 |

---

## 二、 Docker 网络与存储管理

### 1. 网络管理

| 命令 | 描述 |
| :--- | :--- |
| `docker network ls` | 查看所有网络 |
| `docker network create <name>` | 创建自定义网桥 |
| `docker network connect <net> <id>` | 将容器加入指定网络 |
| `docker network inspect <net>` | 查看网络详情及加入的容器 |

### 2. 数据卷 (Volume) 管理
| 命令 | 描述 |
| :--- | :--- |
| `docker volume ls` | 列出所有数据卷 |
| `docker volume inspect <name>` | 查看卷在宿主机的物理路径 |
| `docker volume prune` | 删除所有未被使用的匿名卷 |

---

## 三、 Docker Compose 多容器编排

### 1. 服务启停与管理
| 命令 | 描述 |
| :--- | :--- |
| `docker-compose up -d` | 后台启动所有编排服务 |
| `docker-compose down` | 停止并移除容器、网络 |
| `docker-compose restart` | 重启所有服务容器 |
| `docker-compose ps` | 查看各服务运行状态 |
| `docker-compose logs -f <service>` | 实时查看指定服务日志 |

### 2. 扩缩容与配置
| 命令 | 描述 |
| :--- | :--- |
| `docker-compose scale <service>=N` | 扩展指定服务实例数量 |
| `docker-compose config` | 验证并输出格式化后的配置 |
| `docker-compose exec <service> sh` | 进入指定编排服务的内部 |

---

## 四、 镜像与系统维护

### 1. 镜像维护
| 命令 | 描述 |
| :--- | :--- |
| `docker images -a` | 列出所有镜像 |
| `docker image history <image>` | 查看镜像的分层构建细节 |
| `docker build -t <name>:<tag> .` | 构建镜像 |
| `docker rmi $(docker images -f "dangling=true" -q)` | 清理所有无标签的悬空镜像 |

### 2. 资源监控与清理

| 命令 | 描述 |
| :--- | :--- |
| `docker stats` | 实时监控 CPU/内存/IO 占用 |
| `docker system df` | 查看磁盘占用统计 |
| `docker system prune -a --volumes` | **深度清理**：删除所有停止的容器、未使用镜像及匿名卷 |

---

## 五、 生产环境运维原则

1.  **资源限制**：在 `docker run` 或 `compose` 中务必添加 `--memory` 和 `--cpus` 参数。
2.  **日志轮转**：在 `daemon.json` 中配置 `max-size` 以防磁盘被日志写满。
3.  **禁止容器内写入**：重要数据文件必须通过 `-v` (Volume) 挂载到宿主机。
4.  **自动重启**：所有生产容器应配置 `restart: unless-stopped` 策略，以应对宿主机重启。