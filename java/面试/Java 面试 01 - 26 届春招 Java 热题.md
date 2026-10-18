---
title: "Java 面试 01 - 26 届春招 Java 热题"
date: 2026-10-16
tags: [java, 面试, 八股, 集合, 并发]
source: "鱼皮·编程导航 / codefather"
---

# Java 面试 01 - 26 届春招 Java 热题

> 26 届春招 Java 高频面试题汇总（集合 / 并发 / 反射与代理 / JDK 新特性 / JVM / Spring 机制），每题为「回答重点 + 扩展知识」结构，逐批追加。

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

---

## 四、Java 中 ConcurrentHashMap 1.7 和 1.8 之间有哪些区别？

**回答重点**：

- **锁的粒度**：JDK 1.7 采用**分段锁**，每个 `Segment` 独立、可并发访问，默认 16 个 `Segment`，最多 16 个线程并发执行；JDK 1.8 移除 `Segment`，锁粒度细化到链表/红黑树的**节点级别**。
- **并发控制手段**：1.8 插入用 **CAS**，只有在更新链表或红黑树时才用 `synchronized`，且只锁住链表/树的**头节点**，进一步减少锁竞争、提升并发度。
- **数据结构**：1.7 只有**数组 + 链表**；1.8 与 `HashMap` 一样引入**红黑树**。
- 此外，**扩容机制**与 **`size` 计算**的方式也不一样。

### ConcurrentHashMap 1.7 的结构（分段锁）

原理：先通过 key 的 hash 得到 `Segment` 数组下标，将该 `Segment` 上锁，再通过 key 的 hash 得到 `Segment` 内 `HashEntry` 数组的下标——此后就与 `HashMap` 一致了。因此可简化理解为：**每个 `Segment` 里存放的就是一个单独的 `HashMap`**。

- **并发度示例**：若有 6 个 `Segment`，就等于有 6 把锁，最多 6 个线程同时操作，并发度为 6；相比直接给整个 `put` 方法上锁，并发度更高，这就是分段锁。
- **上锁来源**：`Segment` **继承了 `ReentrantLock`**，自身具备加锁功能。
- **缺陷**：`Segment` 数组一旦初始化就**不扩容**，只有 `HashEntry` 数组会扩容——导致并发度过于死板，不能随数据增加而提高。

### ConcurrentHashMap 1.8 的结构（节点级锁）

- 更细粒度的锁控制：可理解为 **1.8 `HashMap` 数组的每个位置都是一把锁**，数组扩容后锁也随之变多，并发度增加。
- 思想转变：不再分段，直接把 `Node` 数组的每个节点各上一把锁。
- 不再借助 `ReentrantLock`，直接用 `synchronized`（侧面说明 JDK 1.8 对 `synchronized` 优化后性能已不亚于 `ReentrantLock`）。
- **插入流程**：计算 key 的 hash 得到下标 → 若该下标为空，用 **CAS** 塞入新 `Node`；若已有节点，用 `synchronized` 锁住该节点（其他线程无法访问该节点及其之后的所有节点）→ 判断 key 是否相等，相等则替换 value，否则新增节点（与 `HashMap` 一致）。

### 扩容的区别

**JDK 1.7**：

- **基于 Segment**：每个 `Segment` 内含一个 `HashMap`；当某个 `Segment` 达到扩容阈值时，**单独为该 `Segment` 扩容**，不影响其他 `Segment`。
- **扩容过程**：每个 `Segment` 维护自己的负载因子，超过阈值时该 `Segment` 的 `HashMap` 才扩容，整体并非一次性全部扩容。

**JDK 1.8**：

- **全局扩容**：取消 `Segment` 变成全局数组（类似 `HashMap`），任意位置元素超阈值时**整个数组扩容**。
- **基于 CAS**：扩容通过 CAS 保证线程安全，避免锁住整个数组；多个线程可同时协助扩容。
- **渐进式扩容**：不是一次性重新分配全部数据，而是多个线程共同参与、逐步迁移旧数据，降低扩容时的性能开销。

#### 渐进式扩容分析

- put 时若发现当前 node 的 hash 值为 **-1**，表明该节点正在扩容（ForwardingNode），触发**协助扩容机制**。
- 思路：扩容无非是搬迁 Node。假设数组长 32，可分段搬迁——31~16 由线程 A 搬、15~0 由线程 B 搬。
- 实现：维护一个 `transferIndex` 变量，来的线程**死循环 CAS 争抢下标**——下标已分配完就不需要搬，CAS 抢到待搬下标就去帮忙搬。

### size 逻辑的区别

**JDK 1.7**（「尝试」思想）：调用 `size` 时先**尝试三次不加锁获取**；三次总数一致说明数量没变，直接返回；不一致说明有线程在增删 map，则加锁计算，此时其他线程无法操作 map。

```java
if (retries++ == RETRIES_BEFORE_LOCK) {
    for (int j = 0; j < segments.length; ++j)
        ensureSegment(j).lock(); // force creation
}
// ...再累加数量
```

**JDK 1.8**：直接计算返回，采用**类似 `LongAdder` 的思想**——累加不再基于单个成员变量，而是搞一个数组，每个线程在各自对应下标处累加，最后统一汇总。

- put 时通过 `addCount` 方法维护 `counterCells` 数量（remove 也会调用）。
- 流程：先 CAS 修改 `baseCount`，成功即返回；失败说明有竞争，则通过 hash 选一个 `CounterCell` 修改。最终 `size = baseCount + 所有 CounterCell`。
- 本质：用 counterCell 数组**分散并发场景下对单个成员变量的竞争压力**，提高并发度——这就是 `LongAdder` 的思想。

> 相关：[[Java 11 - Set与Map]]、[[Java 14 - HashMap 源码解析]]、[[Java 16 - 锁与并发工具]]、[[Java 29 - Synchronized 关键字详解]]

---

## 五、Java 中 ConcurrentHashMap 的 get 方法是否需要加锁？

**回答重点**：

- **不需要加锁**。通过 `volatile` 关键字，`ConcurrentHashMap` 能确保 `get` 方法的线程安全——即使在写入发生时，读取线程仍能获得最新数据，不会引发并发问题。
- 具体是通过 `Unsafe#getXXXVolatile` 以及用 `volatile` 修饰节点的 `val`、`next` 指针来实现的。

### 写时加锁

写操作（`put`、`remove`）需要加锁，以保证写入时的线程安全与一致性；而 `get` 操作不需要锁。这样 `ConcurrentHashMap` 实现了**高效的读写分离**——读操作不阻塞写操作，从而提高并发性能。

### get 方法源码分析

- 主要定位逻辑在这一行：`e = tabAt(tab, (n - 1) & h)) != null`。
- `tabAt` 内部使用的就是 `Unsafe#getObjectVolatile` 来保证可见性。
- `getObjectVolatile` 实际是一个 `native` 本地方法，通过 JNI（Java Native Interface）调用底层 C++ 实现；其原理是根据对象的起始地址和字段偏移量直接从内存读取字段值，再通过**内存屏障**确保该读取具有 `volatile` 语义（读操作前后各插入一个读屏障 load barrier，保证字段值不被 CPU 缓存）。
- 因此不需加锁：利用 `Unsafe` 获取元素，再对比 hash 值与 key 即可得到 value（即普通 map 的 get 流程）。
- `Node` 节点内的 `val` 和 `next` 指针同样被 `volatile` 修饰，也能保证可见性。
- 综上，无论是 hash 映射到数组中的具体 `node`，还是 hash 冲突时借助 `next` 指针遍历链表，或是定位后再获取 `val`，这几个关键点都能保证可见性——因此不需要加锁。

### Unsafe 类

`Unsafe` 是 Java 提供的一个内部类，用于执行不安全操作，提供**直接操作内存和线程的能力**。关键功能：

- **直接内存访问**：允许直接分配、释放、读写内存。
- **对象操作**：允许直接操作对象字段（设置或获取字段值）。
- **线程操作**：包括暂停和恢复线程、管理锁等。
- **CAS 操作**：提供 compare-and-swap，支持原子性更新。

> 相关：[[Java 07 - 并发三大根源：可见性、原子性、有序性]]、[[Java 14 - HashMap 源码解析]]、[[Java 16 - 锁与并发工具]]

---

## 六、Java 中 HashMap 的扩容机制是怎样的？

**回答重点**：

