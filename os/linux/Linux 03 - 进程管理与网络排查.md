---
title: "Linux 03 - 进程管理与网络排查"
date: 2024-01-01
tags: [Linux]
---

# Linux 03 - 进程管理与网络排查

## 进程管理

```bash
# 查看进程
ps aux                    # 所有进程
ps -ef                    # 标准格式
top                       # 实时进程监控（按 P=CPU, M=内存, q=退出）
htop                      # 增强版 top（推荐）

# 查找进程
pgrep -u user nginx       # 查找用户下的 nginx 进程
pidof nginx               # 查找 nginx 的 PID

# 杀死进程
kill PID                  # 默认 SIGTERM（15）
kill -9 PID               # SIGKILL（强制杀）
kill -15 PID              # SIGTERM（优雅终止）
pkill -f "process_name"   # 按名字杀
pkill -u user             # 杀某用户的所有进程
```

### 后台运行

```bash
# 后台运行
nohup command > output.log 2>&1 &
# 或使用 screen/tmux
screen -S session_name
tmux new -s session_name
```

## 系统资源

```bash
# CPU
lscpu                     # CPU 信息
cat /proc/cpuinfo         # 详细 CPU 信息
uptime                    # 系统负载（1m/5m/15m）

# 内存
free -h                   # 内存使用
cat /proc/meminfo

# 磁盘
iostat -x 1               # 磁盘 I/O（需 sysstat）
iotop                     # 进程级 I/O 监控

# 网络
ss -tuln                  # 监听端口（代替 netstat）
ss -tup                   # 已建立的连接
netstat -an               # 所有连接（需安装 net-tools）
```

## 网络排查命令

```bash
# DNS
nslookup example.com
dig example.com

# 连通性
ping -c 4 example.com
traceroute example.com       # 路由追踪

# HTTP 调试
curl -v http://example.com   # 完整请求/响应
curl -I https://example.com  # 仅响应头

# socket 统计
ss -s                        # socket 统计
```

### 排查思路

```bash
# 1. 端口是否监听
ss -tuln | grep 8080

# 2. 防火墙是否放行
iptables -L -n               # Linux 防火墙
ufw status                   # Ubuntu 防火墙

# 3. 服务状态
systemctl status nginx
journalctl -u nginx -n 50    # 查看服务日志最近 50 行

# 4. 资源瓶颈
# CPU：top 按 P 排序
# 内存：free -h
# 磁盘：df -h, iostat -x
# 网络：iftop, nethogs
```

## 日志查看

```bash
# 实时跟踪
tail -f /var/log/nginx/access.log

# 搜索
grep "ERROR" /var/log/app.log
grep -c "timeout" /var/log/app.log

# journalctl（systemd 日志）
journalctl -xe                  # 最近的错误
journalctl -f                   # 实时跟踪
journalctl -u nginx --since today
```
