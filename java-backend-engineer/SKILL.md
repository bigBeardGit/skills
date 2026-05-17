---
name: java-backend-engineer
description: >
  Java 后端开发专家 skill。当用户涉及 Java 后端开发、SSM/Spring Boot/Spring Cloud 框架使用、Redis/Nginx
  等中间件、高并发多线程编程、JVM 调优、框架源码分析、设计模式应用、代码重构、Java 面试准备、系统架构设计时，
  **必须**使用此 skill。触发场景包括：写 Java 代码、分析 Spring 源码、设计高并发方案、讨论 JVM 原理、
  使用 MyBatis/Redis/消息队列、微服务架构设计、代码审查、性能优化、面试题解答等。即使用户没有明确说
  "Java 后端"，只要涉及上述技术栈，就必须调用此 skill 提供专家级指导。
---

# Java 后端开发工程师

以资深 Java 后端架构师的视角，提供 Java 技术栈全链路的专业指导。精通 Java SE、SSM、Spring Boot、Spring Cloud、中间件、高并发、设计模式和源码分析。

## 核心能力域

---

## 任务类型索引与按需加载

> 本 skill 采用**索引+详细参考**结构。SKILL.md 保留核心速查与任务路由，详细论述性内容存放在 `references/` 目录下。根据用户问题的任务类型，按需读取对应参考文件。

### 按任务类型速查

| 任务类型 | 识别关键词/场景 | 优先读取 | SKILL.md 速查章节 |
|----------|----------------|----------|-------------------|
| **语言基础** | "解释语法"、"这段代码什么意思"、"Java基础"、"面向对象"、"泛型"、"Stream" | `references/java-fundamentals.md` | §1.0 语言基础索引 |
| **集合/JVM/IO** | "HashMap原理"、"ArrayList扩容"、"GC调优"、"类加载"、"NIO"、"Netty" | SKILL.md §1.1~1.4 | — |
| **框架使用** | "Spring事务"、"Bean生命周期"、"MyBatis缓存"、"自动配置"、"Gateway"、"Feign" | SKILL.md §2~5 | — |
| **高并发** | "线程安全"、"线程池"、"AQS"、"volatile"、"synchronized"、"JMM" | `references/concurrency.md` | §6 速查 |
| **设计模式** | "用什么模式"、"模式对比"、"工厂/单例/策略"、"重构到模式" | `references/design-patterns.md` | §7 速查 |
| **架构设计** | "如何设计系统"、"微服务拆分"、"DDD"、"高可用"、"CQRS"、"Clean Architecture" | `references/architecture.md` | §8 架构索引 |
| **代码规范/Review** | "命名规范"、"代码审查"、"重构"、"Clean Code"、"Effective Java" | `references/coding-standards.md` | §9 速查 + 审查清单 |
| **源码分析** | "Spring源码"、"IoC原理"、"AOP实现"、"MyBatis执行流程" | SKILL.md §10 + 源码分析方法论 | — |

### 加载策略

- **默认情况**：先阅读 SKILL.md 的对应章节速查，快速定位知识域
- **深度需求**：当用户问题涉及某领域的系统设计、原理剖析、方案对比时，主动读取对应的 `references/*.md` 获取完整论述
- **多域交叉**：如"高并发架构设计"需同时读取 `references/concurrency.md` + `references/architecture.md`

---

### 1. Java SE 深度掌握

#### 语言基础与面向对象（索引）

> 详细深度内容见 `references/java-fundamentals.md`。以下为极简速查。

**核心一句话**：
- **对象**：一切皆为对象，操纵的是引用；`==` 比地址，`.equals()` 比内容
- **初始化顺序**：父静 → 子静 → 父实例 → 父构造 → 子实例 → 子构造
- **组合优于继承**：Has-A 比 Is-A 更灵活；继承破坏封装
- **多态陷阱**：构造器中不要调用可重写方法；静态方法不参与多态
- **异常**：受检异常（可恢复）vs 非受检异常（编程错误）；绝不吞异常
- **泛型**：编译期类型安全，擦除实现；PECS = Producer Extends, Consumer Super
- **Stream**：惰性求值，不改变源数据；并行流慎用；Optional 只作返回值
- **版本特性**：JDK 8(Lambda+Stream) → 17(密封类) → 21(虚拟线程)；新项目优先 LTS

---

#### 集合框架
- **ArrayList**: 底层 Object 数组，扩容机制 1.5 倍，初始容量 10，线程不安全
- **LinkedList**: 双向链表，适合频繁插入删除，实现了 Deque 接口
- **HashMap**: 数组+链表+红黑树，JDK8 优化，扩容 2 倍，负载因子 0.75
  - 扰动函数: `(h = key.hashCode()) ^ (h >>> 16)`，减少哈希冲突
  - 树化阈值 8，反树化阈值 6，最小树化容量 64
- **ConcurrentHashMap**: JDK7 分段锁(Segment)，JDK8 CAS+synchronized，并发度更高
- **CopyOnWriteArrayList**: 写时复制，读多写少场景，add 时复制新数组

#### IO/NIO
- **BIO**: 同步阻塞，一个连接一个线程，适合连接数少的场景
- **NIO**: 同步非阻塞，Selector+Channel+Buffer，适合高并发连接
- **AIO**: 异步非阻塞，基于回调，Linux 下 epoll 实现
- **Netty 基础**: 基于 NIO 的事件驱动网络框架，理解 EventLoop、ChannelPipeline、ByteBuf

#### JVM
- **内存模型**: 堆(年轻代 Eden/S0/S1、老年代)、元空间、虚拟机栈、本地方法栈、程序计数器
- **垃圾回收**:
  - CMS: 并发标记清除，低停顿，内存碎片，JDK14 移除
  - G1:  region 化内存，可预测停顿，适合大堆
  - ZGC/Shenandoah: 亚毫秒级停顿，染色指针/读屏障
- **类加载机制**: 双亲委派模型，Bootstrap → Extension → Application → Custom
  - 打破双亲委派: SPI 机制、OSGi、Tomcat 隔离
- **调优参数**: `-Xms/-Xmx` 建议相同避免动态调整，`-XX:+UseG1GC`，`-XX:MaxGCPauseMillis`

#### 反射与代理
- **动态代理**: JDK 动态代理(基于接口，`InvocationHandler`)、CGLIB(基于继承，`MethodInterceptor`)
- Spring AOP 默认 JDK 代理，无接口时 fallback 到 CGLIB

### 2. SSM 框架精通

#### Spring Core
- **IoC 容器**: BeanFactory vs ApplicationContext，后者含国际化、事件传播、资源加载
- **Bean 生命周期**: 实例化 → 属性赋值 → Aware 接口 → BeanPostProcessor.before → 初始化 → BeanPostProcessor.after → 销毁
- **循环依赖解决**: 三级缓存（singletonObjects、earlySingletonObjects、singletonFactories），构造器注入无法解决
- **事务传播**: REQUIRED(默认)、REQUIRES_NEW、NESTED、SUPPORTS、NOT_SUPPORTED、MANDATORY、NEVER
- **事务失效场景**: 非 public 方法、同类自调用、异常被吞、rollbackFor 配置错误、数据库引擎不支持

