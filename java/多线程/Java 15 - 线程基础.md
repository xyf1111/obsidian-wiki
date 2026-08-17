---
title: "Java 15 - 线程基础"
date: 2026-06-11
tags: [java, 多线程]
---

# Java 15 - 线程基础

## 线程与进程

| 概念 | 说明 |
|------|------|
| **进程** | 程序的一次执行，独立的内存空间；操作系统资源分配的最小单位 |
| **线程** | 进程内的执行单元，共享进程内存与系统资源；CPU 调度的最小单位 |

Java 默认线程：`main` 主线程 + GC 垃圾回收线程。运行 Java 程序即启动一个 JVM 进程，进程内所有代码以线程方式运行。

### 为什么需要线程

- 程序速度受三大因素制约：**CPU（计算能力）、内存（临时存储）、IO（持久化速度）**，三者速度差异巨大：`CPU > 内存 >> IO`，IO 是最主要瓶颈。
- 早期单核 CPU 通过「多进程分时复用」实现同时运行多个程序：操作系统给每个进程分配一段 CPU 使用权（**时间片**），用完即切换下一个进程（毫秒级，用户无感知）。
- 进程占用时间片时若需进行耗时 IO 操作，会让出 CPU 给其他进程，待 IO 完成后再重新获取时间片，以提高 CPU 利用率。
- 进程切换开销大（每个进程有独立内存空间、不共享数据），为提升并发性能与 CPU 利用率，进程内部诞生了**线程**：线程共享进程的内存空间与系统资源，切换与通信成本更低。

### 进程与线程的区别

- **调度粒度**：线程是 CPU 调度的最小单位，进程是操作系统分配资源的最小单位；线程的划分尺度小于进程。
- **开销**：创建/终止进程需分配独立内存空间与系统资源，开销较大；创建/终止线程共享进程资源，开销较小。
- **调度方式**：操作系统为进程分配时间片；线程作为进程内执行单元由进程来调度。
- **并行能力**：多核处理器下多线程可实现真正并行执行，进一步提高系统性能。

## 创建线程的三种方式

### 方式一：继承 Thread

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("线程运行中");
    }
}

new MyThread().start();
```

### 方式二：实现 Runnable（推荐）

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("任务执行中");
    }
}

new Thread(new MyTask()).start();

// Lambda 简写
new Thread(() -> System.out.println("任务")).start();
```

### 方式三：实现 Callable + FutureTask（可获取返回值）

```java
class MyCall implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return 42;
    }
}

FutureTask<Integer> task = new FutureTask<>(new MyCall());
new Thread(task).start();
Integer result = task.get();  // 阻塞获取结果
```

## 线程生命周期

```
新建 (New) → 就绪 (Runnable) → 运行 (Running) → 死亡 (Terminated)
                ↑                    ↓
             阻塞 (Blocked / Waiting / Timed_Waiting)
```

| 状态 | 说明 |
|------|------|
| **NEW** | 新建，未调用 start() |
| **RUNNABLE** | 就绪 + 运行中 |
| **BLOCKED** | 等待锁 |
| **WAITING** | 无限期等待（wait/join/park） |
| **TIMED_WAITING** | 限期等待（sleep/wait(time)/join(time)） |
| **TERMINATED** | 执行完毕 |

## 常用方法

| 方法 | 说明 |
|------|------|
| `start()` | 启动线程 |
| `sleep(ms)` | 当前线程休眠（不释放锁） |
| `join()` | 等待线程结束 |
| `yield()` | 让出 CPU 时间片 |
| `interrupt()` | 中断线程 |
| `isAlive()` | 线程是否存活 |

## 线程优先级

范围 1~10，默认 5（`Thread.NORM_PRIORITY`），仅作**建议**，具体由 OS 决定。