- 扩容基于**负载因子（load factor）**决定：默认 `0.75`，当已存储元素数量超过当前容量的 75% 时触发扩容。
- 例如初始容量 16、负载因子 0.75，则扩容阈值为 `16 × 0.75 = 12`，当存入第 13 个元素时就触发扩容。
- 触发扩容时，容量扩大为当前容量的**两倍**（16 → 32 → 64…）。
- 扩容时需要**重新计算所有元素的哈希值并重新分配到新的哈希桶**，这个过程称为 **rehashing**。

### rehashing 细节

直觉上，每个元素应重新 hash、逐个搬迁过去——JDK 1.7 就是这样实现的。JDK 1.8 在此做了优化，关键点在于**数组长度是 2 的次方，且扩容为 2 倍**。

- 定位下标用 `(数组长度-1) & hash`。举例：长度 16 → `16-1=15，二进制 001111`；长度 32 → `32-1=31，二进制 011111`。二者差别只在倒数第 5 位多了一个 1。
- 因此重点就在 key 的 hash 值**从右往左数第五位是否为 1**：为 1 说明需搬迁到新位置，新下标 = 原下标 + 16（原数组大小）；为 0 说明吃不到新数组长度的高位，仍留在原位、不迁移。
- 这也意味着链表数据的搬迁是一次性算完再整批搬运——扩容时节点下标**只有「原位置」或「原位置 + 老数组长度」两种可能**，不存在第三种选择。

**总结**：

- 扩容发生时，`HashMap` 并非简单地把元素复制到新数组；每个元素的哈希会按新的数组容量重新参与下标计算，存储位置会发生变化。
- Java 8 之后的扩容不需要每个节点重新 hash 算下标，因为元素的新位置只与「高位」有关，通过与老数组长度做 `&` 运算判断是否为 0 即可定位新下标，因此链表中的元素可能只需部分移动，这一优化减少了扩容时的计算开销。

### 扩容的考虑

- 每次扩容都要遍历当前所有元素、重新计算哈希并移动它们，因此扩容是比较耗时的操作；频繁扩容会明显拖累整体性能。
- **优化策略**：如果能预估 `HashMap` 大概会存多少数据，可在创建时就指定一个较大的初始容量，以减少扩容次数。例如要存 100 万个元素，可直接设置 `1024 × 1024` 的初始容量，避免多次扩容。

> 相关：[[Java 11 - Set与Map]]、[[Java 14 - HashMap 源码解析]]

---

## 七、Java 中 ReentrantLock 的实现原理是什么？

**回答重点**：

- `ReentrantLock` 本质是**基于 AQS 实现的可重入锁**，支持**公平**和**非公平**两种获取方式。
- 内部依赖一个 `state` 变量 + 两个等待队列：**同步队列**（双向链表）与**等待队列**（条件队列，单向链表）。
- 线程通过 **CAS 修改 `state`** 来争抢锁；争抢失败则进入同步队列排队等待。
- 当 `condition` 条件不满足时，线程进入等待队列等待。
- **公平锁与非公平锁的区别**：获取锁时，公平锁是「加入同步队列尾部排队」，非公平锁是「直接尝试 CAS 争抢」。

### CAS

CAS（Compare-And-Swap，比较并交换）是一种乐观锁原子操作：比较内存当前值是否等于预期值，相等才写入新值，整个过程由 CPU 指令保证原子性。它是 `ReentrantLock`、`AtomicInteger` 等并发组件的基础。

### 自旋锁

自旋锁（Spinlock）是一种**轻量级**锁机制：线程获取锁失败时不立即阻塞，而是在循环中反复尝试获取，直到成功。它避免了线程上下文切换开销，**适用于锁等待时间较短的场景**。

```java
public class SpinLock {
    private final AtomicBoolean lock = new AtomicBoolean(false);

    public void lock() {
        while (!lock.compareAndSet(false, true)) {
            // 自旋等待
        }
    }

    public void unlock() {
        lock.set(false);
    }
}
```

缺点主要有两点：

- **锁饥饿问题**：高并发场景下，可能存在某个线程一直 CAS 失败，始终抢不到锁。
- **性能问题**：多核处理器对同一变量高并发 CAS，会导致**总线风暴**问题。

### CLH 自旋锁

针对自旋锁的问题，演进出基于队列的自旋锁 **CLH**（Craig, Landin, and Hagersten），适用于多处理器环境下的高并发场景。

- 原理：维护一个**隐式队列**，使线程在等待锁时**自旋在本地变量**上，从而减少对共享变量的争用和缓存一致性流量。
- 线程按序排队争抢锁，每个线程不再 CAS 争抢同一个变量，而是**自旋判断其前驱线程的状态**；前驱释放锁后，后续线程即可抢锁。
- 优点：排队按序争抢解决了**锁饥饿**；CAS 自旋监听前驱状态避免了**总线风暴**。
- 缺点：自旋期间线程一直占用 CPU，仅适用于锁等待时间较短的场景。
- 「隐式」的含义：线程之间并没有真正用指针连接，而是用 `AtomicReference + ThreadLocal` 实现隐式关联。

```java
public class CLHLock {
    private static class CLHNode {
        volatile boolean isLocked = true; // 默认加锁状态
    }

    private final ThreadLocal<CLHNode> currentNode;
    private final ThreadLocal<CLHNode> predecessorNode;
    private final AtomicReference<CLHNode> tail;

    public CLHLock() {
        this.currentNode = ThreadLocal.withInitial(CLHNode::new);
        this.predecessorNode = new ThreadLocal<>();
        this.tail = new AtomicReference<>(new CLHNode());
    }

    public void lock() {
        CLHNode node = currentNode.get();
        CLHNode pred = tail.getAndSet(node);
        predecessorNode.set(pred);

        // 自旋等待前驱节点释放锁
        while (pred.isLocked) {
        }
    }

    public void unlock() {
        CLHNode node = currentNode.get();
        node.isLocked = false; // 释放锁
        currentNode.set(predecessorNode.get()); // 回收当前节点
    }
}
```

### AQS 对 CLH 的改造

- 因为 CLH 的自旋会占用 CPU，**AQS 把「自旋等待前驱节点」改成了「阻塞线程」**。
- 后续线程阻塞后无法主动发现前驱已释放锁，因此**前驱线程需显式通知后续线程**——这正是 AQS 需要维护**显式双向队列**的原因（前驱要能找到后继）。
- 若前驱线程等待超时或主动取消，需从队列中移除，后面的线程需要「顶」上来。

AQS 中线程的等待状态：

| 状态 | 值 | 含义 |
|------|----|------|
| 默认 | 0 | 初始化时的默认值 |
| CANCELLED | 1 | 由于超时、中断或其他原因，该节点被取消 |
| SIGNAL | -1 | 该节点准备就绪，正常等待资源 |
| CONDITION | -2 | 该节点位于条件等待队列中 |
| PROPAGATE | -3 | SHARED 情况下有效，将 releaseShared 动作传播到其他节点 |

> 相关：[[Java 16 - 锁与并发工具]]、[[Java 07 - 并发三大根源：可见性、原子性、有序性]]

---

## 八、Java 中 volatile 关键字的作用是什么？

**回答重点**：`volatile` 主要保证变量的**可见性**和**禁止指令重排优化**。

1）**可见性（Visibility）**：当一个线程修改了 `volatile` 变量的值，新值会立即刷新到主内存，其他线程读取该变量时可立即获得最新值，从而避免线程间因缓存一致性问题导致的「看见旧值」现象。

2）**禁止指令重排序（Ordering）**：`volatile` 通过内存屏障禁止特定情况下的指令重排序，保证程序执行顺序符合预期。对 `volatile` 变量的**写**操作会在其**前面**插入一个 `StoreStore` 屏障，对 `volatile` 变量的**读**操作会在其**后面**插入一个 `LoadLoad` 屏障，确保多线程环境下某些代码块执行顺序可预测。

### 可见性问题与 volatile 的解决

Java 多线程环境中，每个线程都有自己的工作内存（CPU 缓存），会从主内存读取变量副本再操作，因此线程 A 修改某个变量后，线程 B 不一定能立即看到。`volatile` 通过强制线程直接从主内存读取或写入变量，解决了可见性问题。

示例：没有 `volatile` 时，一个线程可能永远不会看到另一个线程对共享变量的修改。

