---
title: "Git 04 - Git 与 GitHub 学习路线"
date: 2026-07-19
tags: [Git, GitHub, devops]
source: "鱼皮·编程导航 / codefather"
---

# Git 04 - Git 与 GitHub 学习路线

> Git 版本控制与 GitHub 协作平台从入门到实战的完整学习路径。

## 概述

Git 是目前最主流的分布式版本控制系统，是团队协作开发不可或缺的工具。GitHub 是目前最主流的代码开源托管平台，提供代码管理、协作、CI/CD 等功能。

**Git vs GitHub：** Git 是工具（版本控制系统），GitHub 是平台（代码托管服务）。可以用 Git 向任何代码托管平台（GitHub/GitLab/Gitee）提交代码。

## 整体学习建议

1. **不要背命令** — Git 只是工具，先了解常用命令作用，随用随查
2. **多手敲命令** — 初期建议手敲命令而非全依赖可视化工具
3. **实践是关键** — 用的越多越熟练
4. **善用 AI 辅助** — 遇到冲突等问题可问 AI 工具
5. **学 GitHub 首选官方文档** — 全面权威准确、支持中文，入门跟着官方文档「入门指南」学这 4 个重点：在 GitHub 上探索项目、用 Git 管理项目代码、导入项目、管理仓库

**GitHub 亮点功能：** 仓库数据可视化（提交年图）— 查看仓库基本信息与变更历史

## 零、无痛上手

Git 可随用随学，先安装 Git，用可视化工具（IDEA/VS Code 自带功能、GitHub Desktop）完成拉取和提交操作即可入门。

