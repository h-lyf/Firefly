---
title: AdGuard 个人过滤规则编写指南
published: 2026-07-07
description: 深入解析 AdGuard 自定义过滤规则语法，包括基础过滤、元素隐藏、高级修饰符及排除规则，助你精准拦截广告。
image: api
tags: [AdGuard, 广告拦截, 隐私保护, 教程, 规则编写]
category: 系统管理
draft: false
---

AdGuard 的自定义规则基于 Adblock Plus 标准，并在此基础上进行了扩展。掌握这些语法，可以帮你精准拦截特定域名或网页元素。

## 1. 基础过滤规则 (Blocking Rules)
最简单的用法是直接屏蔽特定链接。

| 语法 | 说明 | 示例 |
| :--- | :--- | :--- |
| `\|\|domain.com^` | 拦截该域名及其所有子域名 | `\|\|ads.example.com^` |
| `domain.com` | 只要 URL 中包含此字符串即拦截 | `example.com/ad/` |
| `\|http://domain.com` | 从 URL 开头进行精确匹配 | `\|http://badsite.com` |
| `domain.com^` | 拦截该域名下的所有请求 | `tracker.com^` |

---

## 2. 元素隐藏规则 (Cosmetic Rules)
用于隐藏网页上残留的空白位、弹窗或特定的广告容器。

| 语法 | 说明 | 示例 |
| :--- | :--- |
| `domain.com##.selector` | 隐藏 CSS 类名为 selector 的元素 | `baidu.com##.ad-banner` |
| `domain.com##div#id` | 隐藏 ID 为 id 的 DIV 元素 | `example.com##div#popup` |
| `domain.com##.class:has-text(广告)` | 隐藏包含特定文字的元素 | `##.content:has-text(推广)` |
| `domain.com#?#css-selector` | 使用高级选择器（如伪类） | `##.sidebar:has(a[href*="ads"])` |

---

## 3. 高级修饰符 (Modifiers)
通过修饰符可以精确控制规则生效的条件。

* **指定域名**：仅在特定网站生效。
  `||ads.com^$domain=example.com`
* **限制资源类型**：只拦截图片、脚本或框架。
  `||ads.com^$image,script,subdocument`
* **排除规则**：让某些请求避开拦截（白名单）。
  `@@||white-list.com^`

---

## 4. 进阶调试与实战技巧



### 实战案例：
1. **彻底清理页面广告位**：
   ```markdown
   ! 屏蔽某网站右侧广告栏
   example.com##.sidebar-right
   ! 屏蔽页面底部浮窗
   example.com##.float-bottom-fixed