#### Spring MVC
- **请求处理流程**: DispatcherServlet → HandlerMapping → HandlerAdapter → Controller → ViewResolver → View
- **注解**: `@Controller`、`@RestController`(= @Controller + @ResponseBody)、`@RequestMapping` 派生注解
- **参数绑定**: `@RequestParam`、`@PathVariable`、`@RequestBody`(JSON 反序列化 Jackson)、`@ModelAttribute`
- **异常处理**: `@ControllerAdvice` + `@ExceptionHandler`，全局统一异常返回
- **拦截器**: `HandlerInterceptor` 接口，preHandle → postHandle → afterCompletion

#### MyBatis
- **执行流程**: SqlSessionFactory → SqlSession → Executor → StatementHandler → ParameterHandler → ResultSetHandler
- **缓存**: 一级缓存(SqlSession 级别，默认开启)、二级缓存(Mapper Namespace 级别，需配置)
- **延迟加载**: 基于 CGLIB 代理，关联对象按需加载
- **插件机制**: 拦截 `Executor`、`StatementHandler`、`ParameterHandler`、`ResultSetHandler`
- **#{} vs ${}`: `#{}` 预编译防 SQL 注入，`${}` 直接拼接，用于动态表名/列名
- **动态 SQL**: `<if>`、`<choose>`、`<where>`、`<foreach>`、`<bind>`

### 3. Spring Boot 精通

#### 自动配置原理
- `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
- `META-INF/spring.factories`(Spring Boot 2.7-) 或 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`(2.7+)
- 条件注解: `@ConditionalOnClass`、`@ConditionalOnProperty`、`@ConditionalOnMissingBean`
- 自定义 starter: 创建 `xxx-spring-boot-autoconfigure` + `xxx-spring-boot-starter`

#### 内嵌容器
- 默认 Tomcat，可切换 Jetty/Undertow
- 通过 `WebServerFactoryCustomizer` 自定义线程池、连接数等参数

#### 监控与运维
- **Actuator**: `/actuator/health`、`/actuator/metrics`、`/actuator/env`
- **Micrometer**: 指标收集，对接 Prometheus/Grafana

### 4. Spring Cloud 体系

#### 核心组件
- **Nacos**: 注册中心 + 配置中心，支持 AP/CP 切换，对比 Eureka(纯 AP)、Consul(CP/AP)
- **OpenFeign**: 声明式 HTTP 客户端，集成 Ribbon(负载均衡)、Sentinel(熔断)
  - 底层基于 JDK 动态代理 + 编码器/解码器
- **Gateway**: 基于 Spring WebFlux + Reactor，路由、断言、过滤器
  - 对比 Zuul1(阻塞) / Zuul2(异步 Netty)
- **Sentinel**: 流量控制、熔断降级、系统自适应保护，对比 Hystrix(已停更)
- **Seata**: 分布式事务，AT(默认，基于 undo_log)、TCC、Saga、XA 模式

#### 微服务设计原则
- 服务拆分: 按业务边界(DDD 限界上下文)、避免过度拆分
- 数据库独立: 每个服务独享数据库，禁止直接访问其他服务数据库
- API 版本控制: URL 路径 `/v1/users` 或 Header `Accept: application/vnd.api.v1+json`
- 分布式一致性: CAP 理论，CP 或 AP 选型，最终一致性方案

### 5. 中间件

#### Redis
- **数据类型与应用场景**:
  - String: 缓存、计数器、分布式锁(`SET key value NX EX 10`)
  - Hash: 对象存储、购物车
  - List: 消息队列(生产者消费者)、时间线(lpush + lrange)
  - Set: 去重、交集并集(共同好友)
  - ZSet: 排行榜、延迟队列(时间戳做 score)
  - Bitmap/HyperLogLog/Geo: 布隆过滤器、UV 统计、地理位置
- **持久化**: RDB(快照，fork 子进程，恢复块但丢数据)、AOF(日志，fsync 策略，恢复慢但完整)、混合持久化(4.0+)
- **高可用**: 主从复制(全量+增量)、哨兵(自动故障转移)、Cluster(16384 槽位，去中心化)
- **缓存问题**: 穿透(布隆过滤器/缓存空值)、击穿(互斥锁/逻辑过期)、雪崩(随机 TTL/多级缓存/熔断降级)
- **线程模型**: 单线程事件循环 + IO 多路复用(epoll/kqueue)，6.0 引入多 IO 线程处理网络读写

#### Nginx
- **核心功能**: 反向代理、负载均衡(轮询/权重/ip_hash/least_conn)、静态资源、HTTPS/TLS
- **架构**: Master-Worker 多进程模型，Worker 基于 epoll 事件驱动
- **配置优化**: `worker_processes auto`、`worker_connections 1024`、`keepalive_timeout`、`gzip` 压缩
- **Lua 扩展**: OpenResty，通过 `ngx_lua` 模块实现动态网关、限流、鉴权

#### 消息队列
- **RocketMQ**: NameServer 注册中心，Broker 主从，Topic/Queue 模型
  - 事务消息: Half 消息 → 本地事务 → Commit/Rollback → 回查机制
  - 顺序消息: 同一 Queue 内有序，Hash 取模保证同 key 同队列
- **Kafka**: 高吞吐日志型 MQ，Topic-Partition 模型，ISR 机制，零拷贝(sendfile)
- **RabbitMQ**: 交换机类型(direct/topic/fanout/headers)，TTL+死信队列实现延迟消息

### 6. 高并发与多线程（综合两本经典著作）

> 详细深度内容见 `references/concurrency.md`

> 综合《Java并发编程的艺术》（方腾飞、魏鹏、程晓明）与《Java Concurrency in Practice》（Brian Goetz 等，JCIP）两本经典著作的核心思想。
>
> 《并发编程的艺术》侧重底层实现原理（JMM、volatile、synchronized、AQS、线程池源码、并发容器）；《并发编程实战》侧重设计原则与工程实践（线程安全、安全发布、取消与关闭、活跃性、性能与可伸缩性）。两者结合，既有"底层如何工作"的深刻理解，又有"上层如何设计"的工程智慧。
>
> **详细参考**：需要深入了解某项机制的完整源码分析、设计原则、测试方法时，阅读 `references/concurrency.md`。

#### 6.1 并发三大问题与 JMM

| 问题 | 含义 | 解决手段 |
|------|------|----------|
| **原子性** | 操作不可中断，要么全做要么不做 | `synchronized`、Lock、原子类（CAS） |
| **可见性** | 线程修改共享变量后其他线程立即可见 | `volatile`、`synchronized`、Lock、`final` |
| **有序性** | 程序按代码顺序执行 | `volatile`、`synchronized`、happens-before |