```java
private static boolean flag = false;

public static void main(String[] args) {
    new Thread(() -> {
        while (!flag) {
            // do something
        }
        System.out.println("Thread terminated.");
    }).start();

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    flag = true; // 没有 volatile, 这个改变对其他线程不可见
}
```

如果把 `flag` 声明为 `volatile`，线程 A 修改后线程 B 能立即看到修改并终止循环。

### 禁止指令重排序与双重检查锁定

Java 编译器和 CPU 为了优化性能可能对指令重排序。单线程程序一般不受影响，但多线程环境下重排序可能导致意外结果。经典场景是单例模式中的**双重检查锁定（Double-Checked Locking）**：

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

上例中若不使用 `volatile`，指令重排序可能导致 `instance` 在其构造函数完成之前就被返回，使其他线程访问到一个未初始化完全的对象。

### volatile 不能保证操作的原子性

`volatile` 保证了可见性和有序性，但**不能保证操作的原子性**。原子性意味着一个操作不可分割、不能被中断。典型例子是 `i++`，它实际分为「读取 `i` 的值 → 递增 → 写回」三个步骤；若多个线程同时执行 `i++`，每个线程都可能读到相同的初始值，最终结果可能不正确。

```java
private static volatile int count = 0;

public static void main(String[] args) {
    for (int i = 0; i < 1000; i++) {
        new Thread(() -> count++).start();
    }
    System.out.println(count); // 结果可能不为 1000
}
```

解决方式是使用 `AtomicInteger` 或者 `synchronized` 块。

### volatile 与 synchronized 的对比

- **性能**：`volatile` 是轻量级的同步机制，开销较小，但只能用于变量的可见性和禁止重排序，无法实现复杂的同步逻辑；`synchronized` 是重量级同步机制，可保证代码块的原子性和可见性，但开销较大。
- **使用场景**：`volatile` 适用于简单的状态标志、标记等场景；`synchronized` 更适合复杂的临界区保护、需要确保多个操作原子性的场景。

### Java 内存模型（JMM）与 volatile

Java 内存模型（Java Memory Model, JMM）定义了多线程环境下共享变量的可见性和指令重排序规则。`volatile` 是 JMM 的关键组成部分，提供了一种简化方式来实现线程间的内存可见性。JMM 中规定：

- 对一个 `volatile` 变量的**写**操作，happens-before 于每一个后续对该 `volatile` 变量的**读**操作。
- `volatile` 变量的操作不会与其他内存操作进行重排序。

> 相关：[[Java 07 - 并发三大根源：可见性、原子性、有序性]]、[[Java 16 - 锁与并发工具]]

---

## 九、Java 中什么情况会导致死锁？如何避免？

**回答重点**：这是操作系统中的经典理论——死锁的发生必须**同时满足**以下四个条件，称为「死锁的必要条件」：

1. **互斥条件**：每个资源只能被一个线程占用。
2. **占有和等待**：线程在持有至少一个资源的同时，等待获取其他资源。
3. **不可抢占**：线程所获得的资源在未使用完毕之前不能被其他线程抢占。
4. **循环等待**：多个线程形成一种头尾相接的循环等待资源关系。

只要**打破上述任意一个条件**，就能避免死锁的发生。

### 避免死锁的方法

1. **按序申请资源**：确保所有线程在获取多个锁时，按照相同的顺序获取锁。
2. **尽量减少锁的范围**：将锁的粒度尽可能缩小，减少持有锁的时间（可通过拆分锁或使用更细粒度的锁实现）。
3. **使用尝试锁机制**：使用 `ReentrantLock` 的 `tryLock` 方法，在一段时间内尝试获取锁，若无法获取则放弃或采取其他措施，避免死锁。
4. **设置超时等待时间**：为锁操作设置超时，防止线程无限期地等待锁。
5. **避免嵌套锁**：尽量避免在一个锁的代码块中再次尝试获取另一个锁。

> 相关：[[Java 16 - 锁与并发工具]]

---

## 十、Java 中有哪些垃圾回收算法？

**回答重点**：基础算法有三类——**标记-清除**（Mark-Sweep）、**标记-整理**（Mark-Compact）、**复制**（Copying）。现代 JVM 按分代思想组合使用：新生代对象「朝生夕死」，用复制算法以极低的存活复制成本换无碎片；老年代存活率高，用标记-清除 / 标记-整理。

### 标记-清除（Mark-Sweep）

两个阶段：

- **标记阶段（tracing）**：从 GC Roots（栈、寄存器、全局变量等）出发遍历对象图，标记每一个可达对象。
- **清除阶段**：扫描堆，把未被标记的对象当作垃圾回收。

**优点**：实现简单，能处理堆中的所有对象。

**缺点**：清除阶段不移动、不整理内存，空闲空间通常以**空闲链表（双向链表）**管理，于是产生**内存碎片**——总内存明明够，却因为找不到足够大的连续块而申请失败；分配时还要遍历链表找合适的内存块，比较耗时。

针对缺点的两种工程改进：

- **多空闲链表**：按内存块大小拆成多个链表（如大分块链表、小分块链表，还可继续细分），申请时按所需大小查对应链表，加快分配。
- **位图标记法**：把标记位放在对象头里对**写时复制**不兼容——每次 GC 都要改对象，而 fork 出来的进程共享同一块内存，改对象必然触发复制。改用位图后，堆内存按块划分，由「对象地址 − 堆起始地址」算出对象落在第几块，把位图中对应 bit 置 1 即可标记。位图还能加速清除阶段的遍历：不必逐对象扫描堆找标记位。

但无论标记位放对象头还是用位图，**碎片问题都解决不了**，于是引出标记-复制与标记-整理。

### 标记-复制（Copying）

堆分成 From、To 两块，对象只在 From 上分配。发生 GC 时找出所有存活对象复制到 To，再整体回收 From 区，最后两块交换角色（To 变 From，From 变 To）。

**优点**：

- 存活对象被紧凑排列，**不产生内存碎片**；
- 不需要空闲链表，**移动指针即可分配**，效率很高；
- **CPU 缓存亲和性好**：从根开始深度优先遍历，把关联对象找齐并分配在相近地址，依据局部性原理，一个对象被加载时其引用对象往往同时进缓存，访问直接命中。

**缺点**：

- 对象只能在 From 区分配，而 From 只有堆的一半，**内存利用率 50%**；
- 存活对象多时**复制压力大**，速度会慢；
- 需要移动对象，**不适用于保守式 GC**；
- 深度优先即递归调用，有栈溢出风险 → **Cheney 算法**改用迭代的广度优先遍历规避。

### 标记-整理（Mark-Compact）

思路与复制相近，区别是不分区：在同一空间内直接把存活对象向边界整理。

**优点**：无内存碎片，且不必像复制算法那样预留一半空间，**内存利用率更高**。

**缺点**：同一空间内既要标记又要移动，需对堆做多次搜索，整体耗时更长；堆越大，这个代价越突出。

### 分代收集中的落地

新生代采用复制算法：划分为 **Eden + 两个 Survivor（S0、S1）**。对象默认在 Eden 分配，Minor GC 时把 Eden 与当前 Survivor 中的存活对象复制到另一个空闲 Survivor，两块 Survivor 交替承担 From / To 角色。这样只付出很小一块空闲区域（而非半个堆），就拿到了复制算法「无碎片 + 指针分配」的全部好处——这也是「为何新生代要分 S0/S1/Eden」的根因。

> 相关：[[Java 17 - JVM内存模型与GC]]、[[Java 56 - OOM 内存溢出排查实战]]

---

## 十一、Java 中有哪些集合类？请简单介绍

**回答重点**：Java 集合分两大体系——**Collection**（单元素集合，下分 List / Set / Queue）与 **Map**（键值对，通过 key 找 value）。这是面试开场的暖场题，先讲清体系骨架，再挑自己熟悉的实现类深入，面试官会顺着你给的点继续挖。

### List（有序、可重复）

| 实现类 | 底层结构 | 特点 |
|---|---|---|
| `ArrayList` | 动态数组 | 随机访问快，中间插入/删除慢（要搬移元素） |
| `LinkedList` | 双向链表 | 插入/删除快，查询慢 |
| `Vector` | 动态数组 | 线程安全（方法加 `synchronized`），开销大，已不推荐 |

### Set（不重复）

| 实现类 | 底层结构 | 特点 |
|---|---|---|
| `HashSet` | 哈希表 | 元素无序，不允许重复 |
| `LinkedHashSet` | 链表 + 哈希表 | 维护**插入顺序**，不允许重复 |
| `TreeSet` | 红黑树 | 元素**有序**（自然顺序或 Comparator），不允许重复 |

