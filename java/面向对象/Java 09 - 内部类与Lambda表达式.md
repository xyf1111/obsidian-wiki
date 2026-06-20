---
title: "Java 09 - 内部类与Lambda表达式"
date: 2026-06-11
tags: [java, 面向对象]
---

# Java 09 - 内部类与Lambda表达式

## 内部类

定义在另一个类内部的类。

### 成员内部类

```java
public class Outer {
    private int id;
    
    public class Inner {
        public void show() {
            System.out.println(id);  // 可访问外部类私有成员
        }
    }
}
```

### 静态内部类

```java
public class Outer {
    private static int count;
    
    public static class StaticInner {
        public void show() {
            System.out.println(count);
        }
    }
}
```

### 局部内部类

定义在方法中的类，只能在方法内使用。

### 匿名内部类

常用于简化事件监听、线程等场景：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("匿名内部类");
    }
}).start();
```

## Lambda 表达式（Java 8+）

简化**函数式接口**（只有一个抽象方法的接口）的实现。

```java
// 传统匿名内部类
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("传统写法");
    }
};

// Lambda 写法
Runnable r2 = () -> System.out.println("Lambda 写法");
```

### Lambda 语法

```java
(参数列表) -> { 方法体 }

// 无参
() -> System.out.println("Hello")

// 单参（可省略括号）
x -> x * 2

// 多参
(x, y) -> x + y

// 多条语句
(x, y) -> {
    int sum = x + y;
    return sum;
}
```

### 常用函数式接口（java.util.function）

| 接口 | 参数 | 返回值 | 用途 |
|------|------|--------|------|
| `Consumer<T>` | T | void | 消费型 |
| `Supplier<T>` | 无 | T | 供给型 |
| `Function<T,R>` | T | R | 函数型 |
| `Predicate<T>` | T | boolean | 判断型 |