**JMM 核心**：主内存 + 线程工作内存。happens-before 是判断数据竞争和线程安全的主要依据（8条规则：程序次序、监视器锁、volatile变量、传递性、start、join、中断、对象终结）。

**volatile 内存语义**：
- 可见性：写 volatile 立即刷新到主内存，读 volatile 使工作内存失效
- 有序性：禁止指令重排序（通过内存屏障实现：写前 StoreStore、写后 StoreLoad、读后 LoadLoad/LoadStore）
- 只保证单个读/写的原子性，`volatile++` 不原子

**synchronized 锁升级**（JDK6 优化）：
```
无锁 → 偏向锁（同一线程再次进入无需 CAS）→ 轻量级锁（CAS 自旋）→ 重量级锁（操作系统 Mutex）
```

**final 初始化安全性**：构造函数中 final 域的写不会重排序到构造器之外。前提是 this 引用不能在构造期间逸出（不要在构造器中启动线程、注册监听器、发布内部类）。

#### 6.2 线程安全性（JCIP 核心哲学）

> **编写线程安全代码的核心在于：对共享可变状态的访问进行管理。**

- **无状态的对象总是线程安全的**（没有字段，或只有 final 字段）
- **竞态条件**：`check-then-act`（如懒汉单例）和 `read-modify-write`（如 `i++`）
- **用锁保护状态**：每个共享可变变量都应只由一个锁保护。使用 `@GuardedBy("lock")` 标注
- **活跃性与性能**：同步块只包含必要操作，将耗时的 I/O/计算移出同步块

**对象的共享与安全发布**：
- **逸出**：对象在构造完成前被发布（this 引用逸出、返回私有可变引用）
- **安全发布的四种模式**：静态初始化、`volatile` 引用、`final` 域、锁保护
- **不可变对象**一定是线程安全的；**事实不可变对象**（发布后不再修改）被安全发布后无需额外同步
- **线程封闭**：栈封闭（局部变量）、`ThreadLocal`（线程隔离副本）

**对象的组合**：
- **实例封闭**：将非线程安全对象封装在另一个对象内，通过封装对象的同步保证安全
- **Java 监视器模式**：所有可变状态封装，由对象自己的内置锁保护
- **私有锁优于内置锁**：`private final Object lock = new Object();`，防止客户端错误参与同步
- **线程安全性委托**：如果组件已是线程安全的且状态变量独立，可将安全性委托给组件（但复合操作仍需额外同步）

#### 6.3 AQS 与 Lock 体系

**AQS 核心**（`AbstractQueuedSynchronizer`）：
- `state`（int 同步状态）+ FIFO 同步队列（CLH 变体）+ Condition 条件队列
- 独占模式：`acquire` → `tryAcquire`（子类实现）→ 入队 → 自旋/阻塞 → `release` → 唤醒后继
- 共享模式：`acquireShared` → `tryAcquireShared`（返回值 <0 失败，>0 还有剩余）→ `releaseShared` 传播唤醒
- `Node.waitStatus`：CANCELLED(1)/SIGNAL(-1)/CONDITION(-2)/PROPAGATE(-3)
- **ConditionObject**：`await` 释放锁并加入条件队列；`signal` 将节点从条件队列转移到同步队列并唤醒

**Lock 对比**：

| 特性 | `synchronized` | `ReentrantLock` | `ReentrantReadWriteLock` | `StampedLock` |
|------|---------------|-----------------|-------------------------|---------------|
| 可重入 | ✓ | ✓ | ✓ | ✗ |
| 可中断/超时 | ✗ | ✓ | ✓ | ✓ |
| 公平性 | 非公平 | 可选 | 可选 | 非公平 |
| 条件变量 | 1个隐式 | 多个 Condition | 支持 | 不支持 |
| 特色 | 自动释放、锁升级 | 灵活控制 | 读读共享 | 乐观读（无锁读） |

> **选择建议**：优先用 `synchronized`（简洁、自动释放、JDK6后性能差距不大）；需要可中断/多条件/公平锁时用 `ReentrantLock`。

**读写锁的锁降级**：获取写锁 → 获取读锁 → 释放写锁 → 以读锁继续。读锁**不能**升级为写锁（会死锁）。

#### 6.4 线程池深入

**ThreadPoolExecutor 七大参数**：核心线程数、最大线程数、存活时间、时间单位、任务队列、线程工厂、拒绝策略。

**execute 流程**：
```
当前线程 < corePoolSize → 创建核心线程执行
  ↓ 否
加入 workQueue → 队列满？
  ↓ 是
当前线程 < maximumPoolSize → 创建非核心线程执行
  ↓ 否
执行拒绝策略
```

**拒绝策略**：AbortPolicy（抛异常，默认）、CallerRunsPolicy（调用者执行，背压）、DiscardPolicy（静默丢弃）、DiscardOldestPolicy（丢弃最老）。生产环境推荐自定义（记录日志+持久化补偿）。

**参数设计**：
- CPU 密集型：`corePoolSize = CPU核数 + 1`
- IO 密集型：`corePoolSize = CPU核数 × 2` 或更大（线程等待时不占 CPU）
- 公式：`最佳线程数 = CPU核心数 × (1 + 等待时间/计算时间)`
- **禁止使用 Executors 便捷方法**（Fixed/Cached/Single 都有 OOM 风险）

**Worker 复用机制**：`runWorker` 内循环从队列取任务执行。核心线程调用 `take()` 永久阻塞；非核心线程调用 `poll(keepAliveTime)` 超时退出。

**Fork/Join 框架**：
- 基于工作窃取（Work-Stealing），每个线程维护双端队列，空闲时从其他线程尾部窃取任务
- `RecursiveTask<V>`（有返回值）、`RecursiveAction`（无返回值）
- 适合计算密集型、可递归分解的任务（数组求和、排序）

#### 6.5 并发容器

| 容器 | 核心机制 | 适用场景 |
|------|----------|----------|
| **ConcurrentHashMap** | JDK8: CAS + synchronized(桶头) + 红黑树 | 高并发 Map，读基本无锁 |
| **CopyOnWriteArrayList** | 写时复制数组（加锁写，无锁读） | 读多写极少（读占比90%+） |
| **ArrayBlockingQueue** | 有界数组 + 单锁 | 有界生产者-消费者 |
| **LinkedBlockingQueue** | 链表 + 双锁（putLock/takeLock） | 吞吐量更高的生产者-消费者 |
| **SynchronousQueue** | 不存储元素，直接传递 | 任务交接、Executors.newCachedThreadPool |
| **DelayQueue** | 优先级堆 + 到期才能取出 | 延迟任务调度 |

**生产者-消费者模式**：使用 `BlockingQueue.put()`（满时阻塞）和 `take()`（空时阻塞），天然实现流量削峰和背压。

#### 6.6 原子操作与 CAS

**CAS**：Compare-And-Swap，CPU 原子指令（`cmpxchg`）。三个操作数 V（内存地址）、A（预期值）、B（新值），V==A 时更新为 B。

**ABA 问题**：值 A→B→A，CAS 误判未变化。解决：`AtomicStampedReference`（值+版本号）。

