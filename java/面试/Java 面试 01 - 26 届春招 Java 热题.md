---
title: "Java 面试 01 - 26 届春招 Java 热题"
date: 2026-10-16
tags: [java, 面试, 八股, 集合, 并发]
source: "鱼皮·编程导航 / codefather"
---

# Java 面试 01 - 26 届春招 Java 热题

> 26 届春招 Java 高频面试题汇总（集合 / 反射与代理 / JDK 新特性），每题为「回答重点 + 扩展知识」结构，逐批追加。

## 一、JDK 1.8 对 HashMap 除了红黑树还进行了哪些改动？

**回答重点**：

- **改进哈希函数计算**：优化「扰动函数」，让哈希值的高低位都参与到桶的选择中，使分布更均匀、减少冲突。
- **扩容机制优化**：扩容迁移时不再对每个元素重新计算哈希值，而是根据原数组长度的高位来判断元素留在原位置还是迁移到新位置，减少计算、提升扩容效率。
- **头插法改为尾插法**：头插法插入时无需遍历链表，但扩容时会把链表逆序，多线程下可能成环导致死循环；改为尾插法后扩容顺序保持一致，规避该问题。

### 哈希函数的优化

JDK 1.7 的实现经历了四次异或，操作较多：

```java
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

JDK 1.8 简化为把 key 哈希码的高 16 位与低 16 位异或，得到的 hash 值同时拥有高位和低位特性，分布更均匀、不易冲突：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这是 JDK 开发者在**速度、实用性、哈希质量**之间做的权衡。

### 扩容机制优化

扩容时依据原容量高位判断元素归属：因数组长度是 2 的幂，扩容后元素的新下标要么是「原下标」，要么是「原下标 + 原容量」，无需重算哈希。

> 相关：[[Java 14 - HashMap 源码解析]]

### 头插法与尾插法

- 1.7 头插法：插入无需遍历链表、直接替换头结点；缺点是扩容时链表逆序，多线程操作下可能出现环并造成死循环。
- 1.8 尾插法：每次都从尾部插入，扩容后链表顺序与之前一致，因此不会出现多线程扩容成环的情况。

延伸：**改成尾插法后 HashMap 就不会死循环了吗？** 严格说仍可能，此时问题出在红黑树（如并发 `put` 触发树化 / 退化时的结构错乱），可进一步查阅 CSDN 专题文章：`blog.csdn.net/qq_33330687/article/details/101479385`。

---

## 二、JDK 动态代理和 CGLIB 动态代理有什么区别？

**回答重点**：

- **JDK 动态代理**基于接口，要求被代理类一定有定义接口，底层用 `java.lang.reflect.Proxy` + `InvocationHandler`，通过反射调用目标方法。
- **CGLIB** 基于 ASM 字节码生成工具，通过**继承**目标类生成子类来实现代理，因此不需要接口、可代理普通类，但要注意 `final` 方法（不可继承，无法被代理）。
- 性能随 JDK 版本而变（取自 haiq 的博客实测）：
  - JDK 6：调用次数少时两者差距不明显（JDK 动态代理甚至更快）；次数增多后 CGLIB 略快。
  - JDK 7：情况逆转——次数少（100 万）时 JDK 动态代理快约 30%；次数多（5000 万）时快接近 1 倍。
  - JDK 8：与 JDK 7 基本一致。

### JDK 动态代理

基于接口，使用 `Proxy` 类 + `InvocationHandler` 接口实现。

```java
// 接口
public interface Service {
    void perform();
}

// 需要被代理的实现类
public class ServiceImpl implements Service {
    @Override
    public void perform() {
        System.out.println("target method executed");
    }
}
```

切面 / 处理类：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class ServiceInvocationHandler implements InvocationHandler {
    private final Object target;

    public ServiceInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method invoke");
        Object result = method.invoke(target, args);
        System.out.println("After method invoke");
        return result;
    }
}
```

创建并使用代理对象：

```java
import java.lang.reflect.Proxy;

public class DynamicProxyDemo {
    public static void main(String[] args) {
        Service target = new ServiceImpl();
        Service proxy = (Service) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new ServiceInvocationHandler(target)
        );

        proxy.perform();
    }
}
```

实现原理：

1. 实现 `InvocationHandler` 接口得到一个切面类；
2. 通过 `Proxy` 根据目标类的类加载器、接口和切面类生成代理类；
3. 代理类把所有接口方法的调用转发到切面类的 `invoke()` 方法上，再由反射调用目标类的方法。

深入一点：代理类会在静态代码块中通过反射把接口的所有方法拿到并存入静态变量中（反编译后可见）。

