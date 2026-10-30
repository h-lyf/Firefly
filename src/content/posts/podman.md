---
title: Podman 与 Quadlet 常用命令及声明式运维指南
published: 2026-10-30
description: 涵盖 Podman 容器生命周期、Pod 组网、镜像管理、网络存储，以及基于 Quadlet 的 Systemd 声明式容器服务配置。
image: api
tags: [Podman, Quadlet, Systemd, 容器化]
category: 系统管理
draft: false
---

Podman 作为无守护进程（Daemonless）、原生支持无根运行（Rootless）的新一代容器引擎，在安全性与系统集成度上具有天然优势。配合 Systemd 现代化的 Quadlet 机制，能够轻松实现容器服务的声明式管理。

本文汇集了 Podman 的日常常用命令以及 Quadlet 的配置实践，助你高效运维。

---

## 一、 Podman 基础容器管理

Podman 命令与 Docker 高度兼容（大多数情况下可将 `podman` 直接别名为 `docker`），且默认支持**普通用户无特权运行**。

### 1. 生命周期控制
| 命令 | 描述 |
| :--- | :--- |
| `podman run -d --name <name> <image>` | 后台启动容器（默认 Rootless 模式） |
| `podman stop / start / restart <id>` | 启停与重启容器 |
| `podman rm -f <id>` | 强制删除容器 |
| `podman generate systemd --name <name>` | *(旧版方式)* 为运行中的容器生成 Systemd 服务文件 |

### 2. 交互与调试
| 命令 | 描述 |
| :--- | :--- |
| `podman exec -it <id> sh` | 进入容器内部终端 |
| `podman logs -f --tail 100 <id>` | 实时跟踪最后 100 行日志 |
| `podman inspect <id>` | 查看容器的详细 JSON 配置信息 |
| `podman top <id>` | 查看容器内运行的进程树（宿主机视角） |
| `podman diff <id>` | 查看容器内文件系统的修改历史 |

---

## 二、 Pod 管理（Podman 核心特色）

Podman 引入了 Kubernetes 中的 **Pod** 概念，允许多个容器共享同一个网络与存储空间（Network/IPC Namespace）。

| 命令 | 描述 |
| :--- | :--- |
| `podman pod create --name <pod_name> -p 8080:80` | 创建一个共享网络端口的 Pod |
| `podman run -d --pod <pod_name> --name web nginx` | 将容器加入指定 Pod |
| `podman pod ls` | 查看当前所有 Pod |
| `podman pod top <pod_name>` | 查看 Pod 内所有容器的进程 |
| `podman pod rm -f <pod_name>` | 删除 Pod 及其关联的所有容器 |

---

## 三、 Podman 网络与存储管理

### 1. 网络管理
| 命令 | 描述 |
| :--- | :--- |
| `podman network ls` | 查看所有容器网络 |
| `podman network create <name>` | 创建自定义网络（Rootless 默认使用 netavark/pasta） |
| `podman network connect <net> <id>` | 将容器加入指定网络 |
| `podman network inspect <net>` | 查看网络详情及连接的容器 |

### 2. 数据卷 (Volume) 管理
| 命令 | 描述 |
| :--- | :--- |
| `podman volume ls` | 列出所有数据卷 |
| `podman volume inspect <name>` | 查看卷在宿主机的物理路径（通常位于 `~/.local/share/containers/`） |
| `podman volume prune` | 删除所有未被使用的卷 |

---

## 四、 Podman Quadlet 声明式配置（重点）

**Quadlet** 是 Podman 自 v4.4+ 推出的现代化容器管理工具。它允许你使用类似 Systemd `.service` 的声明式文件（`.container`、`.pod`、`.volume`、`.network`），自动生成并由 Systemd 原生接管容器服务。

### 1. Quadlet 配置文件存放路径
*   **用户级（Rootless，推荐）**：`~/.config/containers/systemd/`
*   **系统级（Root）**：`/etc/containers/systemd/`

### 2. 编写 Quadlet 示例：Nginx 容器 (`nginx.container`)
在 `~/.config/containers/systemd/nginx.container` 中新建文件：

```ini
[Unit]
Description=Nginx Web Server Container
After=network-online.target

[Container]
Image=docker.io/library/nginx:latest
ContainerName=my-nginx
PublishPort=8080:80
Volume=nginx-data:/usr/share/nginx/html:ro
Exec=nginx -g "daemon off;"

[Install]
# 随系统启动/用户登录自动加载
WantedBy=default.target
```

### 3. 常用 Quadlet 指令映射表
| Quadlet 配置项 (`[Container]`) | 等价 Podman 参数 | 描述 |
| :--- | :--- | :--- |
| `Image=nginx:latest` | `nginx:latest` | 镜像名称 |
| `PublishPort=8080:80` | `-p 8080:80` | 映射端口 |
| `Volume=/host/path:/container/path` | `-v /host/path:/container/path` | 挂载卷/目录 |
| `Environment=KEY=VAL` | `-e KEY=VAL` | 环境变量 |
| `Network=my-net.network` | `--network my-net` | 关联 Quadlet 定义的网络或普通网络 |
| `Pod=my-pod.pod` | `--pod my-pod` | 将容器放入 Quadlet 定义的 Pod |

### 4. Quadlet 常用管理命令
| 命令 | 描述 |
| :--- | :--- |
| `systemctl --user daemon-reload` | 解析/重载 Quadlet 配置文件并生成 Systemd 服务 |
| `systemctl --user status nginx` | 查看生成的容器服务状态 |
| `systemctl --user start / stop nginx` | 启停 Quadlet 容器服务 |
| `systemctl --user enable nginx` | 设置容器开机/登录自启 |
| `/usr/libexec/podman/quadlet --dryrun` | 预览 Quadlet 生成的原生 Systemd 指令（调试用） |

---

## 五、 镜像与系统维护

### 1. 镜像维护
| 命令 | 描述 |
| :--- | :--- |
| `podman images` | 列出本地所有镜像 |
| `podman build -t <name>:<tag> .` | 构建镜像（基于 Buildah，无需运行守护进程） |
| `podman pull / push <image>` | 拉取 / 推送镜像 |
| `podman rmi $(podman images -f "dangling=true" -q)` | 清理悬空镜像 |

### 2. 资源监控与清理
| 命令 | 描述 |
| :--- | :--- |
| `podman stats` | 实时监控 CPU/内存/网络/磁盘 IO |
| `podman system df` | 查看容器、镜像及卷的磁盘占用统计 |
| `podman system prune -a --volumes` | **深度清理**：删除停止的容器、未使用的镜像及卷 |

---

## 六、 Podman & Quadlet 生产最佳实践

1.  **全面转向 Rootless 模式**：优先以普通用户运行容器，有效抵御容器逃逸风险。
2.  **开启普通用户驻留（Keep-Alive）**：
    Rootless 容器若需随开机自启，需执行以下命令允许用户脱机运行进程：
    ```bash
    loginctl enable-linger $USER
    ```
3.  **使用 Quadlet 代替传统 Compose/Cron**：将生产服务配置写成 `.container` 或 `.pod` 文件，交由 Systemd 进行健康检查、依赖控制与日志收集（`journalctl -u nginx.service`）。
4.  **SELinux 标签策略**：在启用了 SELinux 的系统（如 RHEL/Fedora/CentOS）上挂载宿主机目录时，需在路径后增加 `:z` 或 `:Z` 选项，例如：`-v /data:/app:z`。