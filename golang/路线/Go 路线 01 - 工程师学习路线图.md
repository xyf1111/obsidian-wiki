---
title: "Go 工程师学习路线图"
date: 2026-06-11
tags:
  - golang
  - 学习路线
  - roadmap
source: ""
aliases:
  - "Go 学习路线"
  - "Go 工程师成长路径"
---

# Go 工程师学习路线图

> 结合你的知识库（2021年基础 + 2026年补充），以下是 Go 后端工程师的完整学习路径。

---

## 第一阶段：Go 基础（你已完成 ✅）

你的 `Go Study 01-18` 覆盖了这部分，包括：
- 变量、控制流、函数
- 切片、map、结构体
- 接口、错误处理
- 并发编程（goroutine、channel、select）
- 文件操作、JSON、测试

## 第二阶段：Go 核心原理（你已完成 ✅）

你的 `Go Hash`、`Go Slice`、`GC`、`Context`、`GMP 模型` 覆盖了：
- 数据结构底层实现
- 垃圾回收机制
- 并发调度模型
- Context 原理

## 第三阶段：现代 Go（本次补充 ✅）

### 3.1 泛型 (Go Generics)
📄 [[references/blog/golang/Go 泛型|Go 泛型笔记]]
- 类型参数、类型约束、近似元素 `~`
- 泛型集合、泛型函数
- 标准库泛型包（slices / maps / cmp）

### 3.2 Go 1.21 ~ 1.24 新特性
📄 [[references/blog/golang/Go 新特性 1.19-1.24|Go 新特性笔记]]
- `slog` 结构化日志
- `slices` / `maps` / `cmp` 包
- 循环变量语义修正
- 增强 `http.ServeMux`（路径参数）
- 迭代器 (Iterator)
- PGO 性能优化

### 3.3 现代性能调优工具链
📄 [[references/blog/golang/Go性能调优|Go性能调优笔记（已更新）]]
- `go tool pprof -http` Web UI
- `go tool trace`
- PGO (Profile-Guided Optimization)
- `runtime/metrics` 可观测性
- 软内存限制（GOMEMLIMIT）

---

## 第四阶段：Web 开发进阶

### 4.1 HTTP 框架
| 框架 | 特点 | 学习建议 |
|------|------|---------|
| **标准库** `net/http` | Go 1.22 增强后够用 | 优先掌握 |
| **Gin** | 轻量、高性能、生态好 | 企业常用 |
| **Echo** | 性能更好，API 简洁 | 值得了解 |
| **Fiber** | 类 Express.js 风格 | 可选 |

### 4.2 数据库
- **MySQL**：📄 你的笔记已覆盖（索引、调优、面试）
- **Redis**：📄 你的笔记已覆盖
- **Go ORM**：推荐学习 **GORM**（最流行的 Go ORM）
- **Go migrate**：数据库迁移工具 `golang-migrate/migrate`

### 4.3 中间件
- **RabbitMQ**：📄 你的笔记已覆盖
- **Kafka**：Go 生态推荐 `segmentio/kafka-go` 或 `confluent-kafka-go`
- **gRPC**：📄 你有 `Grpc01` 笔记，可深入学习

---

## 第五阶段：工程实践

### 5.1 项目结构
```bash
# 标准 Go 项目布局
myproject/
├── cmd/           # 入口
│   └── server/
│       └── main.go
├── internal/      # 私有代码
│   ├── handler/
│   ├── service/
│   ├── repository/
│   └── model/
├── pkg/           # 可导出的公共库
├── api/           # API 定义（proto / OpenAPI）
├── config/        # 配置
├── migrations/    # 数据库迁移
└── go.mod
```

### 5.2 测试金字塔
```
     E2E 测试
    ┌───────┐
    │  少量  │  ← 用 testcontainers-go
   ┌┴───────┴┐
   │集成测试  │  ← 用 dockertest / 真实依赖
  ┌┴─────────┴┐
  │ 单元测试   │  ← 用 mock / 依赖注入
  └───────────┘
```

你的笔记已有 Go 测试系列（单元测试、网络测试、MySQL/Redis 测试），可结合 [testcontainers-go](https://github.com/testcontainers/testcontainers-go) 做集成测试。

### 5.3 DI / 依赖注入
- 推荐 **wire**（Google 编译时 DI）：[github.com/google/wire](https://github.com/google/wire)
- 或 **fx**（Uber 运行时 DI）：[github.com/uber-go/fx](https://github.com/uber-go/fx)

---

## 第六阶段：进阶方向

### 6.1 微服务
- **框架**：go-kit、go-micro、**Kratos**（B站出品，中文友好）
- **Service Mesh**：了解 Istio / Linkerd
- **API 网关**：Kong、APISIX

### 6.2 云原生
- **Docker**：📄 你的 `Go Docker` 笔记
- **Kubernetes**：Client-Go、Operator 模式
- **可观测性**：OpenTelemetry（trace + metric + log）

### 6.3 系统编程
- **网络编程**：epoll、kqueue、TCP/IP 协议栈
- **高性能**：[fasthttp](https://github.com/valyala/fasthttp)、**bytebufferpool**、**ants**（协程池）
- **内存管理**：`sync.Pool`、`arena`（已移除）、对象池模式

---

## 推荐开源项目学习

| 项目 | 学习价值 | Go 特性 |
|------|---------|---------|
| [gin](https://github.com/gin-gonic/gin) | HTTP 框架设计 | 中间件模式、路由树 |
| [cobra](https://github.com/spf13/cobra) | CLI 工具开发 | CLI 框架设计 |
| [grpc-go](https://github.com/grpc/grpc-go) | RPC 实现 | 流控制、拦截器 |
| [etcd](https://github.com/etcd-io/etcd) | 分布式 KV 存储 | Raft 协议、高可用 |
| [minio](https://github.com/minio/minio) | 对象存储 | S3 兼容、高性能 IO |

---

## 学习建议

1. **现有知识库优势**：你的 2021 年笔记虽然时间早但基础扎实，覆盖了 Go 核心概念
2. **当前重点**：先消化本次补充的「泛型 + 新特性 + 现代性能调优」
3. **下一步项目实践**：用泛型和 slog 重构一个旧项目
4. **持续更新**：我会设置定时任务，每周从 GitHub 获取 Go 热门学习资源

---

## 相关笔记

- [[references/blog/golang/Go 泛型|Go 泛型]]
- [[references/blog/golang/Go 新特性 1.19-1.24|Go 新特性]]
- [[references/blog/golang/Go性能调优|Go 性能调优]]
- [[references/CC-Blog-Go语言核心原理|Go 语言核心原理]]
- [[references/CC-Blog-消息队列与中间件|消息队列与中间件]]
