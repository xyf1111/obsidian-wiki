---
title: "Java 47 - Java 学习资源与项目汇总"
date: 2026-09-16
tags: [Java, 学习资源, 开源项目, 源码分析]
source: "鱼皮·编程导航 / codefather"
---

# Java 47 - Java 学习资源与项目汇总

> GitHub 上值得跟着学的 Java 文档教程与实战项目，按「基础 → 类库框架 → 系统设计 → 源码」四类整理——用文档学习的好处是可以自己把控节奏、方便记录笔记。

## 核心要点

### 基础

| 项目 | 说明 |
| --- | --- |
| **On Java 8** | 《On Java 8》中文版，Java 8（当前主流版本）的在线学习手册，支持文档内搜索，适合新手入门 Java。原为开源项目，因出版纸质书目前应已不再维护 |
| **JavaGuide** | 全网知名的 Java 学习 + 面试指南：不仅全面讲解 Java 基础知识，还覆盖 Java 后端工程师必备技能（MySQL、Redis、系统设计等），既是教程也是完整的技术栈知识点总结 |
| **LearningNotes** | 一位 Java 学习者的笔记，包含 Java 基础、框架、Android 开发、设计模式、数据结构与算法、网络等知识体系，甚至还有自己的面试经历，非常适合作为「如何做笔记」的参考 |
| **java-learning** | 一份 Java 学习笔记，博客讲解 + 源码实例，包含 Java SE 与 Java Web 知识点；内容偏基础特性与编程细节总结，适合已了解 Java 基础语法、想巩固并深入学习的同学 |

### 类库框架

| 项目 | 说明 |
| --- | --- |
| **SpringBoot Guide** | JavaGuide 作者 Guide 哥的又一指南项目，专注 SpringBoot 教程与知识总结，并整理实战项目练手，适合从 0 到 1 学习 SpringBoot（路线见 [[Java 44 - Spring Boot 学习路线]]） |
| **springcloud-learning** | 涵盖大部分核心组件的 Spring Cloud 教程，包括 Spring Cloud Alibaba 与分布式事务 Seata，基于 Spring Cloud Greenwich 与 Spring Boot 2.1.7；22 篇文章 + 32 个 Demo，覆盖多数应用场景，适合能开发单体应用、想学分布式与微服务开发的同学进阶（路线见 [[Java 45 - Spring Cloud 微服务学习路线]]） |
| **spring-security-jwt-guide** | 同样是 Guide 哥的项目，以「文档 + 源码」的方式带读者从零入门 Spring Security 模块（含 JWT 认证） |
| **guava-study** | 针对 Google 知名开源类库 Guava 的学习项目：Guava 包含大量高质量 API，可让 Java 代码更优雅简洁，本项目帮助用好 Guava、提升开发效率 |
| **RxJavaLearningMaterial** | RxJava 本质是异步操作库，用极简洁的逻辑处理繁琐复杂的异步任务，深受 Android 开发者喜爱；该项目是详细的 RxJava 学习攻略与指南，从入门、原理到实战讲解透彻 |

### 系统设计

| 项目 | 说明 |
| --- | --- |
| **mall-learning** | 一套电商系统实战学习教程，包含架构、业务、技术要点的全方位解析；技术栈覆盖 SpringBoot、MyBatis、Elasticsearch、RabbitMQ、Redis、MongoDB、MySQL，并采用 Docker 容器化部署——技术广度与深度都值得跟做一遍 |
| **miaosha** | 秒杀系统一直是开发难点，本项目是秒杀系统的设计与实现，帮助学习其中的关键设计、开拓思维 |

系统设计通用方法论可参考 [[概念 27 - 高并发系统设计实战]]。

### 源码

| 项目 | 说明 |
| --- | --- |
| **JavaSourceCodeLearning** | Java 流行框架源码分析项目，目前包含 Spring、SpringBoot、SpringAOP、SpringSecurity、SpringSecurity OAuth2、JDK、Dubbo 等源码分析，讲解深入透彻，适合通过学习框架底层源码提升水平 |
| **LearningJDK** | 专注 JDK 源码的阅读笔记，已经阅读了几百个 JDK 类，适合想深入了解 JDK 的同学 |

### 使用建议

- **按目标选型**：入门补基础 → On Java 8 / java-learning；面试突击与知识体系梳理 → JavaGuide；框架上手 → SpringBoot Guide 与 spring-security-jwt-guide；进阶分布式 → springcloud-learning；练手项目 → mall-learning、miaosha
- **文档式学习适合自己控节奏**，建议边读边记笔记（可参考 LearningNotes 的笔记组织方式），而不是只看不写
- 阅读源码类项目（JavaSourceCodeLearning、LearningJDK）建议放在有一定框架使用经验之后，先会用再究其原理
- 更多通用学习资源与网站见 [[工具 34 - 编程学习网站与资源推荐]]，整体 Java 学习路线见 [[Java 01 - 学习路线]]、学习方法见 [[Java 46 - Java 学习方法（6 年经验）]]

> 来源：鱼皮·编程导航 / codefather
