---
title: "网络 03 - TCP三次握手与四次挥手"
date: 2024-01-01
tags: [网络]
---

# 网络 03 - TCP 三次握手与四次挥手

## 三次握手（建立连接）

```
客户端                         服务端
  │                               │
  ├── SYN=1, seq=x ──────────────►│  1. SYN（客户端请求连接）
  │                               │
  │◄── SYN=1, ACK=1, seq=y, ack=x+1──┤  2. SYN+ACK（服务端确认）
  │                               │
  ├── ACK=1, seq=x+1, ack=y+1 ──►│  3. ACK（客户端确认建立）
  │                               │
  │◄══════ 连接建立，传输数据 ════►│
```

### 为什么是三次而不是两次？

- **防止已过期的连接请求到达服务端**：如果客户端第一个 SYN 在网络中滞留，重传后又收到 ACK，三次握手让服务端等待客户端的最后一次 ACK，避免创建空的连接
- **同步初始序列号**：双方都需要确认对方的 ISN（Initial Sequence Number）

### 为什么不是四次？

三次已经足够完成双方 ISN 的确认，四次多余。

## 四次挥手（断开连接）

```
客户端                         服务端
  │                               │
  ├── FIN=1, seq=u ─────────────►│  1. FIN（客户端关闭写）
  │                               │
  │◄── ACK=1, seq=v, ack=u+1 ──────┤  2. ACK（服务端确认收到）
  │                               │
  │◄── FIN=1, ACK=1, seq=w, ack=u+1──┤  3. FIN（服务端关闭写）
  │                               │
  ├── ACK=1, seq=u+1, ack=w+1 ──►│  4. ACK（客户端确认关闭）
  │                               │
  │     TIME_WAIT (2MSL)          │
```

### 状态迁移

```
客户端：ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
服务端：ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

### 为什么要 TIME_WAIT？

1. **保证最后一个 ACK 到达服务端** — 如果 ACK 丢失，服务端重发 FIN，客户端还能重发 ACK
2. **防止旧连接的数据包干扰新连接** — 2MSL（Maximum Segment Lifetime）确保旧数据包在网络中消散

### TIME_WAIT 过多

服务端（尤其是短连接服务）可能出现大量 TIME_WAIT：

```bash
# 查看 TIME_WAIT 数量
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

# 内核参数调优
# /etc/sysctl.conf
net.ipv4.tcp_tw_reuse = 1      # 允许重用 TIME_WAIT 连接（客户端）
net.ipv4.tcp_fin_timeout = 30  # 减小 FIN-WAIT-2 超时
```

## 常见面试场景

| 问题 | 答案 |
|------|------|
| 为什么连接是三次，断开是四次？ | SYN 和 ACK 可以合并（三次），FIN 和 ACK 不能合并（服务端可能还有数据要发） |
| TCP 粘包怎么处理？ | 应用层定长/定界符/长度字段（消息头+消息体） |
| 大量 CLOSE_WAIT 说明什么？ | 服务端程序没有正确关闭连接（没有调用 close） |
| 大量 TIME_WAIT 怎么办？ | 开启 tcp_tw_reuse，使用长连接代替短连接 |
| SYN Flood 攻击？ | 服务端收到大量 SYN 但不完成握手，占用半连接队列。启用 SYN Cookie 防御 |