⚠️ 网上「Set 是无序集合」的说法不准确——有序与否要看具体实现类：`LinkedHashSet` 保插入序，`TreeSet` 保持排序。

### Queue

| 实现类 | 底层结构 | 特点 |
|---|---|---|
| `PriorityQueue` | 优先级堆 | 按自然顺序或指定 Comparator 出队 |
| `LinkedList` | 双向链表 | 同时实现 `Deque`，可作 FIFO 队列使用 |

### Map（键值对，键不重复）

| 实现类 | 底层结构 | 特点 |
|---|---|---|
| `HashMap` | 哈希表 | 键无序，允许一个 `null` 键与多个 `null` 值 |
| `LinkedHashMap` | 链表 + 哈希表 | 维护插入顺序（也可配 `accessOrder` 做 LRU） |
| `TreeMap` | 红黑树 | 按键有序 |
| `Hashtable` | 哈希表 | 线程安全（全局锁），不允许 `null` 键/值 |
| `ConcurrentHashMap` | 哈希表 | 线程安全，适合高并发，不允许 `null` 键/值 |

### 记忆抓手

按「接口 → 抽象类 → 实现类」的三层框架记忆（`AbstractList` / `AbstractSet` / `AbstractMap` 等），既方便串起整个类图，也能顺带引出「接口与抽象类的区别」这一延伸考点。

> 相关：[[Java 10 - Collection与List]]、[[Java 11 - Set与Map]]、[[Java 14 - HashMap 源码解析]]

---

## 十二、Java 中的 HashMap 和 Hashtable 有什么区别？

**回答重点**：`Hashtable` 是 JDK 1.0 的遗留类，方法级 `synchronized` 保证线程安全但性能差；`HashMap` 非线程安全但单线程性能好，且允许一个 `null` 键与多个 `null` 值。需要线程安全时用 `ConcurrentHashMap`，而不是 `Hashtable`。

| 维度 | HashMap | Hashtable |
|---|---|---|
| **线程安全性** | 非线程安全，多线程下可能数据不一致 | 线程安全，大部分方法用 `synchronized` 同步 |
| **性能** | 无同步开销，单线程优于 Hashtable | 全局锁使不同操作也被串行化，性能低 |
| **null 支持** | 允许 1 个 `null` 键、多个 `null` 值 | 不允许 `null` 键/值，插入即抛 `NullPointerException` |
| **继承结构** | 继承 `AbstractMap`，实现 `Map` | 继承已废弃的 `Dictionary`，后来才实现 `Map`；Java 2 之后逐渐被 HashMap 取代，不推荐使用 |
| **迭代器** | **快速失败**（fail-fast）的 `Iterator`，迭代中结构性修改抛 `ConcurrentModificationException` | **弱一致性**的 `Enumerator`，迭代中修改不抛异常（但同样不建议这么用） |

### 线程安全场景的正确选择

`ConcurrentHashMap` 是 `Hashtable` 的替代方案：

- `Hashtable` 用一把全局锁，读读、读写、写写全部互斥；
- `ConcurrentHashMap` 读操作无锁化，写操作只锁局部（JDK 1.7 分段锁 → JDK 1.8 桶级 CAS + synchronized），高并发下吞吐远高于 `Hashtable`；
- 代价是不支持 `null` 键/值——因为 `get` 返回 `null` 时无法区分「键不存在」与「值就是 null」，无锁读场景下这个歧义无法用 `containsKey` 补救。

> 相关：[[Java 14 - HashMap 源码解析]]、[[Java 11 - Set与Map]]、[[Java 13 - fail-fast机制]]

---

## 十三、Java 中的 LinkedHashMap 是什么？

**回答重点**：`LinkedHashMap` 继承 `HashMap`，在哈希表之上额外用一条**双向链表**串起所有节点，因此既能 O(1) 定位，又能保证遍历顺序：默认是**插入顺序**，构造时传 `accessOrder = true` 则变成**访问顺序**（即 LRU 语义）。典型用途是「需要保持插入顺序的 Map」和「本地 LRU 缓存」。

### 扩展知识

**1）实现方式：HashMap 的槽位 + 一条链表**

`LinkedHashMap.Entry` 继承 `HashMap.Node`，只多了两个指针 `before` / `after`，把所有 Entry 按顺序串成双向链表。哈希桶仍然是原来的数组 + 链表/红黑树结构，所以查找逻辑完全复用父类，链表只是「额外的顺序视图」。

内部成员 `accessOrder` 默认 `false`：

| accessOrder | 链表顺序 | 命中节点后的动作 |
|---|---|---|
| `false`（默认） | 插入顺序 | 位置不变 |
| `true` | 访问顺序 | 把该节点移到链表尾部（标记为最近访问） |

**2）靠三个回调钩子接入父类流程**

`HashMap` 在关键位置预埋了三个空实现的模板方法，`LinkedHashMap` 重写它们来维护链表：

```java
void afterNodeAccess(Node<K,V> p);      // get / put 命中已有节点后
void afterNodeInsertion(boolean evict); // 插入新节点之后
void afterNodeRemoval(Node<K,V> p);     // 删除节点之后
```

好处是 `putVal` / `getNode` / `removeNode` 一行不用改——父类只管哈希表，子类只管顺序，典型的模板方法模式。

**3）用 `removeEldestEntry` 做 LRU**

`afterNodeInsertion` 里会调用 `removeEldestEntry(eldest)`，默认实现返回 `false`（永不淘汰）。子类重写它即可在插入时淘汰链表头部最久未使用的节点：

```java
private static final class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxCacheSize;

    LRUCache(int initialCapacity, int maxCacheSize) {
        super(initialCapacity, 0.75F, true); // accessOrder = true
        this.maxCacheSize = maxCacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return this.size() > this.maxCacheSize;
    }
}
```

要点：`accessOrder` 必须为 `true`，否则链表头是「最早插入」而不是「最久未访问」，LRU 会退化成 FIFO。注意 `LinkedHashMap` 本身**不是线程安全**的，并发场景需外部加锁或换 `Collections.synchronizedMap` / Caffeine。

**4）手写 LRU（经典笔试题）**

面试常要求不用 `LinkedHashMap` 实现 O(1) 的 LRU：`HashMap` 负责 key → 节点定位，自定义双向链表负责顺序，`head` / `tail` 用哨兵节点省掉边界判断。

```java
class LRUCache<K, V> {
    class Node {
        K key; V value; Node prev, next;
        Node() {}
        Node(K key, V value) { this.key = key; this.value = value; }
    }

    private final int capacity;
    private final Map<K, Node> map = new HashMap<>();
    private final Node head = new Node(), tail = new Node();

    LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public V get(K key) {
        Node node = map.get(key);
        if (node == null) return null;
        moveToHead(node);
        return node.value;
    }

    public void put(K key, V value) {
        Node node = map.get(key);
        if (node != null) {
            node.value = value;
            moveToHead(node);
            return;
        }
        if (map.size() >= capacity) {
            map.remove(tail.prev.key);   // 淘汰最久未用
            unlink(tail.prev);
        }
        Node newNode = new Node(key, value);
        map.put(key, newNode);
        addToHead(newNode);
    }

    private void moveToHead(Node node) { unlink(node); addToHead(node); }

    private void addToHead(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    private void unlink(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
}
```

`get` 与 `put` 都是 O(1)；`head` 侧是最近使用、`tail.prev` 是最久未用。

> 相关：[[Java 14 - HashMap 源码解析]]、[[Java 11 - Set与Map]]、[[Java 13 - fail-fast机制]]

---

## 十四、Java 中的序列化和反序列化是什么？

**回答重点**：**序列化**是把对象转换成字节序列，使其能落盘、跨网络传输或进缓存；**反序列化**是把字节序列还原成对象。Java 原生方案要求类实现 `Serializable` 标记接口，并用 `ObjectOutputStream` / `ObjectInputStream` 读写。

### 扩展知识

**1）为什么必须"压扁"对象**

JVM 里的对象是"立体"的：字段里存的是指向堆内存地址的引用（比如 `0x1234`），对象之间还可能有循环引用。这些地址只在当前进程有效，直接传给另一台机器毫无意义。序列化的本质就是把**对象图**转成自包含的字节序列（记录类型信息 + 字段值 + 引用关系），接收端据此重建一份等价的图结构。