**LongAdder（JDK8）**：高并发计数利器。内部 `base` + `Cell[]` 分段累加，线程分散到不同 Cell 减少 CAS 竞争。吞吐量比 `AtomicLong` 高 10 倍以上。`sum()` 非原子，适合统计而非精确值。

#### 6.7 同步工具类速查

| 工具类 | 作用 | 特点 |
|--------|------|------|
| **CountDownLatch** | 等待 N 个线程完成 | 一次性，计数不能重置 |
| **CyclicBarrier** | N 个线程互相等待到达屏障 | 可循环，可执行屏障动作 |
| **Semaphore** | 控制同时访问的线程数 | acquire/release 许可 |
| **Exchanger** | 两个线程交换数据 | 双向数据传递 |
| **Phaser** | 分阶段控制多线程 | 比 CountDownLatch/CyclicBarrier 更灵活 |

#### 6.8 CompletableFuture 异步编排

```java
// 创建
supplyAsync(() -> result) / runAsync(() -> {})
// 串行
thenApply / thenAccept / thenRun
// 组合
thenCombine(another, (a,b) -> ...) / allOf / anyOf
// 异常
exceptionally(ex -> default) / handle((res, ex) -> ...)
// 超时(JDK9)
orTimeout / completeOnTimeout
```

#### 6.9 取消与关闭（JCIP）

- **协作式取消**：通过中断（interrupt）请求线程停止，不是强制停止
- **中断处理**：阻塞方法抛出 `InterruptedException`；非阻塞任务轮询 `isInterrupted()`。catch 后必须恢复中断状态（`Thread.currentThread().interrupt()`）或重新抛出，**绝对不要吞掉**
- **Future.cancel(true)**：中断正在执行的任务
- **不可中断阻塞的处理**：Socket I/O 关闭底层 socket；NIO 关闭 Channel；Selector 调用 wakeup()
- **优雅关闭**：毒丸对象（Poison Pill）通知消费者结束；Reservation Pattern 等待未完成请求

#### 6.10 活跃性危险（JCIP）

**死锁类型**：
- 锁顺序死锁：线程以不同顺序获取相同锁
- 动态锁顺序死锁：锁顺序取决于运行时参数（解决：按对象 hashCode 定义全局顺序）
- 协作对象间死锁：互相调用对方的方法并持有各自锁
- 开放调用死锁：持有锁时调用外部方法，外部方法又获取其他锁
- 资源死锁：互相等待对方释放资源（如数据库连接）

**避免死锁**：固定全局锁顺序、开放调用（不持锁调外部方法）、使用 `tryLock` 超时放弃。

**饥饿**：线程长期无法获取锁。解决：避免固定优先级、使用公平锁。

**活锁**：线程不断响应对方改变状态，但都无法前进。解决：引入随机等待。

#### 6.11 性能与可伸缩性（JCIP）

**Amdahl 定律**：加速比 = 1 / (F + (1-F)/N)，F 为串行比例。即使 F=10%，加速比上限也只有 10。

**减少锁竞争的三级手段**：
1. **缩小锁范围**：快进快出，只保护必要操作
2. **减小锁粒度**：锁拆分（多个独立状态用多个锁）、锁分段（ConcurrentHashMap）
3. **替代独占锁**：ReadWriteLock、原子变量、无锁数据结构

**线程开销**：上下文切换、内存同步（缓存失效）、阻塞（挂起/唤醒）。

#### 6.12 并发编程规范

- **同步访问共享的可变数据**（《Effective Java》第78条）
- **避免过度同步**：不要在同步区域内调用外来方法（《Effective Java》第79条）
- **线程池必须通过 `ThreadPoolExecutor` 创建**（《阿里手册》），禁止 `Executors` 便捷方法
- **线程名称要有意义**，方便出错时回溯
- **优先使用 `java.util.concurrent`**：`ConcurrentHashMap` 优于 `Collections.synchronizedMap`，`BlockingQueue` 用于生产者-消费者
- **高并发时考量锁的性能损耗**：能用无锁数据结构就不用锁；能锁区块就不锁整个方法体；能用对象锁就不用类锁
- **优先选择不可变对象**：除非有很好的理由让类可变，否则应设为不可变（《Effective Java》第17条）。不可变对象天然线程安全

### 7. 设计模式（综合七本经典著作）

> 详细深度内容见 `references/design-patterns.md`

> 综合 GOF《设计模式》、《Head First 设计模式》、《图解设计模式》、《设计模式之禅》、《大话设计模式》、《研磨设计模式》、《深入设计模式》七本书的核心思想。
>
> **详细参考**: 需要深入了解某种模式的完整分析（意图、场景、UML、代码、对比、框架应用）时，阅读 `references/design-patterns.md`。

#### 7.1 设计模式哲学

**本质定义**：设计模式是在特定环境下，针对一类设计问题的**可复用解决方案**（GOF）。是前辈们对代码开发经验的总结，是提高代码**可复用性、可读性、可维护性、稳健性、安全性**的解决方案（《大话设计模式》）。

**两大核心原则**（GOF 全书反复出现的主题，《Head First》九大 OO 原则之首）：
1. **针对接口编程，而不是针对实现编程** —— 不依赖具体类，只依赖抽象契约，这是松耦合的根基
2. **多用组合，少用继承** —— 继承是白箱复用、编译时静态绑定；组合是黑箱复用、运行时动态绑定，更灵活

**原则与模式的关系**（《研磨设计模式》）：**设计原则是思想层面的指导，设计模式是实现层面的手段。** 模式是原则的具体体现，原则是模式的灵魂。先理解 WHY（原则），再学习 HOW（模式）。

**学习姿势**：不在于死记硬背，而在于更深刻地理解面向对象，打开新的思维方式（《研磨》）。最好的学习方式是在**重构**中体会（《设计模式之禅》）。理解 WHY 比记住 HOW 重要一万倍（《Head First》）。

#### 7.2 七大面向对象设计原则

| 原则 | 核心定义 | 关键实践 | 地位 |
|------|----------|----------|------|
| **O**pen/Closed 开闭原则 | 对扩展开放，对修改关闭 | 新增代码而非修改旧代码；多态替换条件分支 | **总纲/终极目标** |
| **S**ingle Responsibility 单一职责 | 一个类只有一个引起变化的原因 | 分离业务逻辑与界面逻辑；类名清晰表达单一职责 | 基础 |
| **L**iskov Substitution 里氏替换 | 基类可用的地方，子类必须可透明替换 | 子类扩展但不改变父类原有语义；Square≠Rectangle | 继承约束 |
| **I**nterface Segregation 接口隔离 | 客户端不应依赖不需要的接口 | 拆分胖接口为小而专注的接口；按需依赖 | 接口粒度 |
| **D**ependency Inversion 依赖倒置 | 高层与低层都依赖抽象；抽象不依赖细节 | 面向接口编程；构造函数/Setter 注入（Spring IoC） | 依赖方向 |
| **Lo**D 迪米特法则/最少知识 | 一个对象对其他对象了解越少越好 | 不与陌生人说话；避免链式调用 `a.getB().getC()` | 知识隐藏 |
| **CARP** 合成复用原则 | 优先使用组合/聚合，而非继承 | Has-A 优于 Is-A；黑箱复用优于白箱复用 | 复用方式 |

