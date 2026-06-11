---
title: Hermes 安装与配置指南
tags: [hermes, setup, guide, ai-tools]
created: 2026-06-07
updated: 2026-06-07
---

# Hermes 安装与配置指南

> 适用版本：v0.14.0 (2026年5月)

## 系统要求

| 项目 | 要求 |
|------|------|
| OS | Linux / macOS / WSL2 / Android (Termux) |
| Python | 3.11+ |
| Git | 需安装 |
| 可选 | Ollama (本地向量搜索)、Node.js (Web 控制台) |

## 安装

### 方式一：一键脚本（推荐）

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.zshrc
hermes --version
```

### 方式二：PyPI（v0.14+ 轻量路径）

```bash
pip install hermes-agent
```

### 方式三：VPS 生产部署

```bash
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | sudo bash
```

自动安装：Hermes + Node.js + Caddy (TLS) + UFW + fail2ban + systemd

## 初始化

```bash
hermes setup
```

配置向导会引导你设置：
- 模型提供商
- API Key
- 默认模型
- 记忆系统
- 消息平台（可选）

或通过 Web 控制台：
```bash
hermes dashboard
# 浏览器访问 http://localhost:18789
```

## 模型提供商配置

### Anthropic (Claude)

```bash
hermes config set model.provider anthropic
hermes config set model.api_key YOUR_API_KEY
hermes config set model.default claude-sonnet-4-20250514
```

### OpenAI

```bash
hermes config set model.provider openai
hermes config set model.api_key YOUR_API_KEY
```

### 阿里云百炼（国内推荐）

```bash
hermes config set model.provider custom
hermes config set model.base_url https://dashscope.aliyuncs.com/apps/anthropic
hermes config set model.api_mode anthropic_messages
hermes config set model.api_key YOUR_API_KEY
hermes config set model.default qwen3.7-max
```

## Claude Code 协同配置

```bash
# 安装代码桥接插件
hermes plugins install https://github.com/xuyang-liu16/hermes-code-bridge --enable

# 启用子代理
hermes config set agent.subagent.enabled true
hermes config set agent.subagent.provider claude-code
```

然后在对话中使用：
```
/code-bridge Use Codex to review my current diff.
```

## 常用命令

```bash
hermes                  # 交互模式
hermes --tui            # TUI 模式（推荐）
hermes chat -q "..."    # 单次问答
hermes dashboard        # Web 控制台
hermes skills list      # 列出技能
hermes skills install   # 安装技能
hermes config set ...   # 修改配置
```

## 相关页面

- [[hermes-agent]] — Hermes Agent 概述
- [[hermes-skills-system]] — 技能系统详解
- [[hermes-documentation-links]] — 文档资源汇总
