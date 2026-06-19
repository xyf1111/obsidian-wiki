---
title: "Redis 08 - Cluster 集群"
date: 2026-06-13
tags:
  - redis
  - 缓存
  - 集群
aliases:
  - "Redis 08"
---

# Redis 08 — Redis Cluster 集群

## 为什么需要集群

| 场景 | 单节点问题 | 集群方案 |
|------|-----------|---------|
| 数据量 > 内存 | 放不下 | 数据分片 |
| 单点故障 | 宕机服务不可用 | 主从切换 |
| QPS 过高 | 单节点瓶颈 | 多节点分摊 |

## 数据分片

Redis Cluster 使用 **16384 个哈希槽**（hash slot）进行数据分片：

```bash
# CRC16(key) % 16384 → 分配到对应节点
HASH_SLOT = CRC16(key) & 16383
```

### 槽位分配

```
Node A: 0-5460
Node B: 5461-10922
Node C: 10923-16383

写 key="user:123":
CRC16("user:123") % 16384 = 4500
→ 路由到 Node A
```

### 添加/移除节点

```bash
# 添加节点（自动迁移部分槽位）
redis-cli --cluster add-node 新节点:6379 现有节点:6379

# 重新分片
redis-cli --cluster rebalance 任意节点:6379

# 移除节点
redis-cli --cluster del-node 节点:6379 节点ID
```

## 集群架构

```
        ┌─────────────┐
        │  Application │
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │  Redis       │
        │  Cluster     │
        │  (Smart Client)│
        └──────┬──────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───┴───┐ ┌───┴───┐ ┌───┴───┐
│Master1│ │Master2│ │Master3│
│0-5460 │ │5461-  │ │10923- │
│       │ │10922  │ │16383  │
└───┬───┘ └───┬───┘ └───┬───┘
    │          │          │
┌───┴───┐ ┌───┴───┐ ┌───┴───┐
│Slave1 │ │Slave2 │ │Slave3 │
└───────┘ └───────┘ └───────┘
```

### 最小集群

```bash
# 至少需要 3 个 Master 节点
# 高可用需要 + 3 个 Slave 节点

# 创建 6 节点集群（3主3从）
redis-cli --cluster create \
  192.168.1.10:7000 \
  192.168.1.10:7001 \
  192.168.1.10:7002 \
  192.168.1.11:7000 \
  192.168.1.11:7001 \
  192.168.1.11:7002 \
  --cluster-replicas 1
```

## 主从与故障转移

### 节点通信

```
Gossip 协议：
每隔 100ms Ping 随机 5 个节点
交换集群状态：节点存活、槽位信息、配置纪元
```

### 故障检测

```bash
# 节点标记 PFAIL（疑似下线）
# 半数以上 Master 标记 PFAIL → FAIL（确认下线）
# Slave 自动升级为 Master
cluster-node-timeout 15000  # 15s 无响应判定为宕机
```

### 故障转移

```
1. Master A 宕机
2. 其他节点检测到 PFAIL → FAIL
3. Master A 的 Slave 发起选举
4. 多数 Master 投票同意
5. Slave 成为新 Master
6. 槽位分配给新 Master
（整个过程 ~10-15s）
```

## 客户端访问

```go
import "github.com/go-redis/redis/v8"

// 连接 Redis Cluster
rdb := redis.NewClusterClient(&redis.ClusterOptions{
    Addrs: []string{
        "192.168.1.10:7000",
        "192.168.1.10:7001",
        "192.168.1.10:7002",
    },
    // 自动识别集群拓扑
    // 自动路由到正确的节点
    // 自动处理 MOVED/ASK 重定向
})

// 使用方式和单节点一致
rdb.Set(ctx, "key", "value", 0)
val, _ := rdb.Get(ctx, "key").Result()
```

## 集群限制

| 限制 | 说明 | 解决方案 |
|------|------|---------|
| **多键操作** | `MGET`/`MSET` 跨槽位报错 | 使用 hash tag |
| **事务** | `MULTI`/`EXEC` 跨节点不支持 | Hash Tag 让 key 在同一个槽 |
| **Pipeline** | Pipeline 跨节点会变慢 | 按节点分组 |
| **Lua 脚本** | 脚本涉及的 key 必须在同一个节点 | Hash Tag |
| **大 Key** | 无法分片 | 业务拆分 |

### Hash Tag

```go
// 普通 key：可能分布在不同节点
key1 = "user:123:profile"
key2 = "user:123:orders"

// 使用 Hash Tag：确保在同一个槽位
key1 = "{user:123}:profile"
key2 = "{user:123}:orders"

// Hash Tag 规则：只对 {} 内的内容计算 CRC16
// 所以 {user:123} 的哈希值相同
HASH_SLOT("user:123".CRC16())  // 相同！
```

## 哨兵 vs 集群

| 对比 | Sentinel（哨兵） | Cluster（集群） |
|------|-----------------|----------------|
| 数据分片 | ❌ | ✅ 16384 个槽 |
| 高可用 | ✅ 自动切换 | ✅ 自动切换 |
| 水平扩展 | ❌ 不能扩分片 | ✅ 在线扩容 |
| 客户端 | 连接哨兵获取 Master | 连接任意节点自动路由 |
| 最大节点 | 一般 3-5 个 | 理论 1000+ |
| 最小部署 | 1主1从+3哨兵 | 6节点（3主3从） |

## 参考资料

- [Redis Cluster 官方文档](https://redis.io/docs/reference/cluster-spec/)
- [Redis 集群教程](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)
- [go-redis Cluster 客户端](https://github.com/go-redis/redis)
- [Redis 集群设计](https://redis.io/docs/reference/cluster-spec/)