**原则关系**：OCP 是总纲，SRP 是基础，LSP + DIP 保证扩展正确性，ISP + LoD + CARP 是实现手段。

#### 7.3 23种设计模式核心速查

每种模式按 **意图 → 场景 → 实际应用** 描述。详细分析（UML、完整代码、相关模式对比、框架源码映射）见 `references/design-patterns.md`。

##### 创建型（5种）— 对象创建的抽象

| 模式 | 核心意图 | 适用场景 | Java 实际应用 |
|------|----------|----------|--------------|
| **单例** | 确保类只有一个实例，提供全局访问点 | 配置中心、连接池、线程池 | `Runtime`、Spring 默认 Bean、枚举本质 |
| **工厂方法** | 定义创建接口，让子类决定实例化哪个类 | 创建对象需大量重复代码；客户端不需知具体类 | `BeanFactory`、`Collection.iterator()` |
| **抽象工厂** | 创建相关对象家族，不需指定具体类 | 多系列产品（UI 皮肤、数据库访问族） | `ConnectionFactory`、Spring `FactoryBean` |
| **建造者** | 将复杂对象构建与表示分离 | 对象构造参数过多（尤其多可选参数）；分步构建 | `StringBuilder`、OkHttp、Lombok `@Builder` |
| **原型** | 通过复制现有对象创建新对象 | 创建成本高；需保留对象状态 | `Object.clone()`、Spring `prototype` Bean |

**工厂家族对比**：简单工厂(1个if工厂) → 工厂方法(每产品1工厂) → 抽象工厂(每族1工厂)。建造者与工厂的区别：工厂关注"创建什么"，建造者关注"如何一步步构建"。

##### 结构型（7种）— 类与对象的组合

| 模式 | 核心意图 | 适用场景 | Java 实际应用 |
|------|----------|----------|--------------|
| **代理** | 为对象提供代理以控制访问 | 远程访问、延迟加载、权限控制、日志 | Spring AOP、MyBatis Mapper 代理 |
| **装饰器** | 动态给对象添加额外职责 | 运行时透明地增强功能；比继承更灵活 | Java IO 流体系、Spring Cache 装饰 |
| **适配器** | 将一个接口转换为客户希望的另一个接口 | 旧系统兼容；接口不匹配 | `InputStreamReader`、`HandlerAdapter` |
| **桥接** | 抽象与实现分离，各自独立变化 | 两个独立变化维度；避免类爆炸 | JDBC `DriverManager`、AWT Peer |
| **组合** | 树形结构表示"部分-整体"层次 | 统一处理单个对象和组合对象 | UI `Container`/`Component`、文件系统 |
| **外观** | 为子系统提供统一的高层接口 | 简化复杂子系统；层间解耦 | SLF4J、`JdbcTemplate` |
| **享元** | 共享技术支持大量细粒度对象 | 大量相似对象，内存开销大；状态可外部化 | `Integer`缓存(-128~127)、String常量池、连接池 |

**"包装三兄弟"对比**：
- 代理 = 控制访问（客户端通常不知道）
- 装饰器 = 增强功能（客户端通常知道并参与）
- 适配器 = 接口转换（客户端只知道目标接口）

##### 行为型（11种）— 对象间的职责分配与通信

| 模式 | 核心意图 | 适用场景 | Java 实际应用 |
|------|----------|----------|--------------|
| **策略** | 定义算法族，封装并使它们可相互替换 | 多行为差异；运行时切换算法 | `Comparator`、Spring `Resource`、支付选择 |
| **观察者** | 一对多依赖，状态变化自动通知更新 | 事件订阅；发布-订阅；解耦发布者与订阅者 | Spring `ApplicationListener`、消息队列 |
| **模板方法** | 定义算法骨架，子类实现可变步骤 | 算法不变部分统一，可变部分延迟到子类 | `AbstractList`、`JdbcTemplate`、`HttpServlet` |
| **责任链** | 多个对象有机会处理请求，沿链传递 | 审批流程；Filter 链；异常处理 | Servlet Filter、Netty Pipeline |
| **命令** | 将请求封装为对象，支持参数化/撤销/队列 | 请求发送者与接收者解耦；Undo/Redo；事务 | `Runnable`/`Callable`、线程池任务 |
| **迭代器** | 顺序访问聚合元素，不暴露内部表示 | 遍历集合；支持多种遍历方式 | Java `Iterator`、`ListIterator` |
| **状态** | 内部状态改变时改变对象行为 | 对象行为依赖状态；大量状态条件语句 | 订单状态机、`Thread.State`、电梯控制 |
| **中介者** | 用中介封装对象间交互，使其松散耦合 | 多对象复杂通信；避免类间直接引用 | MQ、MVC Controller、聊天室 |
| **备忘录** | 捕获并外部保存对象状态，支持恢复 | 撤销/重做；游戏存档；事务 savepoint | 编辑器撤销、数据库 savepoint |
| **解释器** | 定义语言文法表示，解释语言句子 | 简单领域特定语言；规则引擎 | 正则表达式 `Pattern`、EL 表达式 |
| **访问者** | 作用于对象结构中各元素，不改变元素类 | 对异构结构执行多种不相关操作 | ASM 字节码、编译器 AST 遍历 |

**行为型易混淆模式对比**：
- **策略 vs 状态**：策略由客户端主动选择，状态由内部规则自动驱动
- **策略 vs 模板方法**：策略是整个算法替换（组合），模板方法是骨架固定子类重写（继承）
- **观察者 vs 中介者**：观察者是一对多广播，中介者是多对多集中转发
- **装饰器 vs 责任链**：装饰器每层都处理（层层叠加），责任链找到处理者即停止（或继续传递）

#### 7.4 设计模式应用原则

**何时使用**（《Head First》+《研磨》）：
1. 设计中出现"坏味道"（重复代码、庞大类、条件泛滥）时
2. 需求变化频繁，需要系统保持灵活时
3. 先让代码工作，再在重构中引入模式（不要过度设计）

**常见反模式**：
- 为了用模式而用模式（模式滥用）
- 一个系统堆砌大量模式（模式堆砌）
- 简单方案能解决问题时硬套模式

**重构到模式**（《设计模式之禅》）：
1. 写出能工作的代码 → 2. 识别坏味道 → 3. 应用原则分析 → 4. 引入模式重构 → 5. 保持测试通过

#### 7.5 框架源码中的设计模式

**Spring**（近20种）：单例(Bean)、工厂方法(`BeanFactory`)、代理(AOP)、装饰器(`TransactionAwareCacheDecorator`)、适配器(`HandlerAdapter`)、模板方法(`JdbcTemplate`)、观察者(`ApplicationListener`)、策略(`Resource`)、责任链(`HandlerInterceptor`)、访问者(`BeanDefinitionVisitor`)

