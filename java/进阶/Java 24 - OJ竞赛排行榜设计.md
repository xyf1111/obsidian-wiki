---
title: Java 进阶 - OJ 竞赛排行榜设计
date: 2026-07-23
tags:
  - java
  - 在线判题
  - 排行榜
  - 系统设计
  - 进阶
source: "鱼皮·编程导航 / codefather"
---

# OJ 竞赛排行榜设计

> OJ（Online Judge）竞赛排行榜的设计思路与 Java 代码实现，包括数据库表设计和统计逻辑。

## 设计要点

1. **尽可能实时**：异步处理提交，保证用户体验
2. **保留最优成绩**：维护每个用户对每道题的最优答题情况
3. **排名规则**：总分 → 总耗时 → 总空间（按优先级降序/升序）
4. **可见性控制**：竞赛前不可见；竞赛中动态变化；竞赛后冻结

## 数据库设计

### 表结构概览

| 表名 | 说明 |
|------|------|
| `game` | 竞赛基本信息 |
| `user_game` | 用户-竞赛关联 |
| `game_question` | 竞赛-题目关联（含满分） |
| `game_rank` | 竞赛排名（总分、总耗时、总空间、每题详情 JSON） |

### game_rank 表

```sql
CREATE TABLE `game_rank` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  `gameId` bigint DEFAULT NULL COMMENT '竞赛id',
  `userId` bigint DEFAULT NULL COMMENT '用户id',
  `userName` varchar(255) DEFAULT NULL COMMENT '用户昵称',
  `totalMemory` int DEFAULT NULL COMMENT '总空间(kb)',
  `totalTime` int DEFAULT NULL COMMENT '总用时(ms)',
  `totalScore` int DEFAULT NULL COMMENT '总得分',
  `gameDetail` text COMMENT '竞赛详情(JSON)',
  `createTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updateTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `isDelete` tinyint NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`),
  INDEX `idx_gameId`(`gameId`),
  INDEX `idx_userId`(`userId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

`gameDetail` 字段以 JSON 存储每题最优成绩，结构：
```json
{
  "gameId": 1,
  "userId": 42,
  "submitDetail": {
    "10001": { "id": 10001, "name": "两数之和", "score": 100, "timeCost": 50, "memoryCost": 1024 },
    "10002": { "id": 10002, "name": "反转链表", "score": 80, "timeCost": 120, "memoryCost": 2048 }
  }
}
```

## 核心实体类

**GameRank** — 数据库映射实体，包含总分/总耗时/总空间和 gameDetail 详情。

**GameDetail** — 竞赛详情对象，包含 gameId、userId 和 submitDetail（Map<题目ID, GameDetailUnit>）。

**GameDetailUnit** — 每题最优成绩。提供 `isBetter()` 方法按「分数高→用时少→空间少」比较优劣。

**GameRankDetail** — 排行榜展示 VO，包含排名、用户信息和每题详情列表。

## 核心流程

### 1. 答题提交流程

用户提交题目后：
1. 校验竞赛是否在有效时间内
2. 调用判题服务（异步）
3. 异步更新排行榜信息：
   - 用户无排名记录 → 新建 GameRank，插入当前成绩
   - 已有记录但 gameDetail 为空 → 直接填入
   - 已有记录且该题已存在 → 若新成绩优于旧成绩则替换（增量更新总分/耗时/空间）

### 2. 排行榜统计流程

1. 获取指定竞赛的全部题目 ID
2. 获取参赛的全部用户 ID
3. 遍历每个用户，查询其 GameRank 记录
4. 无记录 → 创建空记录并初始化各题为 0 分
5. 有记录 → 解析 gameDetail JSON，统计总分/总耗时/总空间
6. 排序：`总分降序 → 总耗时升序 → 总空间升序`
7. 分配名次

```java
// 排序逻辑
List<GameRankDetail> sorted = rankDetails.stream()
    .sorted(Comparator.comparing(GameRankDetail::getTotalScore).reversed()
        .thenComparing(GameRankDetail::getTotalTime)
        .thenComparing(GameRankDetail::getTotalMemory))
    .collect(Collectors.toList());
```

## 关键设计决策

- **空间换时间**：在 game_rank 表中冗余存储总分/总耗时/总空间，避免每次查询时动态计算
- **异步更新**：使用 `CompletableFuture.runAsync()` 异步更新排行榜，不阻塞答题主流程
- **JSON 存储**：每题详情存为 JSON，灵活支持不同题型的附加字段，避免宽表设计

> 来源：鱼皮·编程导航 / codefather