**2）关键 API 与约束**

| 角色 | 说明 |
|---|---|
| `java.io.Serializable` | 空标记接口，无任何方法，仅用于让序列化机制放行 |
| `ObjectOutputStream.writeObject(obj)` | 序列化入口 |
| `ObjectInputStream.readObject()` | 反序列化入口 |
| `transient` | 修饰的字段不参与序列化，反序列化后为默认值（`0` / `null` / `false`） |
| `serialVersionUID` | 版本一致性校验用的"类指纹" |

`Serializable` 只是标记：`ObjectOutputStream` 检查到类未实现该接口时直接抛 `NotSerializableException`；`String`、数组、枚举等有各自的内置处理分支，不需要走普通对象路径。

**3）`serialVersionUID` 的真实作用**

```java
private static final long serialVersionUID = 1L;
```

- 数字本身没有业务含义，`1L` 和 IDE 生成的随机长整型等价，只要**序列化端与反序列化端一致**即可；
- 不显式声明时，编译器根据类名、字段、方法签名等信息自动生成一个指纹；
- 因此「先序列化存盘 → 改了类结构（加字段、改签名）→ 再反序列化」会因指纹变化抛 `InvalidClassException`。显式固定 `serialVersionUID` 就能让新增/删除字段保持兼容（新增字段读出来是默认值）。

**4）静态变量不参与序列化**

序列化保存的是**对象实例状态**，`static` 字段属于类而非对象，不会写入字节流；反序列化后读到的静态变量值是当前 JVM 中该类的实时值，而非当初序列化时的值。

**5）性能与替代方案**

JDK 原生序列化体积大（带完整类描述）、速度慢、只适用于 Java 生态，不适合大规模分布式调用。常见替代：

- **Protobuf**：IDL 定义 schema + 二进制编码，跨语言、体积小、性能高，gRPC 默认；
- **Kryo / Hessian**：面向 Java 的高性能二进制序列化，RPC 框架常用；
- **JSON（Jackson / Fastjson2）**：可读性好，跨语言，适合对外接口与调试场景。

**6）安全风险**

反序列化会依据字节流里的类描述**自动加载并构造对象**，攻击者可构造 gadget chain 在 `readObject` 阶段触发任意代码执行——这就是 Java 反序列化漏洞（Fastjson、Shiro、WebLogic 等历史上的 RCE 多源于此）。防护要点：

- 绝不对不可信输入做原生反序列化，优先用 JSON/Protobuf 这类数据契约明确的格式；
- 必须用时设置白名单：JDK 9+ 的 `ObjectInputFilter`（`jdk.serialFilter`）限制可反序列化的类；
- 敏感字段用 `transient` 排除，避免密码等随字节流外泄。

> 相关：[[Java 14 - NIO与序列化]]、[[Java 13 - File与IO流]]

---

## 十五、Java 并发库中提供了哪些线程池实现？它们有什么区别？

**回答重点**：`Executors` 工具类提供 5 种开箱即用的线程池：`FixedThreadPool`、`CachedThreadPool`、`SingleThreadExecutor`、`ScheduledThreadPool`、`WorkStealingPool`。它们的差别本质上是**核心线程数、最大线程数、任务队列类型、空闲回收策略**这四项参数的组合不同。

| 实现 | 线程数 | 任务队列 | 核心特点 | 适用场景 |
|---|---|---|---|---|
| `FixedThreadPool` | 固定 n（core = max = n） | 无界 `LinkedBlockingQueue` | 线程复用，超出并发数的任务排队等待 | 负载稳定、需要限制并发线程数 |
| `CachedThreadPool` | 0 ~ `Integer.MAX_VALUE` | `SynchronousQueue` | 空闲线程 60s 回收，无可用线程就新建 | 大量短耗时任务、任务量波动大 |
| `SingleThreadExecutor` | 1 | 无界 `LinkedBlockingQueue` | 单线程串行执行，天然保证提交顺序 | 需要严格顺序执行的任务 |
| `ScheduledThreadPool` | 指定 core，max 无上限 | `DelayedWorkQueue` | 支持延时 / 固定频率 / 固定间隔调度 | 定时任务、周期任务 |
| `WorkStealingPool`（JDK 8+） | 默认 CPU 核数个并行度 | 每线程一个双端队列 | 空闲线程从其他队列尾部"窃取"任务，自动负载均衡 | 大量小任务、递归分治（底层是 `ForkJoinPool`） |

### 扩展知识

**1）五个池子其实是同一套骨架**

除 `WorkStealingPool`（`ForkJoinPool`）外，其余四种最终都是 `ThreadPoolExecutor`，只是构造参数不同：

```java
// FixedThreadPool
new ThreadPoolExecutor(n, n, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
// CachedThreadPool
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<>());
// SingleThreadExecutor（core = max = 1）
new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
// ScheduledThreadPool
new ScheduledThreadPoolExecutor(corePoolSize);  // 内部用 DelayedWorkQueue
```

理解这一点，答题时就能从"背五个名字"升级为"讲一套调度模型"。

**2）任务提交后的流转顺序**

```
核心线程未满 → 创建核心线程执行
核心线程已满 → 任务入队等待
队列已满     → 创建非核心线程（直到 maximumPoolSize）
已达最大线程 → 交给拒绝策略
```

注意 `CachedThreadPool` 的 `SynchronousQueue` 不存储元素，所以"入队"必然失败，直接走"创建非核心线程"分支——这就是它"来一个任务开一个线程"的原因。

**3）生产环境不要直接用 `Executors`**

《阿里巴巴 Java 开发手册》明确禁止用 `Executors` 创建线程池，根因是**无界**：

- `FixedThreadPool` / `SingleThreadExecutor` 用无界 `LinkedBlockingQueue`，任务积压无上限 → `OutOfMemoryError: Java heap space`；
- `CachedThreadPool` / `ScheduledThreadPool` 的 `maximumPoolSize` 为 `Integer.MAX_VALUE`，线程无上限创建 → `OutOfMemoryError: unable to create new native thread`。

