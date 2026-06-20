---
title: "网络 07 - HTTPS与TLS"
date: 2026-06-11
tags: [网络]
---

# 网络 07 - HTTPS 与 TLS

## HTTP 与 HTTPS 区别

| 对比 | HTTP | HTTPS |
|------|------|-------|
| 协议 | HTTP | HTTP + TLS/SSL |
| 端口 | 80 | 443 |
| 加密 | 明文传输 | 加密传输 |
| 证书 | 不需要 | 需要 CA 证书 |
| 安全 | 易被中间人攻击 | 防窃听、防篡改、防冒充 |

## TLS 握手过程

```
客户端                         服务端
  │                               │
  ├── ClientHello ───────────────►│  1. 客户端打招呼（支持的TLS版本·加密套件·随机数）
  │                               │
  │◄── ServerHello ──────────────┤  2. 服务端回复（选定版本·选定加密套件·证书·随机数）
  │                               │
  │（客户端验证证书有效性）        │
  │                               │
  ├── ClientKeyExchange ────────►│  3. 客户端发 PreMaster Secret（用证书公钥加密）
  │                               │
  │    双方用三个随机数生成会话密钥  │
  │                               │
  ├── ChangeCipherSpec ─────────►│  4. 切换加密
  ├── Finished ─────────────────►│
  │                               │
  │◄── ChangeCipherSpec ─────────┤  5. 服务端切换加密
  │◄── Finished ─────────────────┤
  │                               │
  │        加密通信开始            │
```

## 证书体系

### CA 证书链

```
根 CA（内置在操作系统/浏览器中）
  └── 中间 CA
       └── 服务器证书（example.com）
```

### 证书内容

- 域名（CN / SAN）
- 颁发者（CA 机构）
- 有效期
- 公钥
- 签名（CA 的私钥签名）

### 证书验证

1. 检查域名是否匹配
2. 检查有效期
3. 检查证书链是否完整可信
4. 检查是否被吊销（CRL / OCSP）

## 获取免费证书

### Let's Encrypt + certbot

```bash
# 安装 certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d example.com -d www.example.com

# 续期（自动）
sudo certbot renew

# 查看证书
sudo certbot certificates
```

## HTTPS 性能优化

| 优化 | 说明 |
|------|------|
| **Session Resumption** | 复用之前的会话密钥，减少握手 |
| **OCSP Stapling** | 服务端主动缓存证书状态，减少客户端的验证延迟 |
| **TLS 1.3** | 1-RTT 握手（比 1.2 少一次往返） |
| **HSTS** | 强制客户端使用 HTTPS 访问 |
