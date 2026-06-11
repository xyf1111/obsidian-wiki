---
title: Hermes 技能系统
tags: [hermes, skills, workflow, agent]
created: 2026-06-07
updated: 2026-06-07
---

# Hermes 技能系统

技能（Skills）是 Hermes Agent 最核心的一等公民功能，用于定义可复用的 AI 工作流。概念上类似 Claude Code 的斜杠命令，但更加独立、可共享、可组合。

## 技能文件结构

```
技能目录/
├── SKILL.md        # 核心指令文件（必需）
├── scripts/        # 辅助脚本
├── references/     # 参考资料
└── templates/      # 输出模板
```

## 常用命令

```bash
# 列出已安装技能
hermes skills list

# 安装官方技能
hermes skills install official/research/duckduckgo-search

# 安装社区技能（URL）
hermes skills install https://raw.githubusercontent.com/user/repo/main/skills/skill-name/SKILL.md --name skill-name

# 卸载技能
hermes skills uninstall skill-name
```

## 与 Claude Code Skills 的对比

| 特性 | Hermes Skills | Claude Code Skills |
|------|---------------|-------------------|
| 独立性 | 独立目录、可 Git 管理 | 嵌入 `.claude/skills/` |
| 共享机制 | URL 安装 / 社区市场 | 手动复制 |
| 自动化管理 | Curator 自动评估加固 | 无 |
| 模板系统 | 内置 | 无 |
| 平台暴露 | 可暴露给 Telegram/Discord | 仅 CLI |

## Curator 系统

v0.14 引入的 Curator 可以：
- 自动评估代理创建的技能质量
- 加固技能指令（使其更鲁棒）
- 归档成熟技能到库

## 相关页面

- [[hermes-agent]] — Hermes 概述
- [[hermes-setup-guide]] — 安装指南
- [[hermes-documentation-links]] — 更多资源