### CGLIB

基于 ASM 字节码生成工具，通过继承实现代理，无需接口、可代理普通类，注意 `final` 方法不可代理。

```java
public class Service {
    public void perform() {
        System.out.println("target method executed");
    }
}
```

CGLIB 方法拦截器：

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class ServiceMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method invoke");
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method invoke");
        return result;
    }
}
```

创建并使用代理对象：

```java
import net.sf.cglib.proxy.Enhancer;

public class CglibDynamicProxyDemo {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Service.class);
        enhancer.setCallback(new ServiceMethodInterceptor());

        Service proxy = (Service) enhancer.create();
        proxy.perform();
    }
}
```

CGLIB 通过**字节码生成技术**而非反射来实现调用逻辑。

> 相关：[[Java 19 - 反射机制]]、[[Java 31 - AOP与自定义注解实践]]、[[Java 设计 04 - 装饰者模式及JDK源码应用]]

---

## 三、JDK 8 有哪些新特性？

**回答重点**：JDK 8 较为重要、平时常被问到的特性如下：

1. 用**元空间**替代了永久代；
2. 引入了 **Lambda 表达式**；
3. 引入了新的**日期类**、接口**默认方法 / 静态方法**；
4. 新增 **Stream 流式接口**；
5. 引入 **Optional 类**；
6. 新增 **CompletableFuture、StampedLock** 等并发实现类。

回答时若对 HashMap / ConcurrentHashMap 有准备，可主动抛出「Java 8 修改了 HashMap 和 ConcurrentHashMap 的实现」来引导面试官追问。

### 元空间替代了永久代

JDK 8 要把 JRockit 虚拟机与 Hotspot 融合，而 JRockit 没有永久代，因此去掉了 Hotspot 的永久代（本质也是永久代回收效率太低）。永久代在堆内、受 `-XX:MaxPermSize` 约束，元空间移到本地内存（`-XX:MaxMetaspaceSize`），类元数据分配更加灵活。

> 相关：[[Java 17 - JVM内存模型与GC]]

### Lambda 表达式

Lambda 是 Java 8 引入的匿名函数，可理解为**一段可以传递的代码**（把代码像数据一样传递），本质是**函数式接口的实例**。

```java
// 传统方式
Runnable runnable1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("run");
    }
};

// Lambda 表达式
Runnable runnable2 = () -> System.out.println("run");
```

> 相关：[[Java 34 - 函数式接口与Lambda表达式]]

### 日期类

Java 8 引入新的日期时间 API（`java.time` 包），更简洁易用，解决了旧 API 的诸多问题：

- 旧的 `Date`、`Calendar` 是**可变类且线程不安全**；新日期类**不可变**，创建后不能修改，避免意外修改、提升安全性与可维护性。
- `Date` 本身不含时区信息，必须配合 `Calendar` 处理时区，复杂且易错；新 API 提供专门的时区类（`ZonedDateTime`、`OffsetDateTime`、`ZoneId` 等），方法更直观易用。

```java
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();
```

### 接口默认方法、静态方法

默认方法允许在接口中定义方法的默认实现，实现类无需再实现这些方法；静态方法则用于把相关方法内聚在接口中，无需额外创建对象。

```java
interface MyInterface {
    default void defaultMethod() {
        System.out.println("Default Method");
    }

    static void hello() {
        System.out.println("Hello, New Static Method Here");
    }
}
```

### Stream 流式接口

Stream API 提供高效且易用的集合处理方式，支持链式操作、惰性求值和并行处理。

```java
List<String> list = Arrays.asList("a", "b", "c", "d");
List<String> result = list.stream()
                          .filter(s -> s.startsWith("a"))
                          .collect(Collectors.toList());
```

> 相关：[[Java 23 - Stream 流快速入门]]

### Optional

`Optional` 用于解决可能出现的 `NullPointerException`，提供一种优雅的方式处理可能为空的值。

```java
Optional<String> optional = Optional.of("value");
optional.ifPresent(System.out::println);
```

### CompletableFuture 与 StampedLock

`CompletableFuture` 提供了新的异步编程模型，简化异步任务的编写与管理：

```java
CompletableFuture.supplyAsync(() -> "Hello")
                 .thenApply(s -> s + " World")
                 .thenAccept(System.out::println);
```

`StampedLock` 是 JDK 8 新增的高性能读写锁（乐观读 / 悲观读 / 写三种模式），适合读多写少且对性能敏感的场景。

> 相关：[[Java 46 - CompletableFuture 异步编程]]、[[Java 50 - 四种锁实战（加入队伍功能）]]

> 来源：鱼皮·编程导航 / codefather