推荐显式 `new ThreadPoolExecutor(...)`，把七个参数都写清楚：

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    8,                              // corePoolSize
    16,                             // maximumPoolSize
    60L, TimeUnit.SECONDS,          // keepAliveTime
    new ArrayBlockingQueue<>(200),  // 有界队列，兜住背压
    new ThreadFactory() {           // 自定义线程名，便于排查
        private final AtomicInteger seq = new AtomicInteger(1);
        public Thread newThread(Runnable r) {
            return new Thread(r, "order-pool-" + seq.getAndIncrement());
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy()  // 拒绝策略：回退给调用线程
);
```

**4）线程数怎么估**

- **CPU 密集型**：`N + 1`（N 为核数），再多只会增加上下文切换；
- **IO 密集型**：`N × (1 + 平均等待时间 / 平均计算时间)`，或按并发连接数 × 单任务耗时 / 目标 QPS 反推；
- 最终以压测结果为准，不要迷信公式。

> 相关：[[Java 16 - 锁与并发工具]]、[[Java 15 - 线程基础]]、[[Java 51 - 并发编程学什么（学习框架）]]

---

## 十六、Java 的 synchronized 是怎么实现的？

**回答重点**：`synchronized` 依赖 JVM 的 **Monitor（监视器锁）** 与**对象头 Mark Word** 实现。修饰方法时在方法访问标志上加 `ACC_SYNCHRONIZED`（0x0020），修饰代码块时由编译器在同步块前后插入 `monitorenter` / `monitorexit` 字节码。二者本质相同：都要先拿到锁对象关联的 monitor 所有权才能进入临界区。

### 扩展知识

**1）对象内存布局与 Mark Word**

对象 = 对象头 + 实例数据 + 对齐填充；对象头 = Mark Word + Klass Pointer（+ 数组长度，仅数组有）。Mark Word 在 64 位下复用同一块内存表达不同状态，这是锁实现的核心：

| 锁状态 | Mark Word 内容 | 锁标志位 |
|---|---|---|
| 无锁 | hashCode(31) + 分代年龄(4) + 偏向位(1) | `01` |
| 偏向锁 | 线程 ID(54) + epoch(2) + 分代年龄 + 偏向位 `1` | `101` |
| 轻量级锁 | 指向栈帧 Lock Record 的指针(62) | `00` |
| 重量级锁 | 指向 `ObjectMonitor` 的指针(62) | `10` |
| GC 标记 | 空 | `11` |

**2）锁升级路径**

无锁 → 偏向锁 → 轻量级锁 → 重量级锁。HotSpot 中锁**只能升级不能降级**（偏向锁被撤销后即回到无锁/轻量级，不会退回偏向态），所谓「动态升降级」只是状态迁移的通俗说法。

**偏向锁**：第一个线程 CAS 把自己的线程 ID 写进 Mark Word、标志位置 `101`，后续同一线程进入只做「标志位 + 线程 ID + epoch 比对」，无 CAS 开销。另一个线程来竞争时需在**安全点**撤销偏向（遍历该类的实例改 epoch）。批量重偏向阈值 `-XX:BiasedLockingBulkRebiasThreshold`（默认 20），超过 `-XX:BiasedLockingBulkRevokeThreshold`（默认 40）则该类彻底失去偏向能力。**JDK 15 起偏向锁默认禁用（JEP 374）**，因为多核高并发下撤销/重偏向的开销常常盖过收益。

**轻量级锁**：在当前栈帧划出 Lock Record，把对象头 Mark Word 拷进 `displaced header`（dhw），再 CAS 让对象头指向该 Lock Record。重入时新栈帧的 dhw 写 `null`，解锁时据此判断是否重入；解锁用 CAS 把 dhw 换回对象头，失败说明有竞争 → 膨胀为重量级锁。适用场景是「多线程交替而非同时」访问同一把锁，避免系统调用。

**重量级锁**：竞争激烈时膨胀，线程真正阻塞，靠操作系统互斥量（`pthread_mutex_lock`）实现，涉及用户态/内核态切换，开销大。

**3）ObjectMonitor 内部结构（HotSpot C++ 实现）**

| 字段 | 含义 |
|---|---|
| `_owner` | 当前持有锁的线程 |
| `_recursions` | 重入次数 |
| `_cxq` | 单向链表，CAS 竞争的入口队列 |
| `_EntryList` | 双向链表，被唤醒后等待再次争锁的线程 |
| `_WaitSet` | 调用 `wait()` 的线程（条件等待队列） |
| `_count` | 关联的锁计数 |

- **加锁 `ObjectMonitor::enter`**：CAS 把 `_owner` 设为当前线程 → 成功即持锁，`_recursions++` 表示重入；失败则先**自适应自旋**，仍失败就包装成 `ObjectWaiter` 入 `_cxq`，最后 `park()` 阻塞。
- **解锁 `ObjectMonitor::exit`**：`_recursions--`，减到 0 才真正释放 `_owner`，并按 `QMode` 策略从 `_cxq` / `_EntryList` 迁移节点、唤醒等待线程。
- **`wait()`**：线程进 `_WaitSet`，随后调用 `exit` 释放锁；**`notify()`**：从 `_WaitSet` 取节点，按策略放入 `_cxq` 或 `_EntryList` 并唤醒；`notifyAll()` 即循环处理全部。
- **为什么两个等待队列**：`_cxq` 用单向链表 + CAS 扛住高并发入队，`_EntryList` 用双向链表在唤醒时批量搬迁节点，降低 `_cxq` 尾部竞争。

**4）自旋与锁优化**

自旋 = 空转 CPU 等锁，避免「刚阻塞就被唤醒」的无谓开销；自旋次数用**自适应自旋**（依据上一次在该锁上的自旋成功率动态调整）。另外还有：**锁消除**（逃逸分析判定对象不逃逸则去掉加锁）、**锁粗化**（把相邻的多次加解锁合并成一次）。**可重入性**由 `_recursions` 计数保证，取锁 +1、释放 -1，归零才真正释放。

> 相关：[[Java 29 - Synchronized 关键字详解]]、[[Java 16 - 锁与并发工具]]、[[Java 07 - 并发三大根源：可见性、原子性、有序性]]

---

## 十七、Java 的类加载过程是怎样的？

**回答重点**：类加载是把符合格式的二进制流读进内存、解析成可用 Class 对象的过程，分为**加载 → 连接（验证 / 准备 / 解析）→ 初始化**共 5 个阶段。二进制流不限于 `.class` 文件——字节码工具生成、网络传输的流，只要格式合法 JVM 都接受。

### 扩展知识

**1）五个阶段各做什么**

| 阶段 | 动作 |
|---|---|
| 加载 | 通过类的全限定名获取二进制流，转为方法区（JDK 8+ 为元空间）运行时数据结构，并在堆中生成 `Class` 对象作为访问入口 |
| 验证 | 校验字节码格式、元数据语义、字节码指令、符号引用合法性，确保不会危害 JVM |
| 准备 | 为**静态变量**分配内存并赋**零值**（`int` → 0、引用 → null）；`static final` 常量若为编译期常量则已在常量池，此阶段直接赋真值 |
| 解析 | 把常量池中的**符号引用**替换为**直接引用**（可理解为「计划里的代号 A」换成「真实的人」），可延迟到首次使用时进行 |
| 初始化 | 执行类构造器 `<clinit>()`，即静态变量赋代码中的真实值与静态代码块，父类先于子类 |

**2）触发初始化的典型时机**（主动引用）

`new` / 读写静态字段 / 调用静态方法、反射（`Class.forName`）、初始化子类时先初始化父类、含 `main` 方法的主类、`MethodHandle` 解析结果对应的类。

**3）双亲委派模型**

类加载请求先委派给父加载器，父加载器无法完成时才由当前加载器自己加载。好处有二：**避免类重复加载**（同一个类只被加载一次）、**保护核心类库**（自定义的 `java.lang.String` 永远轮不到被加载，防止核心 API 被篡改）。

**4）类加载器层次**

| 加载器 | 职责 |
|---|---|
| Bootstrap ClassLoader | C++ 实现，加载 `JAVA_HOME/lib` 下的核心类（`rt.jar` / `java.base` 等） |
| Extension / Platform ClassLoader | JDK 9 前为 `ExtClassLoader`（`lib/ext`），JDK 9+ 改为 `PlatformClassLoader` |
| Application ClassLoader | 加载 classpath（含自定义类、第三方 jar），默认的 `ClassLoader.getSystemClassLoader()` |
| 自定义 ClassLoader | 继承 `ClassLoader` 重写 `findClass`，用于热部署、加密类、隔离加载 |

**5）打破双亲委派的场景**

- **SPI**：JDBC `DriverManager` 由 Bootstrap 加载，却要加载厂商实现类 → 用线程上下文类加载器 `Thread.currentThread().getContextClassLoader()` 反向委派。
- **Tomcat**：每个 Web 应用一个 `WebAppClassLoader`，优先自己加载应用内的类，实现应用间隔离与热部署。
- **OSGi**：网状委派，按包级别精确控制可见性。

> 相关：[[Java 18 - 类加载机制]]、[[Java 17 - JVM内存模型与GC]]、[[Java 02 - JDK JRE JVM]]

---

## 十八、Java 线程池有哪些拒绝策略？

**回答重点**：`ThreadPoolExecutor` 内置 4 种拒绝策略，在**核心线程已满 + 队列已满 + 线程数达最大值**时触发。默认是 `AbortPolicy`（抛异常）。

| 策略 | 行为 | 适用场景 |
|---|---|---|
| `AbortPolicy`（默认） | 抛 `RejectedExecutionException`，任务丢失 | 必须让调用方感知失败的场景（订单、支付） |
| `CallerRunsPolicy` | 由提交任务的线程自己执行该任务 | 希望反压、减缓提交速度以稳定系统 |
| `DiscardOldestPolicy` | 丢弃队列中最旧的任务，再重新提交当前任务 | 只关心最新数据（实时行情、监控采样） |
| `DiscardPolicy` | 静默丢弃当前任务，不抛异常 | 任务可丢、允许丢失的场景（埋点日志） |

### 扩展知识

**1）自定义拒绝策略**：实现 `RejectedExecutionHandler`，可做日志、落库、降级重排、阻塞提交等。

```java
public class CustomRejectedExecutionHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println("Task " + r + " rejected");   // 生产环境应改为日志 + 告警
        // 也可以：降级执行 / 写入 MQ 稍后重试 / 阻塞等待队列有空位
    }
}

ThreadPoolExecutor pool = new ThreadPoolExecutor(
        4, 8, 60L, TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(100),
        Executors.defaultThreadFactory(),
        new CustomRejectedExecutionHandler());
