# Java 高并发与多线程深度指南

> 综合《Java并发编程的艺术》（方腾飞、魏鹏、程晓明）与《Java Concurrency in Practice》（Brian Goetz 等，即 JCIP）两本经典著作的核心知识体系。
>
> 《并发编程的艺术》侧重底层实现原理与源码分析（JMM、volatile、synchronized、AQS、线程池、并发容器）；《并发编程实战》侧重设计原则与工程实践（线程安全、对象的共享与组合、安全发布、取消与关闭、活跃性、性能与可伸缩性）。两者结合，既有"底层如何工作"的深刻理解，又有"上层如何设计"的工程智慧。

---

## 目录

1. [并发编程基础哲学](#1-并发编程基础哲学)
2. [Java内存模型JMM](#2-java内存模型jmm)
3. [三大核心问题：原子性、可见性、有序性](#3-三大核心问题原子性可见性有序性)
4. [volatile、synchronized 与 final 的内存语义](#4-volatilesynchronized-与-final-的内存语义)
5. [线程安全性（JCIP）](#5-线程安全性jcip)
6. [对象的共享与安全发布（JCIP）](#6-对象的共享与安全发布jcip)
7. [对象的组合（JCIP）](#7-对象的组合jcip)
8. [AQS 框架深入](#8-aqs-框架深入)
9. [Lock 体系](#9-lock-体系)
10. [并发容器](#10-并发容器)
11. [线程池深入](#11-线程池深入)
12. [同步工具类](#12-同步工具类)
13. [原子操作与 CAS](#13-原子操作与-cas)
14. [CompletableFuture 异步编排](#14-completablefuture-异步编排)
15. [取消与关闭（JCIP）](#15-取消与关闭jcip)
16. [活跃性危险（JCIP）](#16-活跃性危险jcip)
17. [性能与可伸缩性（JCIP）](#17-性能与可伸缩性jcip)
18. [并发程序测试（JCIP）](#18-并发程序测试jcip)

---

## 1. 并发编程基础哲学

### 1.1 JCIP 核心观点

**编写线程安全代码的核心在于：对共享可变状态的访问进行管理。**（JCIP 全书基石）

- **共享**：多个线程访问
- **可变**：变量的值在其生命周期内可以变化
- 如果消除以上任意一个条件（不共享、不可变、访问受管理），就能实现线程安全

**无状态的对象总是线程安全的。** 既不包含任何域，也不包含任何对其他类中域的引用的对象，没有状态可供共享。

### 1.2 并发编程三大核心问题

| 问题 | 含义 | 根源 | Java解决手段 |
|------|------|------|-------------|
| **原子性** | 一个操作不可中断，要么全部执行，要么都不执行 | 线程上下文切换导致操作中途被打断 | synchronized、Lock、原子类（CAS） |
| **可见性** | 一个线程修改共享变量后，其他线程能立即看到最新值 | CPU缓存（工作内存）与主内存不一致 | volatile、synchronized、final、Lock |
| **有序性** | 程序执行的顺序按照代码的先后顺序执行 | 编译器重排序、处理器重排序 | volatile、synchronized、happens-before |

---

## 2. Java内存模型（JMM）

### 2.1 JMM 抽象结构

JMM（Java Memory Model）定义了线程和主内存之间的抽象关系：

- **主内存（Main Memory）**：所有变量存储的区域
- **工作内存（Working Memory）**：每个线程私有的缓存区域，保存了该线程使用到的变量的主内存副本拷贝

线程对变量的所有操作（读取、赋值）都必须在工作内存中进行，不能直接读写主内存。不同线程之间无法直接访问对方的工作内存，线程间变量值的传递均需要通过主内存来完成。

```
线程A ──→ 工作内存A ──┐
                     ├─→ 主内存（共享变量）
线程B ──→ 工作内存B ──┘
```

### 2.2 重排序

重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序。分为三种类型：

1. **编译器重排序**：不改变单线程程序语义的前提下，重新安排语句的执行顺序
2. **指令级并行重排序**：处理器采用指令级并行技术将多条指令重叠执行，不存在数据依赖的指令可重排序
3. **内存系统重排序**：处理器使用缓存和读写缓冲区，使得加载和存储操作看上去是乱序执行的

**as-if-serial 语义**：不管怎么重排序，单线程程序的执行结果不能被改变。编译器和处理器对存在数据依赖关系的操作都不会做重排序。

### 2.3 happens-before 原则（8条规则）

happens-before 是 JMM 最核心的概念，是判断数据是否存在竞争、线程是否安全的主要依据。

| 规则 | 内容 |
|------|------|
| **程序次序规则** | 一个线程中的每个操作，happens-before 于该线程中的任意后续操作 |
| **监视器锁规则** | 对一个锁的解锁，happens-before 于随后对这个锁的加锁 |
| **volatile变量规则** | 对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读 |
| **传递性** | 如果 A happens-before B，且 B happens-before C，那么 A happens-before C |
| **线程启动规则** | Thread 对象的 start() 方法 happens-before 于该线程的每一个动作 |
| **线程终止规则** | 线程中的所有操作都 happens-before 于线程的终止检测（Thread.join() 成功返回） |
| **线程中断规则** | 对线程 interrupt() 的调用 happens-before 于被中断线程检测到中断事件 |
| **对象终结规则** | 一个对象的初始化完成（构造函数执行结束）happens-before 于它的 finalize() 方法的开始 |

> **as-if-serial vs happens-before**：as-if-serial 保证单线程内程序的执行结果不被改变；happens-before 保证正确同步的多线程程序的执行结果不被改变。两者都是为了在不改变程序执行结果的前提下，尽可能提高程序执行的并行度。

### 2.4 内存屏障

内存屏障（Memory Barrier）是一组处理器指令，用于实现对内存操作的顺序限制。JMM 把内存屏障指令分为四类：

| 屏障类型 | 指令示例 | 说明 |
|----------|----------|------|
| **LoadLoad** | Load1; LoadLoad; Load2 | 确保 Load1 数据的装载先于 Load2 及后续装载指令 |
| **StoreStore** | Store1; StoreStore; Store2 | 确保 Store1 数据对其他处理器可见（刷新到内存）先于 Store2 及后续存储指令 |
| **LoadStore** | Load1; LoadStore; Store2 | 确保 Load1 数据装载先于 Store2 及后续存储指令刷新到内存 |
| **StoreLoad** | Store1; StoreLoad; Load2 | 确保 Store1 数据对其他处理器可见（刷新到内存）先于 Load2 及后续装载指令。全能屏障，开销最大 |

**volatile 的内存屏障插入策略**（保守策略）：
- 在每个 volatile 写操作的前面插入一个 StoreStore 屏障
- 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障
- 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障
- 在每个 volatile 读操作的后面插入一个 LoadStore 屏障

---

## 3. 三大核心问题：原子性、可见性、有序性

### 3.1 原子性

原子操作是不可分割的，在多线程环境下不会被线程调度机制打断。

**Java 中的原子性保证**：
- 基本数据类型的读取和赋值（除 long/double 外）是原子操作
- 64 位的 long/double 在没有 volatile 修饰时，读写可能分两次 32 位操作（JMM 允许非原子处理，但现代 64 位 JVM 通常已保证原子性）
- `synchronized` 块内的操作整体具有原子性
- `java.util.concurrent.atomic` 包提供原子操作类

### 3.2 可见性

可见性是指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。

**Java 中实现可见性的方式**：
1. **volatile**：写 volatile 变量会立即刷新到主内存，读 volatile 变量会使工作内存失效，从主内存重新读取
2. **synchronized**：解锁前将工作内存中共享变量的最新值刷新到主内存；加锁时清空工作内存中共享变量的值，从而需要从主内存重新读取
3. **final**：被 final 修饰的字段在构造函数中一旦初始化完成，且构造函数没有把 this 引用逸出，那么其他线程就能看到 final 字段的值（初始化安全性）
4. **Lock**：与 synchronized 内存语义类似

### 3.3 有序性

Java 程序中天然的有序性：
- 在本线程内观察，所有操作都是有序的（线程内表现为串行的语义 as-if-serial）
- 在一个线程中观察另一个线程，所有操作都是无序的（指令重排序 + 工作内存与主内存同步延迟）

**实现有序性的手段**：
- `volatile`：禁止指令重排序
- `synchronized`：一个变量在同一时刻只允许一条线程对其进行 lock 操作

---

## 4. volatile、synchronized 与 final 的内存语义

### 4.1 volatile

**特性**：
1. **可见性**：对 volatile 变量的读，总是能看到（任意线程）对这个 volatile 变量最后的写入
2. **有序性**：禁止指令重排序优化

**volatile 写-读的内存语义**：
- 线程 A 写一个 volatile 变量，实质上是线程 A 向接下来将要读这个 volatile 变量的某个线程发出了（其对共享变量所做修改的）消息
- 线程 B 读一个 volatile 变量，实质上是线程 B 接收了之前某个线程写一个 volatile 变量发出的消息
- 线程 A 写 volatile 变量，线程 B 读 volatile 变量，这两个线程之间的 happens-before 关系成立

> volatile 只能保证单个读/写操作的原子性，对于 `volatile++` 这种复合操作不具有原子性。

### 4.2 synchronized

**synchronized 的三种使用方式**：
1. 修饰实例方法：锁当前实例对象（`this`）
2. 修饰静态方法：锁当前类的 Class 对象
3. 修饰代码块：锁指定的对象

**synchronized 的内存语义**：
- 当线程释放锁（monitorexit）时，JMM 会把该线程对应的工作内存中的共享变量值刷新到主内存中
- 当线程获取锁（monitorenter）时，JMM 会把该线程对应的工作内存置为无效，从而使得被监视器保护的临界区代码必须从主内存中读取共享变量

**锁的底层实现**：
- Java 对象头 Mark Word 中记录锁状态
- JDK6 之前的 synchronized 是重量级锁，直接调用操作系统 Mutex Lock
- JDK6 引入了**锁升级**机制

### 4.3 锁升级（《并发编程的艺术》核心）

JVM 在 JDK6 中引入了偏向锁和轻量级锁，锁状态从低到高：

```
无锁 → 偏向锁 → 轻量级锁（CAS自旋） → 重量级锁（操作系统Mutex）
```

| 锁状态 | 适用场景 | 实现原理 | 开销 |
|--------|----------|----------|------|
| **无锁** | 没有竞争 | CAS 操作 | 最低 |
| **偏向锁** | 只有一个线程访问同步块 | 对象头 Mark Word 记录线程 ID，同一线程再次进入无需 CAS | 极低 |
| **轻量级锁** | 多个线程交替访问（无实际竞争） | CAS 自旋获取锁，线程栈中创建 Lock Record 复制 Mark Word | 较低 |
| **重量级锁** | 多个线程同时竞争 | 操作系统 Mutex，线程阻塞和唤醒需要用户态/内核态切换 | 高 |

**锁升级流程**：
1. 对象刚创建时是无锁状态
2. 第一个线程获取锁时，升级为**偏向锁**，Mark Word 记录线程 ID
3. 第二个线程来竞争时，偏向锁撤销，升级为**轻量级锁**，线程自旋 CAS 获取
4. 自旋超过一定次数（默认10次，或自适应自旋）或有第三个线程竞争，升级为**重量级锁**

> **自适应自旋**：JDK6 引入，自旋时间不固定，由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机认为这次自旋也很可能成功，进而允许更长时间的自旋。

### 4.4 final 的内存语义

final 域具有**初始化安全性**，确保一旦对象构造完成，其他线程就能看到 final 字段正确初始化后的值，而无需同步。

**重排序规则**：
1. **写 final 域的重排序规则**：JMM 禁止把 final 域的写重排序到构造函数之外。编译器会在 final 域的写之后，构造函数 return 之前插入一个 StoreStore 屏障，确保 final 域在其他线程可见之前已经正确初始化
2. **读 final 域的重排序规则**：初次读一个包含 final 域的对象的引用，与随后初次读这个 final 域，这两个操作之间不能重排序。编译器会在读 final 域操作的前面插入一个 LoadLoad 屏障

**关键前提**：构造函数中不能让 this 引用逸出。如果在构造函数中注册事件监听器、启动线程或将 this 赋值给外部变量，可能导致其他线程看到未完全构造的对象。

```java
// 错误：this 引用逸出
public class ThisEscape {
    public ThisEscape(EventSource source) {
        source.registerListener(new EventListener() {
            public void onEvent(Event e) {
                doSomething(e); // 内部类隐式持有 ThisEscape.this
            }
        });
        // 初始化工作还没完成，this 已经逸出
    }
}

// 正确：使用私有构造函数 + 工厂方法
public class SafeListener {
    private final EventListener listener;
    private SafeListener() {  // 私有构造
        listener = new EventListener() {
            public void onEvent(Event e) { doSomething(e); }
        };
    }
    public static SafeListener newInstance(EventSource source) {
        SafeListener safe = new SafeListener();
        source.registerListener(safe.listener); // 构造完成后再发布
        return safe;
    }
}
```

---

## 5. 线程安全性（JCIP）

### 5.1 什么是线程安全

> **线程安全**：当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

更准确地说：一个对象是线程安全的，当调用这个对象的任意方法序列（包括对公共方法的调用以及对公共字段的读写）都不会违反它的任何不变量或后置条件。

### 5.2 原子性：竞态条件

**竞态条件（Race Condition）**：计算的正确性取决于多个线程的交替执行时序。

最常见的竞态条件是**检查-再-执行（Check-Then-Act）**：
```java
if (value == null) {      // 检查
    value = compute();    // 再执行
}
```

另一个典型是**读取-修改-写入（Read-Modify-Write）**：`value++`，实际上是三个步骤：读取 value → 值加1 → 写回 value。

### 5.3 内置锁 synchronized

Java 提供了内置的锁机制来支持原子性：`synchronized` 代码块。

- 每个 Java 对象都可以用作一个实现同步的锁，这些锁被称为**内置锁（Intrinsic Lock）**或**监视器锁（Monitor Lock）**
- 内置锁是**可重入的**：如果某个线程试图获得一个已经由它自己持有的锁，那么这个请求就会成功
- 重入意味着获取锁的操作粒度是"线程"，而不是"调用"

```java
// 可重入示例：子类调用父类的同步方法不会死锁
class Widget {
    public synchronized void doSomething() { }
}
class LoggingWidget extends Widget {
    public synchronized void doSomething() {
        super.doSomething(); // 可以重入成功
    }
}
```

### 5.4 用锁保护状态

对于可能被多个线程同时访问的可变状态变量，**在访问它时都需要持有同一个锁**。在这种情况下，我们称状态变量是由这个锁保护的。

```java
@GuardedBy("this") private int value; // 表示 value 受 this 锁保护
```

**每个共享的可变变量都应该只由一个锁来保护**，从而使维护人员知道是哪一个锁。

### 5.5 活跃性与性能

过于粗粒度的同步会导致性能问题：
- 多个线程排队执行同步代码块，丧失了并发性
- 同步代码块内如果包含耗时操作（I/O、远程调用），会严重降低吞吐量

**原则**：同步代码块内应该只包含必要的操作，将不影响共享状态且可能耗时的操作移出同步块。

---

## 6. 对象的共享与安全发布（JCIP）

### 6.1 发布与逸出

**发布（Publish）**一个对象是指，使对象能够在当前作用域之外的代码中使用。例如：
- 将对象的引用存储到公共静态变量中
- 从非私有方法中返回对象引用
- 将对象引用传递给外部方法

**逸出（Escape）**：当某个不应该被发布的对象被发布时，这种情况称为逸出。

**常见逸出形式**：
1. 发布内部可变状态（返回私有数组引用）
2. 发布内部类实例导致 this 引用逸出（见 4.4 节）
3. 从构造函数中注册监听器或启动线程

### 6.2 安全发布模式

要安全地发布一个对象，对象的引用以及对象的状态必须同时对其他线程可见。一个正确构造的对象可以通过以下方式安全发布：

| 发布方式 | 说明 | 示例 |
|----------|------|------|
| **静态初始化** | 由 JVM 在类的初始化阶段执行，JVM 内部有同步机制 | `public static Holder holder = new Holder(42);` |
| **volatile 引用** | volatile 保证引用的可见性，配合不可变对象效果更佳 | `private volatile Holder holder;` |
| **final 域** | 正确构造对象的 final 域中的引用 | 容器对象构造完成前，引用不可见 |
| **锁保护** | 将引用保存在由锁保护的域中 | `synchronized`  getter/setter |

### 6.3 不可变对象（Immutable）

**不可变对象一定是线程安全的。**

对象不可变需满足三个条件：
1. 对象创建以后其状态就不能修改
2. 对象的所有域都是 final 类型
3. 对象是正确创建的（在对象的创建期间，this 引用没有逸出）

**volatile + 不可变对象**是一种常见的线程安全组合：
```java
// 不可变对象
public class OneValueCache {
    private final BigInteger lastNumber;
    private final BigInteger[] lastFactors;
    public OneValueCache(BigInteger i, BigInteger[] factors) {
        lastNumber = i;
        lastFactors = Arrays.copyOf(factors, factors.length); // 防御性拷贝
    }
    public BigInteger[] getFactors(BigInteger i) {
        if (lastNumber == null || !lastNumber.equals(i))
            return null;
        else
            return Arrays.copyOf(lastFactors, lastFactors.length);
    }
}

// 使用 volatile 安全发布不可变对象
private volatile OneValueCache cache = new OneValueCache(null, null);
```

### 6.4 事实不可变对象（Effectively Immutable）

如果对象从技术上来看是可变的，但其状态在发布后不会再改变，那么这种对象称为"事实不可变对象"。

事实不可变对象不需要满足严格的不可变性条件（如所有域都是 final），但必须被**安全发布**。一旦被安全发布，没有额外同步的情况下任何线程都可以安全地使用它。

### 6.5 线程封闭（Thread Confinement）

**不共享数据，就不需要同步。** 线程封闭是实现线程安全最简单的方式之一。

| 封闭方式 | 说明 |
|----------|------|
| **栈封闭** | 局部变量位于线程栈中，其他线程无法访问。确保不要发布局部变量的引用 |
| **ThreadLocal** | 为每个使用变量的线程都存有一份独立的副本。get/set 都是线程隔离的 |

ThreadLocal 的典型应用：
- SimpleDateFormat（非线程安全）的线程隔离使用
- 数据库连接、Session 管理
- 链路追踪中的 TraceId 传递

---

## 7. 对象的组合（JCIP）

### 7.1 设计线程安全类的三要素

在设计线程安全类的过程中，需要包含以下三个基本要素：
1. **找出构成对象状态的所有变量**
2. **找出约束状态变量的不变条件**
3. **建立对象状态的并发访问管理策略**（同步策略）

### 7.2 实例封闭（Instance Confinement）

**核心思想**：不需要所有组件都是线程安全的，只要能控制所有对组件的访问路径，并通过适当的同步来管理这些访问，就可以构建出线程安全的类。

将非线程安全的对象封装在另一个对象内部，通过封装对象的同步机制保证线程安全。

```java
@ThreadSafe
public class PersonSet {
    @GuardedBy("this")
    private final Set<Person> mySet = new HashSet<>(); // HashSet 本身不是线程安全的

    public synchronized void addPerson(Person p) {
        mySet.add(p);
    }
    public synchronized boolean containsPerson(Person p) {
        return mySet.contains(p);
    }
}
```

### 7.3 Java 监视器模式

**Java 监视器模式（Monitor Pattern）**：把所有可变状态都封装起来，并由对象自己的内置锁来保护。

```java
public class Counter {
    @GuardedBy("this") private long value = 0;
    public synchronized long getValue() { return value; }
    public synchronized long increment() {
        if (value == Long.MAX_VALUE)
            throw new IllegalStateException("counter overflow");
        return ++value;
    }
}
```

**使用私有锁代替内置锁（this）可以提供更强的封装性和安全性**：
```java
public class PrivateLock {
    private final Object myLock = new Object(); // 私有锁
    @GuardedBy("myLock") Widget widget;
    void someMethod() {
        synchronized (myLock) {
            // 访问 widget
        }
    }
}
```

优点：
- 客户端代码无法获取私有锁
- 防止外部代码错误地参与同步策略
- 使用公有锁需要检查整个程序，而使用私有锁只需检查单个类

### 7.4 线程安全性的委托

**委托**：如果类中的各个组件都已经是线程安全的，是否还需要额外的同步？答案取决于类是否在不可变约束上添加其他约束。

- **独立的状态变量**：如果各个状态变量是相互独立的（彼此间没有不变约束），那么可以将线程安全性委托给底层的状态变量
- **当委托失效时**：如果类有复合操作（先检查后执行、读取-修改-写入），则单纯的委托不够，还需要额外的锁机制

```java
// 委托失效的例子：两个 AtomicInteger 之间有不变量约束
public class NumberRange {
    // 不变约束: lower <= upper
    private final AtomicInteger lower = new AtomicInteger(0);
    private final AtomicInteger upper = new AtomicInteger(0);

    public void setLower(int i) {
        if (i > upper.get())  // 非原子操作
            throw new IllegalArgumentException(...);
        lower.set(i);
    }
    // 线程不安全：setLower 和 setUpper 的检查和设置不是原子的
}
```

### 7.5 组合（Composition）

**组合**：在现有线程安全类的基础上添加新功能，同时保持线程安全性。

比"客户端加锁"更推荐的方式是**组合**：将现有类封装为新的类的组件，在新类中使用自己的锁机制。

```java
@ThreadSafe
public class ImprovedList<T> implements List<T> {
    private final List<T> list; // 委托对象
    public ImprovedList(List<T> list) { this.list = list; }

    public synchronized boolean putIfAbsent(T x) {
        boolean contains = list.contains(x);
        if (!contains)
            list.add(x);
        return !contains;
    }
    // ... 其他方法委托给 list
}
```

---

## 8. AQS 框架深入

> 《并发编程的艺术》核心章节。AQS（AbstractQueuedSynchronizer）是 JUC 的基石。

### 8.1 AQS 整体设计

AQS 是用来构建锁或者其他同步组件的基础框架。它使用了一个 **int 成员变量 state 表示同步状态**，通过内置的 **FIFO 队列（CLH 锁队列的变体）** 来完成资源获取线程的排队工作。

核心思想：
- 如果共享资源空闲，将当前请求资源的线程设置为有效工作线程，共享资源设置为锁定状态
- 如果共享资源被占用，通过阻塞等待唤醒机制保证锁分配

**AQS 与 Lock 的关系**：
- **Lock 面向使用者**：定义了使用者与锁交互的接口，隐藏实现细节
- **AQS 面向实现者**：简化了锁的实现方式，屏蔽了同步状态管理、线程排队、等待与唤醒等底层操作

### 8.2 核心数据结构

```java
public abstract class AbstractQueuedSynchronizer {
    private volatile int state;               // 同步状态
    private transient volatile Node head;     // 同步队列头
    private transient volatile Node tail;     // 同步队列尾
    private transient Thread exclusiveOwnerThread; // 独占模式下持有锁的线程
}
```

**Node 节点结构**：
```java
static final class Node {
    volatile int waitStatus;      // 节点等待状态
    volatile Node prev;           // 前驱
    volatile Node next;           // 后继
    volatile Thread thread;       // 当前线程
    Node nextWaiter;              // 等待队列中的后继（共享/独占模式标记）
}
```

**waitStatus 取值**：
| 值 | 常量 | 含义 |
|----|------|------|
| 1 | CANCELLED | 节点被取消（超时或中断），不会再变回其他状态 |
| -1 | SIGNAL | 当前节点的后继节点处于等待状态，当前节点释放锁时需要唤醒后继 |
| -2 | CONDITION | 节点在等待队列中，等待 Condition 唤醒 |
| -3 | PROPAGATE | 共享模式下，释放锁时需要向后传播唤醒 |
| 0 | 初始 | 初始状态 |

### 8.3 独占模式 acquire/release

**acquire 获取锁流程**：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&           // 1. 尝试获取（子类实现）
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // 2. 加入队列并自旋等待
        selfInterrupt();              // 3. 补上中断标记
}
```

流程：
1. `tryAcquire(arg)`：尝试获取同步状态，由子类实现（如 ReentrantLock）
2. `addWaiter(Node.EXCLUSIVE)`：获取失败，将当前线程封装为 Node 加入同步队列尾部（CAS 保证线程安全）
3. `acquireQueued(node, arg)`：自旋或阻塞等待获取锁
   - 判断前驱是否为 head，是则尝试获取
   - 获取失败则调用 `shouldParkAfterFailedAcquire` 判断是否需要阻塞
   - 需要阻塞则调用 `parkAndCheckInterrupt` 使用 `LockSupport.park` 挂起线程
4. `selfInterrupt()`：如果在等待过程中被中断，获取锁后补上中断标记

**release 释放锁流程**：
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {          // 1. 尝试释放（子类实现）
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);     // 2. 唤醒后继节点
        return true;
    }
    return false;
}
```

### 8.4 共享模式 acquireShared/releaseShared

共享模式下，多个线程可以同时获取同步状态（如 Semaphore、CountDownLatch）。

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)  // 返回值 < 0 表示获取失败
        doAcquireShared(arg);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {    // 释放后需要唤醒后继
        doReleaseShared();
        return true;
    }
    return false;
}
```

`tryAcquireShared` 返回值语义：
- 负数：获取失败
- 零：获取成功，但后续共享模式的获取不会成功
- 正数：获取成功，且后续共享模式的获取可能成功

### 8.5 ConditionObject 条件队列

AQS 内部类 ConditionObject 实现了 Condition 接口，每个 Condition 对象都包含一个**条件等待队列**（单向链表）。

**await 流程**：
1. 将当前线程封装为 Node，加入条件等待队列
2. 释放当前线程持有的锁（完全释放，包括重入次数）
3. 线程进入等待状态，直到被 signal 唤醒或中断
4. 唤醒后，将节点从条件队列转移到同步队列，重新竞争锁

**signal 流程**：
1. 将条件队列的首节点转移到同步队列中
2. 修改节点 waitStatus 为 SIGNAL
3. 唤醒该节点对应的线程

**与 Object wait/notify 的区别**：
- Object 的 wait/notify 只能配合一个隐式条件队列；ReentrantLock + Condition 可以创建多个条件队列，实现更精细的等待/唤醒控制
- Condition 的 signal 只唤醒等待该条件的线程，而 notifyAll 会唤醒所有等待该锁的线程

### 8.6 AQS 实现类总结

| 实现类 | 模式 | 核心逻辑 |
|--------|------|----------|
| **ReentrantLock** | 独占 | state 记录重入次数，0 表示未锁定 |
| **ReentrantReadWriteLock** | 独占+共享 | 高16位记录读锁数量，低16位记录写锁重入次数 |
| **CountDownLatch** | 共享 | state 初始化为计数值，countDown 递减，为0时唤醒所有等待线程 |
| **Semaphore** | 共享 | state 初始化为许可数，acquire 减，release 加 |
| **CyclicBarrier** | 独占+共享 | 基于 ReentrantLock + Condition 实现，可循环使用 |

---

## 9. Lock 体系

### 9.1 ReentrantLock

ReentrantLock 是 AQS 独占模式的典型实现，相比 synchronized 提供了更多高级功能：

| 特性 | ReentrantLock | synchronized |
|------|---------------|--------------|
| 可重入 | ✓ | ✓ |
| 可中断 | `lockInterruptibly()` | ✗ |
| 可超时 | `tryLock(long, TimeUnit)` | ✗ |
| 公平性 | 支持公平/非公平 | 非公平 |
| 多条件变量 | 多个 Condition | 一个隐式条件队列 |
| 条件队列 | Condition 精确唤醒 | notifyAll 全部唤醒 |
| 性能 | JDK6 后差距不大 | 有锁升级优化 |
| 异常释放 | 必须在 finally 中 unlock | 自动释放 |

**公平锁 vs 非公平锁**：
- **公平锁**：线程按照请求锁的顺序获取锁。优点是等待线程不会饥饿；缺点是吞吐量大降（线程上下文切换频繁）
- **非公平锁**：线程可以"插队"获取锁。优点是吞吐量高（减少唤醒和阻塞的开销）；缺点是可能导致某些线程长时间等待

> ReentrantLock 默认使用非公平锁。因为当持有锁的线程释放锁时，新到达的线程刚好可以获取锁（还没进入等待队列），而等待队列中的线程需要被唤醒。这种"插队"减少了线程切换开销。

### 9.2 ReentrantReadWriteLock

**读写锁**：读读共享、读写互斥、写写互斥。适合读多写少的场景。

**锁降级**：写锁可以降级为读锁（获取写锁 → 获取读锁 → 释放写锁），但**读锁不能升级为写锁**（会导致死锁）。

锁降级的应用场景：缓存更新时，在持有写锁期间获取读锁，确保数据一致性后释放写锁，然后以读锁继续访问数据。

```java
// 锁降级示例
class CachedData {
    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

    void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // 释放读锁，获取写锁前必须先释放读锁
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                if (!cacheValid) { // 再次检查（双重检查）
                    data = ...;
                    cacheValid = true;
                }
                // 锁降级：在释放写锁前获取读锁
                rwl.readLock().lock();
            } finally {
                rwl.writeLock().unlock(); // 降级为读锁
            }
        }
        try {
            use(data);
        } finally {
            rwl.readLock().unlock();
        }
    }
}
```

### 9.3 StampedLock（JDK8）

StampedLock 提供了三种模式：
1. **写锁**：独占锁，与 ReadWriteLock 的写锁类似
2. **悲观读锁**：与 ReadWriteLock 的读锁类似
3. **乐观读**：不加锁，通过版本戳验证数据是否被修改过

乐观读是 StampedLock 的特色：线程尝试以乐观方式读取，返回一个戳（stamp），读完后验证 stamp 是否有效（即读期间是否有写操作）。如果有效，说明读取期间没有写入；如果无效，升级为悲观读锁重新读取。

```java
class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();

    double distanceFromOrigin() {
        long stamp = sl.tryOptimisticRead();  // 乐观读
        double currentX = x, currentY = y;
        if (!sl.validate(stamp)) {            // 验证是否有写入发生
            stamp = sl.readLock();            // 升级为悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```

> StampedLock 不支持重入，不支持条件变量，使用时需注意。

### 9.4 选择：synchronized vs ReentrantLock

**优先使用 synchronized**：
- 代码更简洁，自动释放锁（即使在异常时）
- 不会忘记在 finally 中 unlock
- JDK6 后 synchronized 性能大幅提升（锁升级优化）

**需要使用 ReentrantLock 的场景**：
- 需要可中断、可超时的锁获取
- 需要公平锁
- 需要多个条件变量（Condition）
- 需要实现复杂的锁获取策略（如轮询、定时）

---

## 10. 并发容器

### 10.1 ConcurrentHashMap

**JDK7：分段锁（Segment）**
- 内部使用 Segment 数组，每个 Segment 是一个独立的 HashEntry 数组
- Segment 继承 ReentrantLock，每个 Segment 独立加锁
- 默认 16 个 Segment，最多支持 16 个线程并发写
- size() 需要对所有 Segment 加锁后统计，多次重试后改为全锁

**JDK8：CAS + synchronized**
- 取消 Segment，直接使用 Node 数组 + 链表/红黑树
- 插入使用 CAS 初始化桶，冲突时使用 synchronized 锁定链表头节点
- 读操作无锁，利用 volatile 保证可见性
- size() 使用 CounterCell 数组分段计数，最后求和

**JDK8 核心优化**：
- 锁的粒度从 Segment 级别降低到桶级别（链表头节点），并发度大幅提升
- 红黑树优化：链表长度 ≥ 8 且数组长度 ≥ 64 时转为红黑树
- 扩容时支持多线程协助迁移（transfer 方法）

### 10.2 CopyOnWriteArrayList / CopyOnWriteArraySet

**写时复制（Copy-On-Write）**：
- 读操作无锁，直接读取当前数组
- 写操作（add/set/remove）时复制一个新数组，在新数组上修改，完成后将引用指向新数组
- 写操作使用 ReentrantLock 保证线程安全

**适用场景**：读多写极少（读占比 90% 以上），且数据量不大。写操作开销大（O(n) 复制数组），且无法保证实时一致性。

### 10.3 阻塞队列

阻塞队列是生产者-消费者模式的理想选择。

| 队列 | 有界性 | 锁 | 特点 |
|------|--------|-----|------|
| **ArrayBlockingQueue** | 有界 | 单锁 | 数组实现，FIFO，支持公平访问策略 |
| **LinkedBlockingQueue** | 可选有界 | 双锁（put/take 分离） | 链表实现，吞吐量通常高于 ArrayBlockingQueue |
| **SynchronousQueue** | 无界（容量为0） | CAS | 不存储元素，每个插入必须等待一个移除，直接传递 |
| **PriorityBlockingQueue** | 无界 | 单锁 | 支持优先级排序 |
| **DelayQueue** | 无界 | 单锁 | 元素只有到期才能取出，用于延迟任务调度 |
| **LinkedTransferQueue** | 无界 | CAS+自旋 | 支持 transfer（直接传递），性能优于 LinkedBlockingQueue |

**LinkedBlockingQueue 的双锁设计**：
- `putLock`：控制生产者入队
- `takeLock`：控制消费者出队
- 两把锁分离，生产者和消费者可以真正并发执行（不像 ArrayBlockingQueue 只有一把锁）

### 10.4 生产者-消费者模式

```java
public class ProducerConsumer {
    private final BlockingQueue<Item> queue;
    public ProducerConsumer(int capacity) {
        this.queue = new ArrayBlockingQueue<>(capacity);
    }
    public void produce(Item item) throws InterruptedException {
        queue.put(item); // 队列满时阻塞
    }
    public Item consume() throws InterruptedException {
        return queue.take(); // 队列空时阻塞
    }
}
```

**串行线程封闭**：通过阻塞队列将对象从一个线程安全地"转移"到另一个线程，对象的所有权转移后，原线程不能再访问该对象。

---

## 11. 线程池深入

### 11.1 ThreadPoolExecutor 七大核心参数

```java
public ThreadPoolExecutor(
    int corePoolSize,           // 核心线程数（常驻）
    int maximumPoolSize,        // 最大线程数
    long keepAliveTime,         // 非核心线程空闲存活时间
    TimeUnit unit,              // 时间单位
    BlockingQueue<Runnable> workQueue,  // 任务等待队列
    ThreadFactory threadFactory,        // 线程工厂
    RejectedExecutionHandler handler    // 拒绝策略
)
```

### 11.2 execute 执行流程

```
提交任务
  │
  ▼
当前运行线程数 < corePoolSize ?
  ├── 是 ──→ 创建新线程执行任务（即使有空闲线程也创建）
  │
  └── 否 ──→ 任务加入 workQueue
                │
                ▼
           队列加入成功？
             ├── 是 ──→ 任务在队列中等待执行
             │            （如果此时线程池中没有工作线程，创建新线程）
             │
             └── 否 ──→ 当前运行线程数 < maximumPoolSize ?
                            ├── 是 ──→ 创建非核心线程执行任务
                            │
                            └── 否 ──→ 执行拒绝策略
```

**源码核心逻辑**：
```java
public void execute(Runnable command) {
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true)) return;  // 创建核心线程
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (!isRunning(recheck) && remove(command))
            reject(command);                      // 线程池已关闭，拒绝
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);               // 创建工作线程消费队列
    }
    else if (!addWorker(command, false))
        reject(command);                          // 队列满且线程数达上限
}
```

### 11.3 Worker 线程复用机制

Worker 是 ThreadPoolExecutor 的内部类，继承 AQS（简化版）并实现 Runnable。

**线程复用核心**：Worker 的 run 方法调用 `runWorker(this)`，其中是一个循环：
```java
final void runWorker(Worker w) {
    Runnable task = w.firstTask;
    while (task != null || (task = getTask()) != null) {
        w.lock(); // Worker 本身作为锁，防止线程池中断时中断正在运行的任务
        try {
            beforeExecute(wt, task);
            try {
                task.run();
            } finally {
                afterExecute(task, thrown);
            }
        } finally {
            task = null;
            w.completedTasks++;
            w.unlock();
        }
    }
    processWorkerExit(w, completedAbruptly);
}
```

`getTask()` 从 workQueue 中获取任务：
- 核心线程：默认调用 `workQueue.take()` 阻塞等待
- 非核心线程：调用 `workQueue.poll(keepAliveTime, TimeUnit)` 超时等待，超时未获取到任务则返回 null，线程退出

### 11.4 拒绝策略

| 策略 | 行为 | 适用场景 |
|------|------|----------|
| **AbortPolicy**（默认） | 抛出 RejectedExecutionException | 快速失败，核心任务 |
| **CallerRunsPolicy** | 由调用线程（提交任务的线程）执行任务 | 流量控制，不丢任务 |
| **DiscardPolicy** | 静默丢弃任务 | 非核心任务，可容忍丢失 |
| **DiscardOldestPolicy** | 丢弃队列中最老的任务，重试提交 | 新任务更重要 |

**自定义拒绝策略**：实现 `RejectedExecutionHandler`，例如记录日志、持久化到 MQ 后续补偿。

### 11.5 线程池参数设计

**核心线程数设置**：
- **CPU 密集型任务**（计算、数据处理）：`corePoolSize = CPU 核心数 + 1`。+1 是为了防止线程偶尔因缺页中断或其他原因暂停时，额外的线程可以顶上去
- **IO 密集型任务**（网络、磁盘）：`corePoolSize = CPU 核心数 × 2` 或更大。因为线程在 IO 等待时不占用 CPU，可以创建更多线程提高并发度

**实践经验公式**：
```
最佳线程数 = CPU核心数 × (1 + 线程等待时间 / 线程计算时间)
```

**队列选择**：
- 有界队列（推荐）：`ArrayBlockingQueue` 或指定容量的 `LinkedBlockingQueue`，防止 OOM
- 无界队列：只有 `LinkedBlockingQueue` 无参构造，危险，队列无限增长会导致内存溢出

**最大线程数**：根据系统资源（内存、连接数）设置上限。太多线程会导致频繁上下文切换，降低性能。

### 11.6 Executors 工厂方法的问题

《并发编程的艺术》和《阿里开发手册》均强烈建议禁止使用 Executors 的便捷方法，直接使用 ThreadPoolExecutor 构造：

| 工厂方法 | 问题 |
|----------|------|
| `newFixedThreadPool` | 使用无界 LinkedBlockingQueue，任务堆积会导致 OOM |
| `newSingleThreadExecutor` | 同上，无界队列 |
| `newCachedThreadPool` | 允许创建 Integer.MAX_VALUE 个线程，线程数失控导致 OOM |
| `newScheduledThreadPool` | 允许创建 Integer.MAX_VALUE 个线程 |

### 11.7 Fork/Join 框架

Fork/Join 框架（JDK7）专为**计算密集型**、可**递归分解**的任务设计，基于**工作窃取（Work-Stealing）**算法。

**核心组件**：
- `ForkJoinPool`：执行 ForkJoinTask 的线程池
- `ForkJoinTask`：抽象任务，子类 `RecursiveTask<V>`（有返回值）、`RecursiveAction`（无返回值）

**工作窃取算法**：
- 每个工作线程维护自己的双端队列（Deque）
- 工作线程从队列**头部**取任务执行（LIFO）
- 当线程队列为空时，从其他线程队列**尾部**窃取任务（FIFO）
- 窃取从尾部进行，减少竞争（工作者操作头部，窃取者操作尾部）

```java
// Fork/Join 计算数组和示例
class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10000;
    private final int[] array;
    private final int start, end;
    // constructor...

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);
        left.fork();          // 异步执行左任务
        long rightResult = right.compute(); // 同步执行右任务
        long leftResult = left.join();      // 等待左任务结果
        return leftResult + rightResult;
    }
}
```

---

## 12. 同步工具类

### 12.1 CountDownLatch（闭锁）

**用途**：等待一个或多个其他线程完成一组操作。

```java
CountDownLatch latch = new CountDownLatch(2);
// 线程1、线程2执行完调用 latch.countDown()
// 主线程调用 latch.await() 阻塞等待，直到计数归零
```

**典型应用**：
- 启动服务前等待多个依赖组件初始化完成
- 并行计算/测试中等待所有子任务完成

> CountDownLatch 的计数器不能重置，一次性使用。

### 12.2 CyclicBarrier（栅栏）

**用途**：让一组线程到达一个屏障时被阻塞，直到最后一个线程到达，屏障才会开门，所有被阻塞的线程才能继续执行。

```java
CyclicBarrier barrier = new CyclicBarrier(3);
// 每个线程执行到 barrier.await() 时阻塞
// 当 3 个线程都到达后，同时放行，可循环使用
```

**与 CountDownLatch 的区别**：
- CountDownLatch：一个（或多个）线程等待其他线程；计数器不能重置
- CyclicBarrier：一组线程互相等待；计数器可循环使用；可传入 Runnable 屏障操作

### 12.3 Semaphore（信号量）

**用途**：控制同时访问特定资源的线程数量，协调各个线程以保证合理的使用公共资源。

```java
Semaphore semaphore = new Semaphore(10); // 10个许可
semaphore.acquire();   // 获取许可，无可用时阻塞
semaphore.release();   // 释放许可
```

**典型应用**：数据库连接池、限流（流量控制）。

### 12.4 Exchanger

**用途**：用于两个线程之间交换数据。当一个线程到达 exchange 调用点时，如果另一个线程也调用了 exchange，则交换数据，否则当前线程阻塞等待。

### 12.5 Phaser（JDK7）

Phaser 是 CountDownLatch 和 CyclicBarrier 的更灵活替代：
- 支持多阶段（phase）控制
- 支持动态注册和注销参与者
- 每个阶段可以定义到达和离开时的动作

---

## 13. 原子操作与 CAS

### 13.1 CAS 原理

CAS（Compare-And-Swap）是 CPU 提供的原子指令（如 x86 的 `cmpxchg`）。包含三个操作数：
- **V**：内存位置（变量的内存地址）
- **A**：预期原值
- **B**：新值

只有当 V 的值等于 A 时，才将 V 的值更新为 B。整个操作是原子的。

**ABA 问题**：
- 线程1读取值 A，准备更新为 C
- 线程2将 A 改为 B，又改回 A
- 线程1执行 CAS 时发现值仍为 A，更新成功，但实际上值已经变化过
- **危害**：在基于引用的无锁数据结构中（如栈），中间变化可能导致指针指向无效节点

**解决方案**：
- `AtomicStampedReference`：维护引用 + 版本号（stamp），每次更新递增 stamp
- `AtomicMarkableReference`：维护引用 + 布尔标记

### 13.2 Atomic 家族

| 分类 | 核心类 | 说明 |
|------|--------|------|
| 基本类型 | `AtomicInteger`、`AtomicLong`、`AtomicBoolean` | 基本类型的原子操作 |
| 数组类型 | `AtomicIntegerArray` 等 | 数组元素的原子操作 |
| 引用类型 | `AtomicReference` | 对象引用的原子替换 |
| 带版本引用 | `AtomicStampedReference` | 解决 ABA 问题 |
| 带标记引用 | `AtomicMarkableReference` | 轻量级标记 |
| 字段更新器 | `AtomicIntegerFieldUpdater` 等 | 反射更新对象的 volatile 字段 |
| 累加器 | `LongAdder`、`DoubleAdder` | 高并发下优于 AtomicLong |

### 13.3 LongAdder（JDK8）

**核心思想**：空间换时间，分段累加。

- 内部维护一个 `base` 变量和 `Cell[]` 数组
- 低并发时直接 CAS 更新 base
- 高并发时线程分散到不同的 Cell 上累加，减少 CAS 竞争
- `sum()` 时汇总 base + 所有 Cell 的值

**LongAdder vs AtomicLong**：
- 低并发：两者性能相近
- 高并发（线程数多、竞争激烈）：LongAdder 吞吐量显著优于 AtomicLong（10 倍以上）
- 需要获取精确当前值时：AtomicLong 的 `get()` 返回精确值；LongAdder 的 `sum()` 是非原子操作，可能不精确

**适用场景**：
- 高并发计数（QPS 统计、请求计数）：用 LongAdder
- 需要精确读取的序列号、ID 生成：用 AtomicLong

### 13.4 Unsafe 与 CAS 底层

`sun.misc.Unsafe` 是 CAS 的底层实现类，提供硬件级别的原子操作。Java9 后被隐藏，推荐使用 `java.lang.invoke.VarHandle`。

```java
// AtomicInteger 的 CAS 实现
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

---

## 14. CompletableFuture 异步编排

CompletableFuture 是 Java8 引入的异步编程工具，实现了 Future 和 CompletionStage 接口。

### 14.1 创建

```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "result");
CompletableFuture<Void> f2 = CompletableFuture.runAsync(() -> { /* 无返回值 */ });
// 可指定自定义线程池，否则使用 ForkJoinPool.commonPool()
```

### 14.2 串行执行

```java
// thenApply: 有入参有返回值（转换结果）
// thenAccept: 有入参无返回值（消费结果）
// thenRun: 无入参无返回值（Runnable）
// 带 Async 后缀的方法在另一个线程执行
f1.thenApply(result -> result.toUpperCase())
  .thenAccept(System.out::println);
```

### 14.3 并行组合

```java
// thenCombine: 等待两个都完成，合并结果
f1.thenCombine(f2, (r1, r2) -> r1 + r2);

// acceptEither: 两个中任意一个完成就消费
f1.acceptEither(f2, System.out::println);

// allOf: 等待所有完成
CompletableFuture.allOf(f1, f2, f3).thenRun(() -> ...);

// anyOf: 任意一个完成
CompletableFuture.anyOf(f1, f2, f3).thenAccept(result -> ...);
```

### 14.4 异常处理

```java
// exceptionally: 捕获异常，返回默认值
future.exceptionally(ex -> "default");

// handle: 无论成功失败都处理，有返回值
future.handle((result, ex) -> ex != null ? "default" : result);

// whenComplete: 类似 handle 但无返回值
future.whenComplete((result, ex) -> { ... });
```

### 14.5 超时处理（JDK9+）

```java
future.orTimeout(1, TimeUnit.SECONDS);       // 超时抛出 TimeoutException
future.completeOnTimeout("default", 1, TimeUnit.SECONDS); // 超时时返回默认值
```

---

## 15. 取消与关闭（JCIP）

### 15.1 协作式取消：中断机制

Java 没有提供安全的强制停止线程的机制，而是提供了**中断（Interruption）**——一种协作机制，让一个线程请求另一个线程停止当前工作。

**中断相关的 API**：
```java
public class Thread {
    public void interrupt() { }           // 设置中断标志
    public boolean isInterrupted() { }    // 查询中断标志（不清除）
    public static boolean interrupted() { } // 查询并清除中断标志
}
```

**中断的响应方式**：
- 抛出 `InterruptedException`：如 `wait()`、`sleep()`、`join()`、`BlockingQueue.take()`/`put()` 等方法，收到中断后清除中断标志并抛出异常
- 检查中断标志：非阻塞任务可以轮询 `isInterrupted()`，发现中断后自行清理并退出

**中断处理最佳实践**：
```java
// 1. 传递 InterruptedException（首选）
public void task() throws InterruptedException { ... }

// 2. 恢复中断状态
public void task() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt(); // 恢复中断标志，让上层处理
        return;
    }
}
```

**绝对不要做的事**：在 catch `InterruptedException` 后什么都不做（吞掉中断）。这会导致调用栈上层无法感知中断。

### 15.2 通过 Future 取消任务

```java
Future<?> future = executor.submit(task);
try {
    future.get(timeout, TimeUnit.SECONDS);
} catch (TimeoutException e) {
    // 超时处理
} catch (ExecutionException e) {
    // 任务执行异常
} finally {
    future.cancel(true); // true = 如果正在运行则中断它
}
```

### 15.3 处理不可中断的阻塞

某些阻塞操作不响应中断，需要特殊处理：

| 阻塞类型 | 处理方式 |
|----------|----------|
| **Socket I/O** | 关闭底层 Socket，`read/write` 抛出 `SocketException` |
| **NIO Channel I/O** | 中断等待在 `InterruptibleChannel` 上的线程，抛出 `ClosedByInterruptException` |
| **Selector** | 调用 `wakeup()` |
| **获取内置锁** | 无法中断，可使用 ReentrantLock 的 `lockInterruptibly()` |

```java
// 重写 interrupt 来关闭 Socket
class ReaderThread extends Thread {
    private final Socket socket;
    public void interrupt() {
        try { socket.close(); } catch (IOException ignored) { }
        finally { super.interrupt(); }
    }
}
```

### 15.4 优雅关闭服务

**毒丸对象（Poison Pill）**：在生产者-消费者队列中放入一个特殊对象，消费者收到后停止工作。

```java
public class IndexingService {
    private static final File POISON = new File("");
    private final BlockingQueue<File> queue;
    private final IndexerThread consumer = new IndexerThread();

    public void start() { consumer.start(); }
    public void stop() { queue.add(POISON); } // 放入毒丸
    public void awaitTermination() throws InterruptedException {
        consumer.join();
    }

    private class IndexerThread extends Thread {
        public void run() {
            try {
                while (true) {
                    File file = queue.take();
                    if (file == POISON) break; // 收到毒丸，退出
                    indexFile(file);
                }
            } catch (InterruptedException consumed) { }
        }
    }
}
```

**Reservation Pattern**：记录未完成的请求数，关闭前等待所有请求处理完毕。

### 15.5 处理非正常线程终止

```java
// 1. 自定义 ThreadFactory 设置未捕获异常处理器
public class MyThreadFactory implements ThreadFactory {
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setUncaughtExceptionHandler((thread, e) -> {
            log.error("Thread {} terminated", thread.getName(), e);
        });
        return t;
    }
}

// 2. 扩展 ThreadPoolExecutor 的 afterExecute 方法
@Override
protected void afterExecute(Runnable r, Throwable t) {
    super.afterExecute(r, t);
    if (t == null && r instanceof Future<?>) {
        try {
            ((Future<?>) r).get();
        } catch (ExecutionException e) {
            t = e.getCause();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    if (t != null) {
        log.error("Task failed", t);
    }
}
```

---

## 16. 活跃性危险（JCIP）

### 16.1 死锁（Deadlock）

死锁是指两个或更多线程互相持有对方需要的锁，导致所有线程都无法继续执行。

**锁顺序死锁**：两个线程以不同的顺序获取相同的锁。
```java
// 线程A: 先锁 accountA，再锁 accountB
// 线程B: 先锁 accountB，再锁 accountA
// 可能死锁！
```

**动态锁顺序死锁**：锁的顺序取决于外部传入的参数。
```java
// 解决方案：通过对象的 hashCode 定义全局顺序
private static final Object tieLock = new Object();
void transferMoney(Account from, Account to, Amount amount) {
    class Helper {
        public void transfer() { ... }
    }
    int fromHash = System.identityHashCode(from);
    int toHash = System.identityHashCode(to);
    if (fromHash < toHash) {
        synchronized(from) { synchronized(to) { new Helper().transfer(); }}
    } else if (fromHash > toHash) {
        synchronized(to) { synchronized(from) { new Helper().transfer(); }}
    } else {
        synchronized(tieLock) { // 哈希冲突时使用决胜锁
            synchronized(from) { synchronized(to) { new Helper().transfer(); }}
        }
    }
}
```

**协作对象间的死锁**：两个线程分别调用两个对象的方法，这两个方法互相调用对方的方法并持有各自的锁。

**开放调用（Open Call）**：在调用某个方法时不需要持有锁。如果在持有锁时调用外部方法（alien method），可能因外部方法获取其他锁而导致死锁。

**资源死锁**：两个线程互相等待对方释放资源（如数据库连接池、线程池）。

### 16.2 避免死锁

1. **固定锁顺序**：所有线程以固定的全局顺序获取锁
2. **开放调用**：尽量缩小同步代码块的范围，不要在持有锁时调用外部方法
3. **支持定时的锁**：使用 `tryLock(long, TimeUnit)`，超时后放弃并回退
4. **使用线程转储分析**：JVM 死锁检测会自动发现死锁并输出在 Thread Dump 中

### 16.3 饥饿与活锁

**饥饿（Starvation）**：某些线程因优先级太低或锁竞争激烈，长时间无法获得锁而无法继续执行。

**活锁（Livelock）**：线程不断改变状态以响应对方，但都无法继续执行（如同两个人在走廊里互相谦让，结果谁也过不去）。
- 解决：引入随机等待，使线程在不同时间重试

**糟糕的响应性**：长时间持有锁导致其他线程等待（如 GUI 事件线程执行耗时操作）。

---

## 17. 性能与可伸缩性（JCIP）

### 17.1 性能 vs 可伸缩性

- **性能**：服务时间、延迟、吞吐量、资源消耗。"多快"
- **可伸缩性**：当增加计算资源（CPU、内存、带宽）时，吞吐量或容量能相应地提升。"多大"

对于并发程序，可伸缩性往往比性能更重要。如果程序不可伸缩，增加资源也无济于事。

### 17.2 Amdahl 定律

```
加速比 = 1 / (F + (1-F)/N)

F = 串行比例（必须串行执行的部分占总执行时间的比例）
N = 处理器数量
```

**关键结论**：
- 即使串行比例只有 10%，无论增加多少 CPU，加速比也永远不超过 10
- 减少串行部分是提高可伸缩性的关键

**隐藏在框架中的串行部分**：
- 线程池的任务队列（所有任务都要经过队列）
- 共享缓存（缓存命中率计算中的共享状态更新）
- 锁竞争（临界区代码是串行的）

### 17.3 线程引入的开销

| 开销类型 | 说明 | 缓解方法 |
|----------|------|----------|
| **上下文切换** | 保存/恢复线程上下文，内核态/用户态切换 | 减少线程数量，增大任务粒度 |
| **内存同步** | 内存屏障导致的缓存失效、刷新 | 减少同步次数，使用原子变量 |
| **阻塞** | 线程挂起和唤醒的代价 | 减少锁持有时间，使用无锁算法 |

### 17.4 减少锁的竞争

**并发程序中，对可伸缩性首要的威胁是独占的资源锁。**

减少锁竞争的三类方法：
1. **减少持有锁的时间**：快进快出，将不影响共享状态的耗时操作移出同步块
2. **减少请求锁的频率**：使用分拆锁（Lock Splitting）或锁分段（Lock Striping）
3. **用协调机制取代独占锁**：ReadWriteLock、原子变量、无锁数据结构

**锁拆分（Lock Splitting）**：如果锁守护多个相互独立的状态变量，将其拆分为多个锁。

**锁分段（Lock Striping）**：将数据分为多个段，每段有独立的锁。ConcurrentHashMap 就是锁分段的典型应用。

**避免热点域**：某些频繁更新的共享变量（如计数器）会成为热点。用独立的计数器数组（如 LongAdder）分摊更新压力。

### 17.5 减少上下文切换

- **无锁并发算法**：CAS 失败时自旋而不是阻塞，但自旋次数要有限制
- **避免不必要的锁**：只在必要时使用同步
- **增大任务粒度**：减少任务提交频率

---

## 18. 并发程序测试（JCIP）

### 18.1 正确性测试

- **基本单元测试**：单线程下的功能测试是必要但不充分的
- **阻塞操作的测试**：验证在条件不满足时线程确实阻塞，条件满足时线程继续执行
- **安全性测试**：构造多线程并发访问的场景，使用计数器、校验和等手段验证正确性
- **产生更多交替操作**：在共享状态的操作之间添加 `Thread.yield()` 或 `sleep`，增加线程切换的概率，更容易暴露竞态条件

### 18.2 性能测试

- **增加计时**：测量吞吐量和响应时间
- **多种算法比较**：对比不同实现（如 synchronizedMap vs ConcurrentHashMap）
- **响应性衡量**：测量服务时间的分布（平均、中位数、99分位）

### 18.3 避免性能测试陷阱

| 陷阱 | 说明 | 避免方法 |
|------|------|----------|
| **垃圾回收** | GC 导致暂停，影响测量 | 预热，排除 GC 时间，多次运行 |
| **动态编译** | JIT 编译导致前期性能差 | 充分预热（-XX:+PrintCompilation） |
| **不真实采样** | 测试时间太短，未覆盖各种情况 | 长时间运行，使用多种输入 |
| **不真实竞争** | 测试中的并发度低于生产环境 | 模拟真实并发数 |
| **无用代码消除** | JIT 将测试代码优化掉 | 确保计算结果被使用（如打印、累加） |

### 18.4 其他测试方法

- **代码审查**：并发代码的审查尤为重要
- **静态分析工具**：FindBugs、SpotBugs 可以检测一些并发问题（如不一致的同步）
- **压力测试**：在极限负载下运行，暴露活跃性问题
- **Thread Dump 分析**：检测死锁和线程阻塞

---

## 附录：并发编程速查口诀

| 场景 | 推荐方案 |
|------|----------|
| 需要可见性，不需要原子性 | `volatile` |
| 单变量原子更新 | `AtomicInteger` / `AtomicLong` |
| 高并发计数/累加 | `LongAdder` |
| 简单的互斥访问 | `synchronized` |
| 需要可中断/可超时/公平锁 | `ReentrantLock` |
| 读多写少 | `ReentrantReadWriteLock` / `StampedLock`（乐观读） |
| 读极多写极少 | `CopyOnWriteArrayList` |
| Map 高并发读写 | `ConcurrentHashMap` |
| 生产者-消费者 | `BlockingQueue` |
| 批量任务等待完成 | `CountDownLatch` / `CompletableFuture.allOf()` |
| 线程互相等待循环使用 | `CyclicBarrier` |
| 限制并发数 | `Semaphore` |
| 计算密集型分治任务 | `ForkJoinPool` + `RecursiveTask` |
| 异步编排流水线 | `CompletableFuture` |
| 线程隔离变量 | `ThreadLocal` |
| 对象安全发布 | 静态初始化 / `volatile` / `final` / 锁保护 |
| 避免死锁 | 全局锁顺序 / 开放调用 / `tryLock` |
| 性能优化 | 缩小锁范围 → 减小锁粒度 → 锁分段 → 无锁/原子变量 |
