---
title: "Elasticsearch 01 - 基础概念与架构"
date: 2024-01-01
tags: [Elasticsearch]
---

# Elasticsearch 01 - 基础概念与架构

## 什么是 Elasticsearch

Elasticsearch 是一个基于 **Apache Lucene** 的分布式、RESTful 搜索引擎。由 Shay Banon 于 2010 年开源，是 ELK（Elasticsearch + Logstash + Kibana）生态的核心。

### 核心能力

- **全文搜索** — 倒排索引、分词、相关性评分（BM25）
- **分析聚合** — 指标聚合、桶聚合、管道聚合
- **分布式** — 分片机制、自动故障转移
- **近实时（NRT）** — 写入后约 1s 可查询
- **REST API** — 全 HTTP 接口，客户端库丰富

## 核心概念

### 与关系型数据库的类比

| Elasticsearch | 关系型数据库 |
|--------------|-------------|
| Index（索引） | Database（数据库） |
| Type（类型，已废弃） | Table（表） |
| Document（文档） | Row（行） |
| Field（字段） | Column（列） |
| Mapping（映射） | Schema（表结构） |
| Shard（分片） | 分区 |

### Index（索引）

索引是文档的集合，类似数据库。命名规则：
- 全小写
- 不能包含 `\`, `/`, `*`, `?`, `"`, `<`, `>`, `|`, `,`, `#`
- 不能以 `-`, `_`, `+` 开头

### Document（文档）

JSON 格式的数据单元，是搜索的基本单位：

```json
{
  "_index": "products",
  "_id": "1",
  "_version": 1,
  "_source": {
    "title": "iPhone 15 Pro",
    "price": 7999,
    "stock": 100,
    "tags": ["手机", "苹果"],
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

### Mapping（映射）

定义字段类型和分析器：

```json
PUT /products
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "price":  { "type": "double" },
      "stock":  { "type": "integer" },
      "tags":   { "type": "keyword" },
      "created_at": { "type": "date" }
    }
  }
}
```

## 集群架构

```
┌──────────────────────────────────────────────┐
│              Elasticsearch Cluster           │
│                                              │
│  Node 1 (Master)     Node 2 (Data)           │
│  ┌──────────────┐    ┌──────────────┐        │
│  │ P0 (Primary) │    │ P0 (Replica) │        │
│  │ P1 (Replica) │    │ P1 (Primary) │        │
│  └──────────────┘    └──────────────┘        │
│                                              │
│  Node 3 (Ingest)     Node 4 (Coordinating)   │
│  ┌──────────────┐    ┌──────────────┐        │
│  │ 数据预处理   │    │ 请求路由     │        │
│  └──────────────┘    └──────────────┘        │
└──────────────────────────────────────────────┘
```

### 节点类型

| 类型 | 配置 | 职责 |
|------|------|------|
| Master | `node.roles: [master]` | 集群管理、元数据 |
| Data | `node.roles: [data]` | 数据存储、CRUD、搜索 |
| Ingest | `node.roles: [ingest]` | 数据预处理 Pipeline |
| Coordinating | `node.roles: []` | 负载均衡、请求路由 |
| Machine Learning | `node.roles: [ml]` | 异常检测（白金版） |

## REST API 快速上手

```bash
# 检查集群健康
GET /_cluster/health

# 创建索引
PUT /products
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}

# 写入文档
POST /products/_doc
{
  "title": "MacBook Pro",
  "price": 14999
}

# 指定 ID 写入
PUT /products/_doc/1
{
  "title": "iPhone 15",
  "price": 7999
}

# 搜索
GET /products/_search
{
  "query": {
    "match": { "title": "手机" }
  }
}
```

## vs Solr

| 特性 | Elasticsearch | Solr |
|------|-------------|------|
| 分布式 | 原生（分片自动） | ZooKeeper 管理 |
| 实时性 | 近实时（默认 1s） | 近实时 |
| 生态 | ELK（Kibana/Logstash/Beats） | SolrCloud + Banana |
| 学习曲线 | 中等 | 较高 |
| 社区活跃度 | 极高 | 中等 |
| 场景偏好 | 日志分析、通用搜索 | 企业搜索、传统项目 |