```

**2）三个常见坑**

- **拒绝策略根本不触发**：`Executors.newFixedThreadPool` / `newSingleThreadExecutor` 用的是**无界** `LinkedBlockingQueue`，队列永不满 → 任务无限堆积直到 OOM。生产上应手动 `new ThreadPoolExecutor` 并指定有界队列。
- **`CallerRunsPolicy` 会阻塞提交线程**（如 Tomcat 的 IO 线程），吞吐下降但起到天然限流作用；反之若提交线程也持有锁，可能引发死锁。
- **优先队列下 `DiscardOldestPolicy` 丢的是「优先级最高」的任务**：它 poll 的是队列头部，而 `PriorityBlockingQueue` 头部正是优先级最高的元素，语义与名字相反。

> 相关：[[Java 15 - 线程基础]]、[[Java 16 - 锁与并发工具]]、[[Java 51 - 并发编程学什么（学习框架）]]

---

## 十九、Spring 如何解决循环依赖？

**回答重点**：核心是**提前暴露尚未初始化完成的 Bean**——Bean 实例化完成后，立刻把「能生产该 Bean 的对象工厂」放进三级缓存；当别的 Bean 反过来依赖它时，就能拿到这个半成品，从而打破「A 等 B、B 等 A」的死循环。三个 Map 分工：`singletonObjects`（一级，完整单例）、`earlySingletonObjects`（二级，已实例化未初始化）、`singletonFactories`（三级，对象工厂，用于延迟生成早期 Bean，尤其要生成 AOP 代理时）。

### 三个缓存的分工

| 缓存 | 字段 | 存放内容 |
|---|---|---|
| 一级 | `singletonObjects` | 完全初始化完成的单例 Bean |
| 二级 | `earlySingletonObjects` | 已实例化、但未完成属性注入与初始化的「半成品」 |
| 三级 | `singletonFactories` | `ObjectFactory`，调用 `getObject()` 才真正产出早期 Bean（可在此织入 AOP 代理） |

### 一次 A ↔ B 循环依赖的完整流程

1. `getBean(A)` → 一级缓存没有 → 标记 A「创建中」→ 进入 `doCreateBean(A)`
2. **实例化** A（`createBeanInstance`）后，立刻把工厂塞进三级缓存——此时 Spring 并不知道会不会发生循环依赖，但它一律先暴露
3. **属性注入** A 时发现依赖 B → `getBean(B)`，B 走一遍同样流程，实例化后也暴露自己的工厂
4. B 注入属性时 `getBean(A)`：一级没有，且 A 正在创建中 → 查二级（空）→ 查三级拿到工厂 → `getObject()` 得到 A（需要 AOP 时在这一步生成代理）→ 工厂从三级删除，A 放入二级缓存
5. B 完成初始化进入一级缓存，返回 A 的属性注入；A 注入 B 后完成初始化，进入一级缓存并清除二级缓存中的自己

关键一行源码（`AbstractAutowireCapableBeanFactory#doCreateBean`）：

