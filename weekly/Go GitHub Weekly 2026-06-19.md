---
title: "Go GitHub Weekly 2026-06-19"
date: 2026-06-19
tags: [weekly, golang]
---

# Go GitHub Weekly — 2026-06-19

## Go 官方

- [Go](https://go.dev) — Go 1.25 进入 Beta 阶段，泛型类型别名（type alias with generics）正式落地，编译器内存占用降低约 15%
- [Go Toolchain](https://go.dev/doc/toolchain) — Go Telemetry 默认开启进入生产化阶段，`go tool` 支持多版本工具链自动切换，forward compatibility 持续完善
- [Go 标准库](https://go.dev/pkg) — `iter` 包进一步扩展，`slices`/`maps` 包新增多个泛型辅助函数；`testing/synctest` 进入实验阶段

## Web 框架

- [Gin](https://github.com/gin-gonic/gin) — v1.11.0 发布，优化路由树匹配性能，新增 `net/http` 中间件原生适配兼容层
- [Fiber](https://github.com/gofiber/fiber) — v3.5.0 发布，基于 fasthttp 的请求/响应池优化，支持 `ListenTLS` 和 `ListenMutualTLS` 优雅关闭
- [Echo](https://github.com/labstack/echo) — v4.14.0 发布，改进 binder 错误处理，支持 OpenAPI 3.1 注解生成
- [Chi](https://github.com/go-chi/chi) — v5.2.0 发布，中间件链路由性能优化，新增 Chi-Swagger 集成支持
- [HTMX](https://htmx.org) — Go 生态中 `temir-htmx` 和 `go-htmx` 工具链成熟，配合 Templ 模板引擎的 SSR 方案日渐流行

## 数据库 / ORM

- [GORM](https://gorm.io) — v1.26.0 发布，新增原生 `sql.DB` 连接复用优化，改进复合主键关联查询，支持 `FindToMap` 批量映射
- [Ent](https://entgo.io) — v0.15.0 发布，新增 Paging 游标分页扩展，改进 Schema 迁移的差量生成能力
- [Bun](https://bun.uptrace.dev) — v1.4.0 发布，新增原生 SQL 批量插入的 `ValuesQueryBuilder`，增强 pgx 驱动兼容层
- [pgx](https://github.com/jackc/pgx) — v5.7.0 发布，新增 pgxpool 连接池健康检查回调，支持 SCRAM-SHA-256 认证扩展参数
- [MongoDB Go Driver](https://github.com/mongodb/mongo-go-driver) — v1.18.0 发布，支持 MongoDB 8.0 Queryable Encryption，聚合管道性能优化

## 微服务 / 工具链

- [Fx](https://uber-go.github.io/fx) — v1.24.0 发布，新增 `fx.Recover` 装饰器用于模块级别 panic 恢复，DI 图懒加载性能优化
- [Wire](https://github.com/google/wire) — v0.6.0 发布，支持泛型注入，减少代码生成复杂度，新增 `wire.PanicOnError` 提供者选项
- [Cobra](https://github.com/spf13/cobra) — v1.9.0 发布，改进 shell completion 生成速度，新增 `Command.PrintErrln` 便捷方法
- [Viper](https://github.com/spf13/viper) — v1.20.0 发布，支持 `context.Context` 贯穿配置解析链路，新增 `OnConfigChange` 增强事件回调
- [Air](https://github.com/air-verse/air) — v1.62.0 发布，改进文件监听性能（基于 fsnotify v2），支持 `.air.toml` 多配置预设
- [Task](https://taskfile.dev) — v3.40.0 发布，新增 `requires` 任务前置条件检查，支持 `includes` 嵌套 Taskfile 继承

## 可观测性

- [OpenTelemetry Go](https://opentelemetry.io/docs/languages/go/) — v1.35.0/v0.57.0 发布，新增 `log/slog` 原生集成，`otlploggrpc` 和 `otlploghttp` 导出器稳定
- [slog](https://go.dev/blog/slog) — Go 1.25 中 `log/slog` 新增 `LevelVar.Level()` 原子读取，`slog.Handler` 接口新增 `WithAttrs` 批量优化路径
- [Prometheus](https://github.com/prometheus/client_golang) — v1.21.0 发布，Native Histogram 正式 GA，promhttp 支持 OpenMetrics 格式，减少序列化开销
- [Pyroscope](https://github.com/grafana/pyroscope) — Grafana Pyroscope Go SDK v0.6.0 发布，基于 eBPF 的持续性能分析（Continuous Profiling）新增 Goroutine 栈采样标签支持

## 其他有趣项目

- [Bubble Tea](https://github.com/charmbracelet/bubbletea) — v1.3.0 发布，支持 `tea.WindowSize` 实时响应，改进 Mouse 事件处理，新增 `tea.ClearScrollArea` API
- [Bubbles](https://github.com/charmbracelet/bubbles) — 新增 `table` 组件（支持可排序列），`spinner` 新增多种动画风格
- [Lip Gloss](https://github.com/charmbracelet/lipgloss) — v1.2.0 发布，新增 256 色和 TrueColor 自适应检测，样式继承优化
- [Wazero](https://github.com/tetratelabs/wazero) — v1.9.0 发布，WASI 0.2 支持稳定，AssemblyScript 兼容层改进，运行时实例化速度提升约 20%
- [TinyGo](https://tinygo.org) — v0.36.0 发布，新增对 `context.Context` 的完整实现支持，ARM Cortex-M85 和 RISC-V P 扩展板级支持
- [GopherJS](https://github.com/gopherjs/gopherjs) — 持续推进 WASM 后端，`syscall/js` 兼容性改进，支持 Go 1.25 新特性编译到浏览器环境
- [GoReleaser](https://goreleaser.com) — v2.5.0 发布，新增 GitHub Actions 原生工作流生成器，支持 WASM 二进制的交叉编译打包
