---
title: "AI 智能客服 01 - Go 多渠道客服系统与 SSE 流式对话"
date: 2026-11-21
tags: [AI, 大模型, Golang, Gin, SSE, 项目实战]
source: "自有项目 ~/code/ai-customer-service"
---

# AI 智能客服 01 - Go 多渠道客服系统与 SSE 流式对话

> 用 Go 写一个能接多渠道的 AI 客服：网页、公众号、飞书共用同一套对话逻辑，大模型通过 OpenAI 兼容协议接入（DeepSeek / 通义 / 智谱 / Kimi 换 BaseURL 即可），回复走 SSE 逐字吐出来。这是 [[RAG 01 - 学习路线]] 和 [[AI 大模型 01 - 应用开发学习路线]] 的落地实践——Phase 1 只做流式对话，RAG 检索留给 Phase 2。

## 一、技术选型

| 层 | 选型 | 说明 |
|----|------|------|
| Web 框架 | Gin | 路由 + 中间件 |
| 大模型 | DeepSeek | OpenAI 兼容协议，换 BaseURL 即可切通义/智谱/Kimi |
| SDK | `sashabaranov/go-openai` | 用官方兼容客户端，不手写 SSE 解析 |
| 流式输出 | SSE | 浏览器 `fetch` + `ReadableStream` 逐段渲染 |
| 配置 | `joho/godotenv` | `.env` 读取，缺密钥直接启动失败 |
| 向量库 | Qdrant（Phase 2） | 尚未接入 |

## 二、目录结构与职责

```
ai-customer-service/
├── cmd/server/main.go          # 入口：加载配置 → 组装依赖 → 注册路由
├── internal/
│   ├── config/                 # 环境变量 → Config 结构体，必填项校验
│   ├── llm/                    # 大模型统一抽象
│   │   ├── model.go            #   Model 接口 + Message/ChatRequest
│   │   └── openai_compat.go    #   OpenAI 兼容实现（流式 Recv 循环）
│   ├── agent/agent.go          # 对话编排：拼 system prompt → 调模型
│   ├── channel/channel.go      # 渠道无关消息模型 + Channel 接口
│   ├── channel/web/handler.go  # 网页渠道：POST /api/chat，SSE 下发
│   └── middleware/ratelimit.go # 按 IP 滑动窗口限流（防刷额度）
└── web/index.html              # 单文件前端，无构建步骤
```

## 三、三个关键抽象

| 抽象 | 位置 | 解决什么问题 |
|------|------|--------------|
| `llm.Model` 接口 | `internal/llm/model.go` | 模型供应商可换（OpenAI 兼容 → 其他协议只需加实现），上层零改动 |
| `channel.Message` / `Channel` 接口 | `internal/channel/channel.go` | 渠道无关统一消息，网页/公众号/飞书共用同一套对话逻辑 |
| `agent.Agent` | `internal/agent/agent.go` | 编排层，RAG 检索、工具调用只在这一层插入 |

```go
// 流式核心：每收到增量就回调 onDelta，前端边收边渲染
type Model interface {
    StreamChat(ctx context.Context, req ChatRequest, onDelta func(delta string) error) (string, error)
}
```

## 四、请求链路

```
浏览器 index.html
  └─POST /api/chat {message}
      └─ 限流中间件（20 次/分钟/IP）→ 长度校验（≤2000 字）
          └─ web.Handler.Chat：写 SSE 头 → 建 history
              └─ agent.Chat：加 system prompt → model.StreamChat
                  └─ DeepSeek 流式返回 → onDelta 写 `data: {"delta":"…"}` → flush
                      └─ 结束写 `data: [DONE]`
```

**注意**：当前 `history` 只有本次 user 消息——服务端无状态，多轮上下文尚未实现（Phase 4）。

## 五、部署事实（阿里云 ECS）

| 项 | 值 |
|----|----|
| 部署目录 | `/opt/ai-customer-service/`（`acs` 二进制 + `web/` + `.env`） |
| 端口 | `:8081`（8080 已被商城后端占用） |
| 进程托管 | PM2，进程名 `ai-customer-service`，`pm2 save` 已固化 |
| 服务器环境 | 无 Go 工具链、无 nginx、2 核 1.8G 内存 |
| 访问方式 | 公网直连 8081（需安全组放行） |

```bash
# 本地交叉编译（服务器是 x86_64）
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -ldflags="-s -w" -o /tmp/acs ./cmd/server
# 上传 + 重启（直接 scp 覆盖运行中的二进制会报 Text file busy）
scp /tmp/acs root@host:/opt/ai-customer-service/acs.new
ssh root@host 'pm2 stop ai-customer-service; mv -f /opt/ai-customer-service/acs.new /opt/ai-customer-service/acs;
  pm2 start acs --name ai-customer-service --cwd /opt/ai-customer-service'
```

## 六、公网防滥用

网页渠道本身**没有鉴权**，公网可达即等于把大模型额度敞开，因此设两道闸门：

| 闸门 | 实现 | 默认 |
|------|------|------|
| 按 IP 限流 | `middleware.PerIP`，内存滑动窗口 + map 超 1 万条整体清空防膨胀 | 20 次/分钟 |
| 单条长度 | `handler.go` 常量 `maxMessageLen` | 2000 字 |

单机内存限流只防误伤，网关级限流/鉴权应放到 Nginx 或 API 网关层。

## 七、踩坑记录

| 现象 | 原因 | 处理 |
|------|------|------|
| `scp: Text file busy` | 二进制被 PM2 进程占用 | 先传 `acs.new`，停进程后 `mv -f` 覆盖 |
| `.env` 里改 `GIN_MODE=release` 不生效 | gin 的 mode 在包 init 时读取，早于 `godotenv.Load()` | main 里显式 `gin.SetMode(os.Getenv("GIN_MODE"))` |
| `ssh: kex_exchange_identification: Connection closed` | 短时间大量 SSH 连接被限流 | 用 `ControlMaster` 复用单连接，或间隔 20s 重试 |
| 外网连 8081 超时、服务器自连正常 | 阿里云安全组只放行了旧端口 | 控制台加规则，服务器本机 iptables 是空的 |
| PM2 日志涨到 6.7 G | 崩溃循环的服务每秒打印堆栈且从不轮转 | `pm2 delete` 重复进程 + `pm2 flush` + `pm2-logrotate` |

## 八、路线图

- [x] Phase 1 网页流式对话（已上线）
- [ ] Phase 2 RAG：文档导入 → 向量检索（Qdrant）→ 带引用回答
- [ ] Phase 3 微信公众号 / 飞书 / 钉钉适配器
- [ ] Phase 4 会话历史、多轮改写、工具调用、降级兜底

## 九、可迁移的工程经验

1. **先定接口再写实现**：`Model` / `Channel` 两个接口让「换模型」和「加渠道」变成新增文件，而不是改老代码。
2. **流式 = 回调**：底层 `StreamChat(onDelta)` + 上层 SSE 写出，协议细节收敛在一个函数里。
3. **无工具链的服务器就交叉编译**：`CGO_ENABLED=0` 出静态二进制，比在服务器装 Go 干净得多。
4. **公网接口先想额度**：限流和长度上限是上线前的最低成本保险。