**JDK**：代理(`Proxy`)、装饰器(IO流)、适配器(`InputStreamReader`)、享元(`Integer`缓存、String常量池)、迭代器(`Iterator`)、策略(`Comparator`)、模板方法(`AbstractList`)、访问者(`FileVisitor`)

**MyBatis**：工厂方法(`SqlSessionFactory`)、建造者(`XMLConfigBuilder`)、代理(Mapper)、装饰器(`CachingExecutor`)、模板方法(`BaseExecutor`)、责任链(Interceptor)、策略(`TypeHandler`)



### 8. 软件架构设计（索引）

> 详细深度内容见 `references/architecture.md`。以下为关键词索引与极简速查。

**架构哲学**：
- 架构 = 重大设计决策（以变更成本衡量），关注非功能性需求
- 架构师是**决策者+沟通者**，核心产出是决策（ADR记录），不是图纸
- 康威定律：设计系统的组织，其产生的设计等同于组织间的沟通结构

**质量属性速查**（ATAM评估）：

| 属性 | 战术 |
|------|------|
| 性能 | 缓存、异步、批处理、负载均衡、数据分区 |
| 可用性 | 冗余、故障转移、降级、熔断、限流 |
| 可修改性 | 封装、抽象、插件架构、配置文件化 |
| 可测试性 | 接口隔离、依赖注入、Mock、契约测试 |

**架构风格选择**：
- **分层**：上层调用下层，层间接口解耦；DDD四层 = 接口/应用/领域/基础设施
- **六边形/端口适配器**：领域为核心，技术细节（DB/Web）是插件
- **微服务**：按业务边界（限界上下文）拆分，数据库独立，同步查询+异步命令
- **CQRS**：读写分离，读多写少场景适用
- **事件溯源**：以事件序列为真相，与CQRS天然配合

**4+1 视图 vs C4 模型**：
- 4+1：逻辑/开发/过程/物理/场景 —— 经典全面
- C4：系统上下文→容器→组件→代码 —— 分层沟通，不同受众看不同层

**核心领域速查**：
- **PEAA**：简单CRUD→事务脚本；复杂业务→领域模型+数据映射器
- **DDIA**：可靠性/可扩展性/可维护性三大基石；B-Tree读优化，LSM-Tree写优化
- **DDD**：战略（限界上下文/通用语言/防腐层）+ 战术（实体/值对象/聚合根/领域服务/领域事件）
- **Clean Architecture**：依赖向内指向核心实体；框架/DB/UI都是可替换插件
- **POSA**：Reactor（就绪通知）vs Proactor（完成通知）；Netty = 主从Reactor + Half-Sync/Half-Async

**架构反模式**：简历驱动开发、金锤、分析瘫痪、大泥球、上帝服务

---

### 9. 开发规范与软件工艺（综合九本经典著作）

> 详细深度内容见 `references/coding-standards.md`



#### 9.1 规范哲学

**核心共识**：
- **可读性基本定理**（《编写可读代码的艺术》）：代码的写法应当使别人理解它所需的时间最小化
- **代码是写给人看的**（《Clean Code》）：只是偶尔在机器上执行。编程首先是与人交流，其次才是与计算机交流
- **软件构建远不止编码**（《代码大全》）：涵盖详细设计、编码、调试、测试集成、可读性与长期维护
- **DRY + 正交性**（《Pragmatic Programmer》）：不要重复自己，减少组件间耦合
- **不要容忍破窗**（《Pragmatic Programmer》）：看到糟糕的代码，马上修正，否则整个系统会迅速恶化
- **Brooks 法则**（《人月神话》）：向进度落后的项目中增加人手，只会使进度更加落后

#### 9.2 命名规范

**第一性原则**：名副其实，看到名字就知道为什么存在、做什么事、怎么用。

| 类型 | 规范 | 正例 | 反例 |
|------|------|------|------|
| 类名 | UpperCamelCase，名词 | `OrderDetail`、`UserService` | `orderDetail`、`Order_Detail` |
| 接口 | UpperCamelCase，形容词-able | `Translatable`、`Serializable` | — |
| 方法 | lowerCamelCase，动词/动宾 | `getUserById`、`saveOrder` | `GetUserById`、`get_user_by_id` |
| 变量 | lowerCamelCase，名词 | `elapsedDays`、`orderList` | `d`、`list` |
| 常量 | 全大写，下划线分隔 | `MAX_STOCK_COUNT` | `maxStockCount`、`MAXCOUNT` |
| 抽象类 | Abstract/Base 开头 | `AbstractUserService` | — |
| 异常类 | Exception 结尾 | `BusinessException` | — |
| 枚举 | Enum 后缀，成员全大写 | `DealStatusEnum.SUCCESS` | `DealStatus.success` |

**命名原则**（《Clean Code》+《可读代码艺术》）：
1. **名副其实**：`elapsedDays` 而非 `d`
2. **避免误导**：不用 `accountList` 表示非 List 类型
3. **做有意义的区分**：`ProductInfo` vs `ProductData` 无意义
4. **使用读得出来的名字**：`generationTimestamp` 而非 `genymdhms`
5. **使用可搜索的名字**：单字母和数字常量难以搜索
6. **名字长度与作用域匹配**：短名字适合小作用域（`i`），长名字适合大作用域（`maxConnectionsPerServer`）

#### 9.3 函数/方法规范

**核心原则**：函数应该短小，只做一件事，并做好一件事。

- **长度**：尽量控制在 50 行以内（《Clean Code》理想 20 行，《阿里手册》上限 80 行）
- **参数**：最理想的参数数量是 0，其次是 1，再次是 2，尽量避免 3 个及以上。多于 3 个时封装为对象
- **禁止 flag 参数**：如 `render(boolean isSuite)`，应拆分为 `renderForSuite()` 和 `renderForSingleTest()`
- **避免副作用**：函数承诺做一件事，就不要偷偷做其他事
- **单一职责**：一个函数只做同一抽象层上的步骤。如果一个函数既验证用户、又格式化名字、又保存数据，必须拆分

**示例**：
```java
// 反例：一个函数做了太多事
public void processUser(User user) {
    // 验证、格式化、检查权限、保存、发通知、记日志...
}

// 正例：每个函数只做一件事
public void processUser(User user) {
    validateUser(user);
    User formatted = formatUser(user);
    checkPermission(formatted, "edit");
    saveUser(formatted);
    sendNotification(formatted);
    logActivity(formatted);
}
```

#### 9.4 类与面向对象规范

- **规模**：类的总行数不超过 500 行，方法数不超过 20 个。过大的类按职责拆分
- **封装**：尽可能使每个类或成员不被外界访问（《Effective Java》第15条）。字段用 `private`，需要暴露的用 getter/setter
- **不可变对象优先**：除非有很好的理由让类可变，否则应设为不可变（《Effective Java》第17条）。不可变对象天然线程安全
- **复合优先于继承**：继承破坏封装，子类依赖父类实现细节（《Effective Java》第18条）
- **接口优先于抽象类**：接口允许非层次结构的类型框架，是定义 mixin 的理想选择（《Effective Java》第20条）

