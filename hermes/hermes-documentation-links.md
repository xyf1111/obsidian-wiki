---
title: Hermes 文档与学习资源
tags: [hermes, documentation, reference, links]
created: 2026-06-07
updated: 2026-06-07
---

# Hermes 文档与学习资源

## 官方资源

| 资源 | 链接 |
|------|------|
| 主仓库 | [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) |
| 阿里云百炼接入 | [help.aliyun.com - Hermes Agent](https://help.aliyun.com/zh/model-studio/hermes-agent) |
| 英文文档 | [alibabacloud.com - Hermes Agent](https://www.alibabacloud.com/help/en/model-studio/hermes-agent) |

## 社区教程（2026 年热点）

| 资源 | 说明 |
|------|------|
| [Hermes 优化指南 (24 篇)](https://github.com/OnlyTerp/hermes-optimization-guide) | 涵盖 setup、migration、LightRAG、Telegram、skill creation |
| [Oh My Hermes](https://github.com/Salomondiei08/oh-my-hermes) | 意见化的 Hermes 工作流层，构建/发布/运维应用 |
| [Hermes Code Bridge](https://github.com/xuyang-liu16/hermes-code-bridge) | Hermes 作为控制平面协调查找 Claude Code、Codex、Kimi Code 等 |
| [技能系统详解](https://www.hostinger.com/tutorials/what-are-hermes-agent-skills) | Hostinger 出品的技能系统 Complete Guide |

## 阿里云实战系列

| 文章 | 链接 |
|------|------|
| Hermes + Claude Code 协同开发团队架构 | [developer.aliyun.com/article/1733545](https://developer.aliyun.com/article/1733545) |
| DevBox 一键部署 Hermes + Claude Code | [developer.aliyun.com/article/1735400](https://developer.aliyun.com/article/1735400) |
| Hermes + Claude Code 阿里云一键部署教程 | [developer.aliyun.com/article/1734011](https://developer.aliyun.com/article/1734011) |
| Hermes Agent + Claude Code 接入百炼 Token Plan | [developer.aliyun.com/article/1736041](https://developer.aliyun.com/article/1736041) |

## 重要版本更新 (v0.14.0, 2026-05)

| 特性 | 说明 |
|------|------|
| PyPI 安装 | `pip install hermes-agent`，启动速度提升约 19s |
| `hermes proxy` | OpenAI 兼容端点，复用 OAuth 订阅 |
| `x_search` | X/Twitter 原生搜索 |
| Teams 端到端 | Teams + LINE + SimpleX Chat 支持 |
| `/handoff` | 无损会话中转切换模型 |
| 持久化看板 | 多代理看板 heartbeat/retry/zombie 检测 |
| `/goal` | 持久化目标跟踪 |
| Curator | 自动评估、加固、归档代理创建的技能 |
| Windows Beta | 初步 Windows 支持 |

## 推荐阅读顺序

1. 先读 [[hermes-agent]] 了解它是什么
2. 再看 [[hermes-setup-guide]] 安装上手
3. 深入 [[hermes-skills-system]] 掌握技能系统
4. 实战：阿里云系列文章，搭建 Hermes + Claude Code 协同环境

## 相关页面

- [[hermes-agent]] — 概述与架构
- [[hermes-setup-guide]] — 安装配置
- [[hermes-skills-system]] — 技能系统
