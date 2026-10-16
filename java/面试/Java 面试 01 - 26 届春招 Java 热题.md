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

> 来源：鱼皮·编程导航 / codefather