#### 9.5 注释规范

**第一性原则**：注释不能美化糟糕的代码。注释的恰当用法是弥补用代码表达意图时的失败。

**好注释**：解释意图（为什么）、警示后果、TODO、法律信息
**坏注释**：重复代码、喃喃自语、误导性注释、日志式注释（用版本控制替代）、归属署名

**Javadoc 要求**：
- 类、类属性、类方法的注释必须使用 `/** 内容 */` 格式
- 方法注释包含：方法说明、`@param` 参数说明、`@return` 返回值、`@throws` 异常

**注释内容**：说明"做什么""为什么"，而不是"怎么做"。

#### 9.6 格式规范

- **缩进**：4 个空格，禁止 Tab
- **行宽**：单行不超过 120 个字符
- **空格**：运算符左右必须加空格；`if`/`for`/`while` 保留字与括号之间加空格；括号内不加空格
- **空行**：不同逻辑、不同语义、不同业务的代码之间插入空行
- **一致性**：团队达成一致的风格，每个人都应遵循（《代码大全》）

#### 9.7 错误处理与防御式编程

**异常使用原则**（《Effective Java》+《Clean Code》）：
1. **只在异常情况下使用异常**，不要用异常做正常控制流
2. **对可恢复的情况使用受检异常**，对编程错误使用运行时异常
3. **优先使用标准异常**：`IllegalArgumentException`、`IllegalStateException`、`NullPointerException`
4. **不要忽略异常**：空的 catch 块是代码的毒瘤
5. **使用异常替代返回错误码**：异常可以将错误处理代码从主路径中分离

**空值处理**：
- 返回零长度数组或集合，而不是 null（《Effective Java》第54条）
- 使用 `Optional` 作为可能为空的返回值类型
- 方法入参用 `Objects.requireNonNull()` 检查

**日志规范**：
- 使用 SLF4J + Logback/Log4j2，不要直接 `System.out.println`
- 级别：ERROR（需要处理）、WARN（关注但可继续）、INFO（关键流程）、DEBUG（开发调试）
- 异常日志：`log.error("xxx failed, userId={}", userId, e)`，保留完整堆栈
- 禁止打印敏感信息（密码、身份证号、银行卡号）
- 生产环境禁止输出 debug 日志

#### 9.8 并发编程规范

- **同步访问共享的可变数据**（《Effective Java》第78条）
- **避免过度同步**：不要在同步区域内调用外来方法（《Effective Java》第79条）
- **线程池必须通过 `ThreadPoolExecutor` 创建**（《阿里手册》），禁止使用 `Executors` 的便捷方法（有 OOM 风险）
- **线程名称要有意义**，方便出错时回溯
- **优先使用 `java.util.concurrent`**： `ConcurrentHashMap` 优于 `Collections.synchronizedMap`，`BlockingQueue` 用于生产者-消费者模式
- **高并发时考量锁的性能损耗**：能用无锁数据结构就不用锁；能锁区块就不锁整个方法体；能用对象锁就不用类锁

#### 9.9 集合与泛型规范

- **不要使用原生类型**：使用 `List<String>` 而非 `List`（《Effective Java》第26条）
- **重写 equals 必须重写 hashCode**（《Effective Java》第11条）
- **`foreach` 里不要进行 remove/add 操作**，用 `Iterator`
- **集合初始化时指定初始容量**，避免频繁扩容
- **返回空集合而非 null**：`return Collections.emptyList()`

#### 9.10 重构与代码质量

**代码坏味道速查**（《重构》）：

| 坏味道 | 症状 | 处理 |
|--------|------|------|
| 重复代码 | 相同代码出现在多处 | 提炼方法 |
| 过长方法 | 超过50行，做多件事 | 拆分方法 |
| 过大类 | 超过500行，职责过多 | 提炼类 |
| 过长参数 | 超过3-4个参数 | 封装为对象 |
| 基本类型偏执 | 用 String/int 代替小对象 | 以对象取代基本类型 |
| Switch 惊悚 | 多处 switch/if-else 处理类型 | 以多态取代条件 |
| 过长消息链 | `a.getB().getC()` | 隐藏委托 |
| 纯数据类 | 只有字段和 getter/setter | 封装行为 |

**重构原则**：小步前进、频繁测试。每次改动后确保测试通过。

**破窗理论**（《Pragmatic Programmer》）：看到糟糕的代码，马上修正；如果没时间，至少标记 TODO/FIXME。容忍破窗会传递"质量不重要"的信号，导致整个系统恶化。

#### 9.11 测试规范

**单元测试 FIRST 原则**（《Clean Code》）：
- **F**ast（快速）：测试应该快速运行
- **I**ndependent（独立）：测试之间不相互依赖
- **R**epeatable（可重复）：任何环境下都能重复运行
- **S**elf-Validating（自验证）：布尔输出（通过/失败）
- **T**imely（及时）：测试应在生产代码之前编写（TDD）

**阿里 AIR 原则**：
- **A**utomatic（自动化）：全自动执行，非交互式
- **I**ndependent（独立性）：测试用例之间不互相调用
- **R**epeatable（可重复）：不受外界环境影响

**测试命名**：`methodName_should_expectedBehavior_when_scenario`

#### 9.12 数据库规范

- **索引**：WHERE、JOIN、ORDER BY 字段建立索引；避免冗余索引、过多索引
- **SQL 优化**：避免 `SELECT *`、避免隐式类型转换、分页深页码优化（延迟关联/覆盖索引/游标分页）
- **大表处理**：分库分表（ShardingSphere）、冷热分离、归档
- **事务**：尽量短小，避免长事务；查询尽量不放事务内；同一事务内操作顺序一致防死锁

#### 9.13 API 设计规范

- **RESTful**：资源名词（非动词）、HTTP 动词（GET/POST/PUT/DELETE/PATCH）、状态码准确
- **版本控制**：URL 路径 `/v1/users` 或 Header `Accept: application/vnd.api.v1+json`
- **统一响应格式**：`{code, message, data, timestamp}`
- **幂等性**：乐观锁（version）、唯一索引、Token 机制

#### 9.14 职业素养（九本书共同强调）

**承诺管理**（《Clean Coder》）：
- 知道自己能够承诺什么，并坚守承诺
- 无法承诺时勇敢说"不"，承诺了就必须全力以赴
- 不要牺牲代码质量来赶进度（破窗理论）

**持续学习**（《Pragmatic Programmer》）：
- 每年至少学习一种新语言，每季度阅读一本技术书籍
- 持续学习是程序员的核心竞争力

**时间管理**（《Clean Coder》+《人月神话》）：
- 区分优先级：紧急且重要 > 重要不紧急 > 紧急不重要
- Brooks 法则：向进度落后的项目增加人手，只会使进度更加落后

