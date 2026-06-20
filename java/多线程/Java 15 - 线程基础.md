---
title: "Java 15 - 线程基础"
date: 2026-06-11
tags: [java, 多线程]
---

# Java 15 - 线程基础

## 线程与进程

| 概念 | 说明 |
|------|------|
| **进程** | 程序的一次执行，独立的内存空间 |
| **线程** | 进程内的执行单元，共享进程内存 |

Java 默认线程：`main` 主线程 + GC 垃圾回收线程。

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