**学习资源：**
- [Git 官方下载](https://git-scm.com/downloads)
- 可视化工具：GitKraken、Sourcetree、TortoiseGit、GitHub Desktop

## 一、Git 基础

### 目标
了解 Git 基本概念和常用命令，可用 Git 命令管理和提交项目代码。

### 知识点
**基本概念：** 什么是 Git（版本控制系统）、工作区/暂存区/本地版本库/远程仓库、文件状态、版本、HEAD、分支

**基本操作：** git init、clone、add、commit、push、fetch、pull、status、log

**分支操作：** 创建/查看/切换/删除分支、git merge

### 学习资源
- [猴子都能懂的 Git 入门](https://backlog.com/git-tutorial/cn/) — 图文并茂
- [廖雪峰的 Git 教程](https://www.liaoxuefeng.com/wiki/896043488029600/)
- [一节课入门 Git（18 分钟）](https://www.bilibili.com/video/BV1s3411g7PS)
- [Learning Git Branching 在线游戏](https://learngitbranching.js.org/?locale=zh_CN) — 理解分支
- [Git 命令大全](https://backlog.com/git-tutorial/cn/reference/)

## 二、GitHub 基础

### 目标
熟悉 GitHub 基本操作，能使用 GitHub 管理代码；了解开源，能向开源项目提交代码。

### 知识点
**基本概念：** 仓库、分支、README、Star、Follow、账户类型

**必备操作：** 搜索代码、创建仓库（公开/私有）、Fork、Watch、上传代码、修改个人信息

**协作流程：** GitHub Flow — Fork → 创建分支 → 修改 → Pull Request → Code Review → Merge → 删除分支

**拓展：** GitHub Issues、贡献代码流程

### 学习资源
- [GitHub 官方 Hello World 入门](https://docs.github.com/cn/get-started/quickstart/hello-world)
- [GitHub 漫游指南](https://github.phodal.com/)
- [教你给开源项目贡献代码](https://github.com/firstcontributions/first-contributions)
- [GitHub 备忘清单](https://training.github.com/downloads/zh_CN/github-git-cheat-sheet/)

## 三、Git 进阶

企业开发中项目更复杂、协作人员更多，需掌握冲突解决和团队协作流程。

### 知识点
**高级操作：** tag、checkout、stash、clean、rebase、reset、revert、diff、blame、reflog、cherry-pick

**协作技能：** SSH 配置、Gitignore、解决冲突（重中之重）、Git hooks（pre-commit）

**工作流：** Git Flow、Monorepo

### 学习资源
- [Git 官方文档](https://git-scm.com/book/zh/v2) — 强烈推荐
- [Git Flow 演示学习](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

## 四、GitHub 进阶

GitHub 集成了丰富的 DevOps 功能。

### 知识点
**开发：** SSH 配置、GitHub Codespaces、GitHub Apps

**协作：** GitHub Discussions、Pull Requests

**项目管理：** Organizations、Issues、Projects、Insights、开源协议

**DevOps：** GitHub Pages、GitHub Actions、Webhook、Packages

### 学习资源
- [GitHub 官方文档](https://docs.github.com/cn)
- [GitHub 秘籍](https://snowdream86.gitbooks.io/github-cheat-sheet/content/zh/)
- [GitHub 隐藏技巧](https://www.bilibili.com/video/BV1q54y1f7h6)

## 五、开源贡献实战建议

> 案例来源：《14 岁，3 次给我的项目贡献代码》— 一位 14 岁少年给开源项目提 PR 的故事。

### 开源贡献的价值

很多开源项目的参与门槛很低：先在本地把项目跑通，再阅读一些代码，说不定就能发现一些小 Bug。**哪怕只改正一个小 Bug，也是有价值的贡献，可以写进简历。**

更重要的是，给开源项目贡献代码，等于获得一次免费的大佬 1 对 1 学习机会——通过 Pull Request 与项目作者交流，即使代码最终没有被合并，过程中的反馈与指点也能带来很大收获。

### 提高 PR 被合并概率的 3 个方法

**1）遵循原项目的写法**

格式、命名、代码风格都要与原项目保持一致，不要做无意义的改动。比如把原本能运行的代码「换个写法」，本质上毫无变化——如果提交里有 100 个这种「没必要改动的改动」，维护者看代码时会直接崩溃。无论开源还是公司内协作，与团队保持统一规范都至关重要。

**2）提供改动说明 + 运行截图**

在代码审查页面，维护者看不到改动后的代码能否运行、运行效果如何。因此在 PR 描述中附上代码改动说明，并贴出效果截图或测试报告，能让维护者轻松判断，大幅提高合并概率。

**3）尽量减少改动，每次只改一个点**

每次 PR 只针对一个功能、Bug 或模块。单次合并的代码越多，对项目的影响可能越大，维护者合并前就越谨慎；改动越小，读代码的人越舒服，也就越容易合并。

### 反面案例：先了解项目的技术栈与架构

一位贡献者想给 sql-mother 增加「SQL 临时存储」功能（防止关闭浏览器后丢失），直接手写了一段操作 LocalStorage 的代码，命名和格式都很规范，但最终没有被合并。

原因：项目已使用 `pinia` 做前端状态管理，并引入 `pinia-plugin-persistedstate` 插件实现 LocalStorage 的自动加载与同步。直接手写 LocalStorage 与项目现有架构重复且不一致。

**启示：** 动手写代码前，先了解项目的技术栈、架构和已有实现，遵循项目既有的方案来写，而不是另起炉灶。

## 面试考点

Git 在面试中占比很低，重点是会用。如果被问到，一般考察团队协作中的 Git 使用经验和冲突解决方法。

### 经典面试题
**理论题：** Git 原理和工作流程、fetch vs pull、rebase vs merge、Git Flow、暂存区作用

**实践题：** 团队协作流程、Gitignore、冲突产生与解决、代码恢复与撤销、分支管理策略

### 面试题库
- [Git 基础面试题 - 面试鸭](https://www.mianshiya.com/bank/1815649049726590977)
- [Git 进阶面试题 - 面试鸭](https://www.mianshiya.com/bank/1815649098609254402)
- [Git 操作面试题 - 面试鸭](https://www.mianshiya.com/bank/1815649161437683714)

> 来源：鱼皮·编程导航 / codefather
