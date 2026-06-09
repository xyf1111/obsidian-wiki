---
title: Hermes Agent
tags: [ai, coding-assistant, agent, terminal]
created: 2026-06-07
updated: 2026-06-07
---

# Hermes Agent

Hermes Agent 是由 **Nous Research** 开发的开源终端优先 AI 编程助手和智能体框架。它超越了一般的代码补全工具，具备持久记忆、技能沉淀、任务调度、多平台消息集成等能力。

## 核心能力

- **终端优先**：类 Claude Code 的 TUI 交互体验
- **持久记忆**：基于 LightRAG 的向量记忆系统，跨会话保留上下文
- **技能系统**：可复用的工作流模板，社区可共享
- **多代理调度**：可作为「技术主管」协调查找 Claude Code、Codex、Gemini CLI 等外部编码工具
- **多平台集成**：支持 Telegram、Discord、Slack、Teams、LINE 等消息平台
- **Cron 任务**：内置定时任务调度
- **Web 控制台**：可视化配置、日志、会话管理

## 与其他工具的对比

| 特性 | Hermes Agent | Claude Code | Codex | Cursor |
|------|-------------|-------------|-------|--------|
| 终端原生 | ✅ | ✅ | ✅ | ❌ (IDE) |
| 持久记忆 | ✅ LightRAG | ❌ (会话级) | ❌ (会话级) | ❌ |
| 技能系统 | ✅ 一等公民 | ✅ (斜杠命令) | ❌ | ❌ |
| 多模型支持 | ✅ 多提供商 | ❌ (仅 Claude) | ❌ (仅 OpenAI) | ✅ |
| 消息平台集成 | ✅ | ❌ | ❌ | ❌ |
| 子代理协同 | ✅ 跨工具 | ✅ (agent tool) | ✅ | ❌ |
| 开源 | ✅ | ❌ | ❌ | ❌ |

## 架构概要

```
Hermes Agent 核心架构：
┌──────────────────────────────────────────┐
│              TUI / Web 控制台              │
├──────────────────────────────────────────┤
│  技能引擎  │  记忆系统  │  任务调度器      │
├──────────────────────────────────────────┤
│  模型路由层 (Anthropic / OpenAI / 自定义)  │
├──────────────────────────────────────────┤
│  代码桥接 (Claude Code / Codex / Kimi...)  │
└──────────────────────────────────────────┘
```

## 相关页面

- [[hermes-setup-guide]] — 安装配置指南
- [[hermes-documentation-links]] — 官方文档与社区资源
- [[hermes-skills-system]] — 技能系统详解