```java
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

`getSingleton(beanName, allowEarlyReference)` 的查找顺序就是一级 → 二级 → 三级；注意若第一步发现 Bean **不在创建中**，直接返回 `null`，不再继续查二级、三级。

### 能解决循环依赖的两个前提

1. 相互依赖的 Bean 必须都是**单例**
2. 注入方式**不能全是构造器注入**，且 beanName 字母序在前的那个不能是构造器注入

### 为什么必须单例、为什么不能全构造器注入

- **原型（prototype）不行**：创建 A1 要新 B1，创建 B1 又要新 A2，A2 又要 B2……无限递归，源码里 `isPrototypeCurrentlyInCreation` 判断会直接抛 `BeanCurrentlyInCreationException`。单例则「B 要的是同一个 A」，Spring 才有操作空间。
- **Bean 创建三步**：实例化 `createBeanInstance` → 属性注入 `populateBean` → 初始化 `initializeBean`（Aware 回调、init-method、AOP 代理）。构造器注入要求在**实例化阶段**就拿到依赖，而此时 A 还没进三级缓存，B 找不到「半成品 A」，双方卡死。
- **一个 set 一个构造器**：A 用 set 注入 B、B 用构造器注入 A → **成功**（A 先实例化并暴露，B 构造时能从 Map 取到 A）；反过来 A 用构造器注入 B、B 用 set 注入 A → **失败**。原因是 Spring 按 **beanName 字母序**创建 Bean，A 永远排在 B 前，字母序在前的那个必须允许「先实例化、后注入」。

### 实践建议

- 循环依赖多说明职责划分有问题，优先重构（抽公共依赖、引入事件/回调）而不是依赖三级缓存兜底。
- Spring Boot 2.6+ 默认 `spring.main.allow-circular-references=false`，循环依赖会直接启动失败；老项目升级时若报此错，先查依赖环再考虑临时打开开关。
- 构造器注入 + `@Lazy` 也能打破环（注入代理，真正使用时才初始化）。

> 相关：[[Java 31 - AOP与自定义注解实践]]、[[Java 25 - Spring Boot RESTful 接口实现与测试]]

---

## 二十、Synchronized 和 ReentrantLock 有什么区别？

**回答重点**：`synchronized` 是 Java 内置关键字，自动加锁/释放，非公平、不可中断、不支持超时与多条件；`ReentrantLock` 是 JUC 提供的 API 级锁（JDK 1.5 引入），支持超时、可中断、公平锁、多条件队列，但必须手动 `unlock()`。两者**都是可重入锁**；JDK 1.6 优化后性能基本持平——日常用 `synchronized` 更简单，需要高级能力时再用 `ReentrantLock`。

| 维度 | `synchronized` | `ReentrantLock` |
|---|---|---|
| 层次 | JVM 关键字，字节码 `monitorenter/monitorexit` | JDK 类库，基于 AQS |
| 释放 | 代码块结束/异常自动释放 | 必须 `finally` 中手动 `unlock()` |
| 公平性 | 只有非公平 | 可选公平 / 非公平 |
| 可中断 | 不支持 | `lockInterruptibly()` 支持 |
| 超时 | 不支持 | `tryLock(timeout, unit)` 支持，可规避死锁 |
| 条件队列 | 单条件（`wait/notify`） | 多 `Condition`，可分组精确唤醒 |
| 可重入 | 支持 | 支持 |

### 可重入锁

同一线程持有某把锁时可以再次获取同一把锁而不死锁。递归调用、外层方法调用内层同步方法时必需：

```java
public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock();

    public void outer() {
        lock.lock();
        try {
            inner();               // 同一线程再次拿同一把锁
        } finally {
            lock.unlock();
        }
    }

    public void inner() {
        lock.lock();
        try {
            // critical section
        } finally {
            lock.unlock();
        }
    }
}
```

**实现方式**：核心是计数器——线程抢到锁计数 +1，重入再 +1；每次释放 -1，减到 0 才真正释放锁。`ReentrantLock` 用 AQS 的 `state` 字段承担这个计数，并记录 `exclusiveOwnerThread`。

### synchronized 在 JDK 1.6 后的优化

- **偏向锁**：同一线程反复获取同一把锁时，把锁标记为偏向该线程，后续进入几乎零开销（JDK 15 起默认禁用并废弃，因维护成本高于收益）。
- **轻量级锁**：无竞争时用 CAS 把 Mark Word 指向栈上的 Lock Record，避免走操作系统互斥量。
- **锁粗化**：把短时间内连续的加解锁合并成一次更大范围的锁，减少开销。
- **锁消除**：JIT 通过逃逸分析判定锁对象不会被其他线程访问，直接去掉同步。

锁状态升级路径：无锁 → 偏向锁 → 轻量级锁（CAS 自旋）→ 重量级锁（OS mutex，线程挂起）。

### 选型建议

- 默认 `synchronized`：代码短、不会忘记解锁、异常安全，JVM 还会持续优化。
- 需要 `tryLock` 超时避免死锁、需要响应中断取消任务、需要公平锁、需要多个 `Condition` 分组唤醒（如生产者/消费者分别唤醒）时用 `ReentrantLock`。
- 用 `ReentrantLock` 时 `lock()` 要写在 `try` **之前**，`unlock()` 放在 `finally`；写在 try 内部一旦加锁失败，finally 里的 `unlock` 会抛 `IllegalMonitorStateException`。

> 相关：[[Java 16 - 锁与并发工具]]、[[Java 15 - 线程基础]]、[[Java 51 - 并发编程学什么（学习框架）]]

---

## 二十一、为什么 HashMap 的容量必须是 2 的 n 次方？

**回答重点**：为了让 `(n - 1) & hash` 能替代 `hash % n` 定位桶——**位运算比取模快得多**；同时 `n - 1` 的低位全为 1，能让哈希码的低位充分参与运算，分布更均匀；扩容时还能只靠「原容量那一位」判断元素留在原位还是迁移，省去重新计算哈希。

### 为什么 (n - 1) & hash 分布均匀

`n = 64` 时二进制是 `0100 0000`，`n - 1 = 0011 1111`，低位全 1，与哈希码做 `&` 能完整保留哈希低位的随机性；若直接用 `0100 0000` 去与，结果低位恒为 0，所有元素只能落在极少数桶里，冲突爆炸。

### 位运算 vs 取模

按哈希码算下标，直觉写法是 `hash % n`（如长度 5 就 `% 5`），但除法指令开销明显高于位与。当且仅当 `n` 是 2 的幂时成立：

```
a % (2^n) == a & (2^n - 1)
```

这也是 `HashMap` 默认容量 16、`tableSizeFor()` 把任意初始容量向上取整到 2 的幂的原因。

### 扩容时的便利

容量 16 → 32 时，元素新下标只可能是「原下标」或「原下标 + 16」，判断依据就是哈希值在**原容量那一位**上是 0 还是 1（`e.hash & oldCap`）：0 留在低位链（lo），1 迁到高位链（hi）。因此扩容不需要重算哈希、不需要重新取模，JDK 1.8 直接把一条链表拆成 lo/hi 两条（红黑树同理拆分），这也是「2 的幂」带来的实际性能收益。

> 相关：[[Java 14 - HashMap 源码解析]]

---

## 二十二、为什么 JDK 1.8 对 HashMap 进行了红黑树的改动？

**回答重点**：链表只能顺序遍历，一旦哈希冲突集中到同一个桶，查找/插入/删除就从 O(1) 退化为 O(n)。JDK 1.8 因此在**链表长度 ≥ 8 且数组容量 ≥ 64** 时把链表树化为红黑树，最坏情况降到 O(log n)；节点数 ≤ 6 时再退回链表，用 6/8 之间的差值留出缓冲，避免反复树化与反树化。

### 树化的两个门槛

- 阈值常量：`TREEIFY_THRESHOLD = 8`、`UNTREEIFY_THRESHOLD = 6`、`MIN_TREEIFY_CAPACITY = 64`
- `putVal` 里 `binCount` 从 0 开始累加，`binCount >= 7`（即桶内已有 8 个节点）才进入 `treeifyBin`
- `treeifyBin` 内部还要再判一次数组长度：`tab.length < MIN_TREEIFY_CAPACITY` 时**只扩容、不树化**（`resize()` 后冲突自然被摊开）

### 为什么容量不到 64 不树化

1. **避免白做工**：小数组冲突多，但一扩容元素就散到新桶里，刚树化完就扩容，树化开销全浪费。
2. **省内存**：`TreeNode` 比普通 `Node` 多出父/左/右/前驱指针，内存约为普通节点的两倍，小规模数据下不划算。

### 为什么阈值偏偏是 8

源码注释给了概率依据：在负载因子 0.75 的假设下，一个桶内节点数达到 8 的概率约为 `0.00000006`（千万分之一）。也就是说正常情况下几乎不会树化，红黑树只是**兜底**——防劣质 `hashCode()` 或刻意构造的哈希碰撞攻击把查询拖成 O(n)。

### 为什么退化阈值是 6 而不是 8

留缓冲区防「反复横跳」：若树化与退化都用 8，元素在 8/9 之间增删就会不停地在链表和红黑树之间转换，而两个方向的转换都有开销。取 6 后，只有真正删到足够少才退化。

### 为什么不干脆全部用红黑树

`TreeNode` 内存约为 `Node` 的两倍，而绝大多数桶只有 0~1 个元素；短链表遍历本身很快且占用小。用「链表为主 + 红黑树兜底」是时间与空间的折中。

> 相关：[[Java 14 - HashMap 源码解析]]

---

## 二十三、为什么 Java 中 HashMap 的默认负载因子是 0.75？

**回答重点**：负载因子 = 元素数量 / 容量，决定「装多满就扩容」。0.75 是**时间与空间的折中值**：调低则扩容频繁（rehash 开销大、空间浪费），调高则哈希冲突增多（桶内链表/红黑树变长，查询退化），0.75 在两者之间取得较好平衡。

### 扩容何时触发

- 阈值 `threshold = capacity × loadFactor`，`size > threshold` 时扩容为两倍并迁移元素
- 例：容量 16、负载因子 0.75 → 阈值 12，存入第 13 个元素时扩容
- 无参构造只记录 `loadFactor = 0.75`，数组容量延迟到**首次 put** 才分配为 16（懒加载）

### 为什么不是 1.0 或 0.5

| 负载因子 | 空间利用率 | 冲突概率 | 代价 |
|---|---|---|---|
| 0.5 | 低（一半空间闲置） | 低，查询快 | 扩容频繁、rehash 开销大 |
| **0.75** | 较高 | 可接受 | 平衡点 |
| 1.0 | 最高 | 高，桶内链表/红黑树变长 | 查询最坏退化到 O(n) |

0.75 还是二进制可精确表示的值（3/4）：容量为 2 的幂时 `capacity × 0.75` 必为整数，阈值计算不会因浮点舍入产生偏差。

### 场景化调整

- **读多写少 / 延迟敏感**：可降到 0.5，冲突更少、查询更快，代价是更多扩容
- **内存受限**：可升到 0.85 甚至更高，减少扩容次数与空桶，代价是冲突与查询开销上升
- 注意负载因子只能在**构造时**传入，运行中无法修改；已知元素规模时更有效的做法是预设初始容量（`expectedSize / 0.75 + 1`），一次性分配到位、彻底避免扩容

> 相关：[[Java 14 - HashMap 源码解析]]

---

## 二十四、为什么 Spring 循环依赖需要三级缓存，二级不够吗？

**回答重点**：二级缓存足以**打破循环依赖**，但解决不了 **AOP 代理的生成时机**。若二级缓存里只放原始对象，其他 Bean 注入到的就是未代理的「裸对象」；若实例化后立刻把代理对象放进二级缓存，又等于在 Bean 初始化之前就生成了代理，违背 Bean 生命周期。三级缓存存的是 `ObjectFactory`，**只有真的发生循环依赖时才调用它**去提前暴露（必要时生成代理），没发生依赖就永不调用，生命周期因此不被破坏。

### 二级缓存方案的两个死结

1. **放原始对象**：B 注入到的是未经 AOP 增强的 A，事务、日志、权限等切面全部失效，且最终一级缓存里是代理对象、B 手里却是原始对象，同一 Bean 出现两个身份。
2. **直接放代理对象**：看似可行，但代理是在**初始化后期**由后置处理器（`AbstractAutoProxyCreator#wrapIfNecessary`）生成的；实例化完就代理，等于对所有 Bean 无条件提前代理——既违背生命周期，又对没有循环依赖的 Bean 白付代理成本。

### 工厂里到底做了什么

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanProcessorCache().smartInstantiationAware) {
            exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
        }
    }
    return exposedObject;
}
```

- 没有可用的 `SmartInstantiationAwareBeanPostProcessor` → 原样返回传入的 bean（不代理）
- 有则交给它决定，`AnnotationAwareAspectJAutoProxyCreator` 在这里判断是否需要代理、需要就返回代理对象
- 也就是说：**是否代理由工厂在「被真正需要」的那一刻决定**，而不是在实例化时盲猜

### 缓存不会堆积

单例 Bean 创建完成后会放入一级缓存 `singletonObjects`，同时从二级 `earlySingletonObjects`、三级 `singletonFactories` 中移除自己。没有循环依赖时三级缓存里的工厂从未被调用，随 Bean 完成即清理，不存在长期占用。

### 与「十九」的分工

「十九」讲循环依赖的整体流程与前提（单例、非全构造器注入、beanName 字母序），本节只回答**「两级为什么不够」**这一个追问点；两节配合看即可。

> 相关：[[Java 31 - AOP与自定义注解实践]]

> 来源：鱼皮·编程导航 / codefather