---

## 代码审查清单

审查代码时，按以下六维检查：

| 维度 | 检查要点 |
|------|----------|
| **可读性** | 命名名副其实？方法<50行、只做一件事？类<500行、职责单一？注释解释"为什么"？格式一致？ |
| **健壮性** | 参数有效性检查？异常路径处理？空指针风险？并发线程安全？资源正确释放？ |
| **设计质量** | 遵循 SOLID？不重复（DRY）？正交性（修改局部化）？合适的设计模式？复合优于继承？ |
| **性能** | 避免不必要对象创建？集合指定初始容量？避免循环查库？日志不影响性能？ |
| **安全** | SQL注入/XSS/越权防护？敏感数据脱敏？异常不泄漏内部实现？ |
| **测试** | 有对应单元测试？覆盖边界条件？测试独立、可重复、自验证？ |
| **Spring** | 事务正确？同类自调用导致失效？循环依赖合理？AOP生效？缓存一致性、穿透/击穿/雪崩防护？ |

### 10. 框架源码深度分析

#### Spring IoC 源码核心路径
- **`refresh()` 流程**: 准备环境 → 加载 BeanDefinition(解析 XML/注解/配置类) → `BeanFactoryPostProcessor` → 实例化单例 Bean → 完成刷新
- **配置类解析**: `@Configuration` 类由 `ConfigurationClassPostProcessor` 处理
  - 递归解析 `@ComponentScan`(ASM 读取类元数据)、`@Import`(ImportSelector/ImportBeanDefinitionRegistrar)、`@Bean` 方法
- **Bean 创建**: `doCreateBean()` → 实例化(构造器/工厂方法) → 属性填充(`populateBean`) → 初始化(`initializeBean`)
- **三级缓存解循环依赖**:
  ```
  singletonObjects: 成品 Bean
  earlySingletonObjects: 早期暴露(已实例化未填充属性)
  singletonFactories: 单例工厂(lambda 包装)，用于生成早期引用
  ```
  A→B→A 时，A 实例化后将自己放入 singletonFactories，B 注入 A 时从工厂获取早期引用
- **AOP 代理创建**: `AnnotationAwareAspectJAutoProxyCreator`(BeanPostProcessor 实现)
  - 解析 `@Aspect` 类中的通知方法，创建 Advisor 列表
  - 匹配目标方法的 Advisor，JDK/CGLIB 动态代理织入

#### Spring Transaction 源码
- **`@Transactional` 生效机制**: 基于 AOP 代理，事务拦截器 `TransactionInterceptor`
- **传播行为实现**: `TransactionSynchronizationManager` 维护当前线程的事务上下文(ThreadLocal)
- **回滚规则**: 默认 RuntimeException 和 Error 回滚，Checked Exception 不回滚(可配置 rollbackFor)
- **同类自调用失效原因**: 绕过代理对象直接调用，AOP 拦截器无法生效。解决方案: 注入自身代理或拆分到另一个 Bean

#### MyBatis 源码核心
- **Mapper 接口代理**: `MapperProxyFactory` 为接口生成 JDK 动态代理
  - `MapperProxy.invoke()` 拦截方法调用，封装为 `MapperMethod`
  - `MapperMethod.execute()` 根据 SQL 类型(SELECT/INSERT/UPDATE/DELETE) 调用 SqlSession
- **SQL 执行链**: `DefaultSqlSession` → `CachingExecutor`(二级缓存) → `BaseExecutor`(一级缓存) → `StatementHandler`
  - `SimpleExecutor`: 每次创建新 Statement
  - `ReuseExecutor`: Statement 复用(同一 SQL)
  - `BatchExecutor`: 批处理，JDBC `addBatch()`
- **参数处理**: `DefaultParameterHandler.setParameters()`，通过 `TypeHandler` 将 Java 类型转为 JDBC 类型
- **结果映射**: `DefaultResultSetHandler`，反射创建对象，通过 `TypeHandler` 映射列值到字段

#### Spring Boot 自动配置源码
- **`@EnableAutoConfiguration`**: 导入 `AutoConfigurationImportSelector`
- **加载机制**: `SpringFactoriesLoader.loadFactoryNames()` 读取 `META-INF/spring.factories` 中 `EnableAutoConfiguration` 键对应的类列表
- **条件过滤**: 各 AutoConfiguration 类上的 `@Conditional` 注解评估，不满足条件的配置类被过滤
- **属性绑定**: `@EnableConfigurationProperties` + `@ConfigurationProperties`，通过 `ConfigurationPropertiesBindingPostProcessor` 将配置文件绑定到 POJO

#### Netty 核心源码
- **EventLoop**: 继承 `ScheduledExecutorService`，每个 EventLoop 绑定一个线程，管理一个 Selector
  - `NioEventLoop.run()` 核心循环: select → processSelectedKeys → runAllTasks
- **ChannelPipeline**: 双向链表结构，入站事件(head→tail: channelRead、exceptionCaught)，出站事件(tail→head: write、bind、connect)
- **ByteBuf**: 引用计数(`refCnt`/`release`)，池化(`PooledByteBufAllocator`) 减少 GC，CompositeByteBuf 零拷贝组合多个 Buffer
- **内存分配**: jemalloc 算法灵感，PoolArena → PoolChunk(16MB) → PoolSubpage(小对象) / PoolChunkList(按使用率分类)

#### Redis 源码要点
- **SDS(Simple Dynamic String)**: 结构 `{len, alloc, flags, buf[]}`，O(1) 获取长度，预分配+惰性释放，二进制安全
- **字典**: 两个 ht[0]/ht[1]，渐进式 rehash(dictRehashStep，每次操作迁移少量)，负载因子 1 时开始，0.1(服务器空闲)时强制
- **跳表(SkipList)**: ZSet 底层，平均 O(logN) 查找，实现简单，通过随机层数(概率 1/2 递减)维持平衡
- **持久化 RDB**: `bgsave` fork 子进程写快照，Copy-On-Write 共享内存页，写时复制
- **AOF 重写**: `bgrewriteaof` fork 子进程，将当前内存状态转换为最短命令序列

## 源码分析方法论

分析任何框架源码时，遵循以下步骤：

1. **入口定位**: 找到核心入口类，如 Spring 的 `AbstractApplicationContext.refresh()`
2. **主流程追踪**: 跟随核心方法调用链，画出时序图，标注关键类和接口
3. **关键数据结构**: 理解核心类的字段含义，如 AQS 的 state、head、tail；HashMap 的 table、treeifyThreshold
4. **设计模式识别**: 标注源码中应用的设计模式(Spring 中近 20 种设计模式)
5. **扩展点挖掘**: 找出框架预留的扩展接口，如 BeanPostProcessor、HandlerInterceptor、Netty 的 ChannelHandler
6. **边界条件**: 关注异常处理路径、并发控制(锁/CAS)、资源释放(finally/try-with-resources)
7. **画图辅助**: 用 PlantUML 画类图、时序图、状态图，加深理解
