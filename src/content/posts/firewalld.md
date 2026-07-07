---
title: Firewalld 防火墙生产运维完整手册
published: 2026-07-06
description: 涵盖 Firewalld 核心命令、区域管理、端口与服务配置、以及高级规则与故障排查的完整指南。
image: api
tags: [Linux, 防火墙, 安全, Firewalld, 运维]
category: 系统安全
draft: false
---

Firewalld 通过“区域 (Zone)”的概念简化了防火墙的管理，能够实现动态规则配置，无需重启服务即可生效。

---

## 一、 服务管理基础
| 命令 | 描述 |
| :--- | :--- |
| `systemctl status firewalld` | 查看防火墙状态 |
| `systemctl start / stop firewalld` | 启动/停止防火墙 |
| `systemctl enable / disable firewalld` | 设置开机自启/关闭自启 |
| `firewall-cmd --state` | 检查防火墙是否在运行 |
| `firewall-cmd --reload` | **重新加载规则** (配置修改后执行，不中断连接) |

---

## 二、 区域 (Zone) 管理
区域是 Firewalld 的核心，代表了不同的信任等级。

| 命令 | 描述 |
| :--- | :--- |
| `firewall-cmd --get-default-zone` | 查看默认区域 |
| `firewall-cmd --get-active-zones` | 查看当前激活的区域及其绑定的网卡 |
| `firewall-cmd --list-all` | 查看当前默认区域的所有配置 |
| `firewall-cmd --list-all --zone=public` | 查看指定区域（如 public）的规则 |
| `firewall-cmd --set-default-zone=public` | 设置默认区域 |

---

## 三、 端口与服务操作
注意：所有操作建议添加 `--permanent` 参数以永久生效，添加后需执行 `--reload`。

### 1. 端口操作
| 命令 | 描述 |
| :--- | :--- |
| `firewall-cmd --permanent --add-port=80/tcp` | 永久放行 80 端口 |
| `firewall-cmd --permanent --remove-port=80/tcp` | 移除 80 端口放行 |
| `firewall-cmd --permanent --add-port=8000-9000/tcp` | 放行端口段 |

### 2. 服务操作
| 命令 | 描述 |
| :--- | :--- |
| `firewall-cmd --get-services` | 查看系统中预定义的所有服务 |
| `firewall-cmd --permanent --add-service=http` | 永久放行 HTTP 服务 |
| `firewall-cmd --permanent --remove-service=ssh` | 移除 SSH 服务放行 |

---

## 四、 高级规则与富规则 (Rich Rules)
当简单端口放行无法满足需求时，使用富规则控制特定 IP 或复杂逻辑。

| 命令 | 描述 |
| :--- | :--- |
| `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" accept'` | 仅允许特定 IP 访问 |
| `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port port="22" protocol="tcp" reject'` | 禁止特定网段访问 SSH |
| `firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080` | **端口转发**：将 80 转发至 8080 |



---

## 五、 运维排查与故障处理

### 1. 规则验证
* 查看所有规则：`firewall-cmd --list-all-zones`
* 检查某端口是否开放：`firewall-cmd --query-port=80/tcp`

### 2. 故障排除小贴士
* **临时生效**：如果不带 `--permanent` 参数，命令立即生效但重启后失效，常用于测试。
* **规则冲突**：如果不小心删除了关键服务（如 SSH），请确保先配置新规则再移除旧规则，否则可能导致远程连接断开。
* **日志查看**：如果规则不生效，检查系统日志：`journalctl -u firewalld`

---

## 六、 生产环境建议
1. **最小权限原则**：默认将 `default-zone` 设置为 `drop` 或 `public`，只手动放行明确需要的端口。
2. **拒绝所有**：配置完毕后，可以通过 `firewall-cmd --panic-on` 开启紧急模式（拒绝所有网络通信），仅在遭遇攻击时使用。
3. **版本控制**：对于生产环境，可以将常用的规则整理成脚本（`sh` 文件），以便在部署新服务器时快速批量应用。