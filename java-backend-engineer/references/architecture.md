### 8. 软件架构设计（综合八本经典著作）

> 综合《软件架构实践》(Len Bass 等)、《架构整洁之道》(Robert C. Martin)、《企业应用架构模式》(Martin Fowler)、《面向模式的软件架构》(POSA 系列)、《大型网站技术架构：核心原理与案例分析》(李智慧)、《数据密集型应用系统设计》(Martin Kleppmann)、《领域驱动设计》(Eric Evans)、《架构师修炼之道》(Gregor Hohpe) 八本经典著作的核心思想。
>
> 架构设计是"在需求与约束之间做决策的艺术"。它既不是高不可攀的象牙塔，也不是随意堆砌的技术拼盘。优秀的架构应该像生物一样**演化**，而非像建筑一样一次成型。本章节旨在建立系统化的架构思维框架，帮助 Java 后端工程师从"写代码"走向"设计系统"。

#### 8.1 架构哲学与本质

**什么是软件架构**（《软件架构实践》+ Grady Booch）：
- 架构是系统的一组**结构**，包含软件元素、元素的外部可见属性、元素间的关系
- 架构是"塑造系统的重大设计决策"，其中"重大"由**变更成本**衡量
- 架构关注**非功能性需求**（质量属性），而代码关注功能性需求

**架构的目标**：
1. **管理复杂度**：通过分解、抽象、封装控制认知负载
2. **支持演化**：为未来的变化预留空间，降低修改成本
3. **满足质量属性**：在性能、可用性、安全性、可修改性之间取得平衡

**架构师的定位**（《架构师修炼之道》）：
- 架构师不是"画图匠"，而是**决策者与沟通者**。Gregor Hohpe 提出"电梯架构师"概念：能在 CEO 与程序员之间自由穿梭，将业务战略转化为技术决策
- 架构师的核心产出是**决策**（技术选型、边界划分、权衡取舍），而非文档或图表
- 架构即政治：技术决策影响组织结构，康威定律（Conway's Law）指出"设计系统的组织，其产生的设计等同于组织间的沟通结构"

#### 8.2 质量属性与架构评估

**质量属性场景**（《软件架构实践》）：
质量属性必须通过具体场景描述，格式为：
> **刺激源** → **刺激** → **环境** → **制品** → **响应** → **响应度量**

核心质量属性：

| 质量属性 | 关注点 | 典型战术 |
|----------|--------|----------|
| **性能** | 响应时间、吞吐量、资源利用率 | 缓存、异步、批处理、负载均衡、数据分区 |
| **可用性** | 故障恢复、冗余、容错 | 心跳检测、故障转移、降级、熔断、限流 |
| **安全性** | 机密性、完整性、不可否认性 | 认证授权、加密、审计、输入校验、WAF |
| **可修改性** | 变更成本、扩展性 | 封装、抽象、信息隐藏、配置文件化、插件架构 |
| **可测试性** | 自动化测试、故障隔离 | 接口隔离、依赖注入、Mock 对象、契约测试 |
| **可部署性** | 发布频率、回滚能力 | 蓝绿部署、金丝雀发布、容器化、基础设施即代码 |

**架构评估**（ATAM / SAAM）：
- **敏感点**：对实现特定质量属性至关重要的架构决策
- **权衡点**：影响多个质量属性的敏感点（如加密提升安全性但降低性能）
- **风险**：可能阻碍系统达成质量目标的架构决策
- **ATAM（架构权衡分析方法）**：通过场景驱动的方式，识别架构对质量属性的满足程度及潜在风险。四个核心步骤：
  1. 识别业务驱动与架构方法
  2. 生成质量属性效用树（Utility Tree）
  3. 分析架构方法对应的高优先级场景
  4. 头脑风暴与场景优先级排序，重新分析

> **原则**：没有最好的架构，只有最适合当前业务阶段和质量属性目标的架构。

#### 8.3 架构视图与可视化

**4+1 视图模型**（《软件架构实践》，Philippe Kruchten）：

| 视图 | 关注点 | 受众 | 典型内容 |
|------|--------|------|----------|
| **逻辑视图** | 功能需求 | 最终用户、分析师 | 包图、类图、领域模型 |
| **开发视图** | 软件管理 | 程序员、项目经理 | 模块划分、层次结构、代码库组织 |
| **过程视图** | 并发与同步 | 系统集成者 | 线程、进程、通信机制 |
| **物理视图** | 部署与拓扑 | 运维工程师 | 服务器、网络、中间件部署 |
| **场景视图** | 用例实现 | 所有干系人 | 关键用例的端到端时序 |

**C4 模型**（Simon Brown，与《架构师修炼之道》高度契合）：
> 使用四个层次表达不同粒度的架构，一图胜千言，不同受众看不同层。

1. **系统上下文图（System Context）**：系统与用户、外部系统的关系。受众：所有人
2. **容器图（Container）**：系统内的可部署单元（应用、数据库、消息队列）及通信。受众：技术负责人
3. **组件图（Component）**：容器内的组件及交互。受众：开发团队
4. **代码图（Code）**：类/接口级别的实现。受众：具体开发者（通常由 IDE 生成）

**架构决策记录 ADR**（Michael Nygard）：
- 格式：标题、上下文、决策、后果、合规
- 价值：记录"为什么这么做"，避免后人重复讨论或盲目推翻
- 存放：与代码库同仓库的 `docs/adr/` 目录

#### 8.4 架构风格与模式

> 架构风格是**全局性**的组织模式，架构模式是**局部性**的解决方案。以下风格在 Java 后端生态中最常见。

**分层架构（Layered Architecture）**（《企业应用架构模式》）：
```
表现层(Presentation) → 领域层(Domain) → 数据源层(Data Source)
```
- 核心规则：上层调用下层，禁止反向依赖；层间通过接口解耦
- 层数可调整：传统三层、DDD 四层（接口/应用/领域/基础设施）、Clean Architecture 四层
- 层可以是开放的（允许跳层）或封闭的（严格逐层）

**六边形架构 / 端口适配器（Hexagonal / Ports & Adapters）**：
- 应用为核心，外部适配器（Web、数据库、消息队列）通过端口连接
- 本质：将技术细节（框架、数据库）视为可替换的插件
- 与 DDD 结合：领域层位于中心，基础设施层实现仓储接口

**微服务架构**（结合第4章，理论升华）：
- 拆分原则：按业务边界（DDD 限界上下文）、而非技术分层拆分
- 数据库独立：每个服务拥有独立数据存储，禁止绕过服务直接访问数据库
- 通信模式：同步（REST/gRPC）用于查询，异步（消息/事件）用于命令和最终一致性
- 反模式：分布式单体（服务拆分了但数据库共享、强同步调用链）

**CQRS（命令查询职责分离）**：
- 命令（写）与查询（读）使用不同模型和数据库
- 适用：读多写少、读模型与写模型差异大、需要高性能查询
- 代价：数据同步延迟、系统复杂度增加

**事件溯源（Event Sourcing）**：
- 不以状态为事实来源，以**领域事件序列**为真相
- 当前状态通过对事件流重放获得
- 与 CQRS 天然配合：事件日志作为写模型，投影构建读模型

#### 8.5 企业应用架构模式（PEAA）

> Martin Fowler 在《企业应用架构模式》中提炼了 40+ 模式，以下是对 Java 后端最具实践价值的分类总结。

**领域逻辑模式**：

| 模式 | 核心思想 | 复杂度 | 数据库耦合 | Java 映射 |
|------|----------|--------|------------|-----------|
| **事务脚本** | 每个业务请求一个过程/方法 | 低 | 高（直接 SQL） | 早期 Servlet + JDBC |
| **表模块** | 围绕表组织逻辑，配合记录集 | 中 | 中 | .NET DataSet 风格，Java 中较少 |
| **领域模型** | 围绕领域名词建对象模型，行为与数据封装 | 高 | 低（通过映射器） | Spring + JPA/MyBatis + DDD |
| **服务层** | 在表现层与领域层之间增加边界，封装应用逻辑 | 中 | 中 | Spring Service（@Service） |

> **选型建议**：简单 CRUD 用事务脚本；复杂业务（规则多、状态多、关联多）必须用领域模型。

**数据源架构模式**：
- **表数据入口（Table Data Gateway）**：一个类封装对一张表的所有 SQL 操作。适合事务脚本
- **行数据入口（Row Data Gateway）**：为查询结果每一行生成一个对象。类似简单 DTO + DAO
- **活动记录（Active Record）**：对象既含数据又含 CRUD 方法（如 Rails、早期 JPA）。缺点是业务复杂时对象职责过重
- **数据映射器（Data Mapper）**：领域对象与数据库完全解耦，由映射器负责转换（MyBatis、Hibernate）。**复杂系统的首选**

**对象-关系行为模式**：
- **工作单元（Unit of Work）**：维护受业务事务影响的对象列表，协调变更写入与并发控制。Spring 的 `@Transactional` 本质
- **标识映射（Identity Map）**：确保同一事务内同一对象只加载一次，避免不一致
- **延迟加载（Lazy Load）**：按需加载关联对象。MyBatis 的 `fetchType="lazy"`、JPA 的 `FetchType.LAZY`

**分布与并发模式**：
- **数据传输对象 DTO**：跨进程传输数据的扁平对象，减少远程调用次数
- **远程外观（Remote Facade）**：为细粒度对象提供粗粒度远程接口
- **乐观离线锁**：通过版本号检测冲突（JPA `@Version`）
- **悲观离线锁**：显式加锁防止冲突（`SELECT ... FOR UPDATE`）

#### 8.6 数据密集型系统设计（DDIA）

> Martin Kleppmann 的《数据密集型应用系统设计》是后端工程师理解分布式数据系统的"圣经"。核心思想：数据是系统的中心，所有架构决策最终都归结为数据如何存储、复制、分区与访问。

**三大基石**：
1. **可靠性（Reliability）**：即使发生故障（硬件、软件、人为），系统仍能正确工作
   - 容错技术：冗余、故障转移、 graceful degradation
2. **可扩展性（Scalability）**：负载增长时，有合理手段保持性能
   - 描述负载：请求速率、读写比例、数据量、访问模式
   - 描述性能：响应时间中位数、p99、p999 尾部延迟、吞吐量
3. **可维护性（Maintainability）**：工程师能高效地在系统上工作
   - 可运维性（监控、自动化）、简单性（抽象隐藏复杂度）、可演化性（易于变更）

**数据模型选择**：

| 模型 | 特点 | 适用场景 | Java 生态 |
|------|------|----------|-----------|
| **关系型** | 强 schema、联结操作、事务支持 | 复杂查询、强一致性业务 | MySQL、PostgreSQL、Oracle |
| **文档型** | 灵活 schema、局部性、嵌套结构 | 内容管理、用户画像、 catalogs | MongoDB、Elasticsearch |
| **图型** | 顶点与边、高效遍历关联关系 | 社交网络、推荐、知识图谱 | Neo4j、JanusGraph |
| **键值型** | 简单、极高性能 | 缓存、会话、配置 | Redis、RocksDB |
| **宽列型** | 列族、高吞吐写、分布式 | 时序数据、日志、海量写入 | Cassandra、HBase |

> **原则**：没有银弹。Polyglot Persistence（多语言持久化）—— 根据数据访问模式选择最合适的存储。

**存储引擎原理**：
- **B-Tree**：读优化、范围查询友好、更新就地。传统 RDBMS 主流
- **LSM-Tree**：写优化、顺序写磁盘、合并压缩。LevelDB、RocksDB、Cassandra、HBase

**复制（Replication）**：
- **主从复制**：写主读从，异步延迟存在。MySQL、Redis 主从
- **多主复制**：多节点可写，冲突需解决。CouchDB、部分场景下的 MySQL Group Replication
- **无主复制**：Quorum 读写（R + W > N），容忍节点故障。Cassandra、Dynamo 风格

**分区/分片（Partitioning/Sharding）**：
- **键范围分区**：范围查询高效，但容易热点。如 HBase Region
- **哈希分区**：均匀分布，但范围查询需扫所有分区。如 Cassandra、Redis Cluster
- **二级索引**：本地索引（每个分区维护，查询需广播） vs 全局索引（跨分区，写放大）

**事务与一致性**：
- **ACID**：原子性、一致性、隔离性、持久性
- **隔离级别**：Read Uncommitted → Read Committed → Repeatable Read → Serializable。越往上并发越低、一致性越强
- **分布式事务**：2PC（两阶段提交，强一致但阻塞） vs Saga（长事务拆分，补偿机制，最终一致）
- **一致性谱系**：
  - 线性一致性（最强，所有操作如同在单一副本上原子执行）
  - 顺序一致性
  - 因果一致性
  - 最终一致性（最弱，但性能最好）

> **CAP 与 PACELC**：CAP 过于简化。PACELC 更精确：
> - 如果分区（P），必须在可用性（A）和一致性（C）之间选择；
> - 否则（E），必须在延迟（L）和一致性（C）之间选择。


#### 8.7 领域驱动设计（DDD）

> Eric Evans 于 2003 年出版的《领域驱动设计：软件核心复杂性应对之道》(Domain-Driven Design: Tackling Complexity in the Heart of Software) 是软件工程史上最具影响力的著作之一。DDD 的提出背景源于一个根本矛盾：**业务复杂性与技术复杂性纠缠在一起，导致系统难以理解和演进**。传统做法中，开发团队将业务逻辑平铺直叙地嵌入数据库脚本或事务脚本中，领域知识被技术细节稀释；而 DDD 主张将"领域"（Domain）——即业务问题本身——置于设计的核心位置，通过建立精准的领域模型来分离这两种复杂性。
>
> DDD 不是某种具体的架构风格（如分层、微服务），而是一套**连接业务与技术的通用语言与建模方法论**。它包含两个互补的维度：
> - **战略设计（Strategic Design）**：从全局视角回答"边界在哪里"——如何划分子域、识别限界上下文、定义上下文间的协作关系。
> - **战术设计（Tactical Design）**：从实现视角回答"边界内如何落地"——在单个限界上下文内部，用实体、值对象、聚合、领域服务等构建块构造可执行的领域模型。
>
> Evans 强调，DDD 的成功不取决于使用了多少模式，而取决于团队是否真正建立了**深入领域的洞察力**。模型不是对现实的被动描述，而是对业务问题的一种"有目的的简化"——它必须足够精确以指导实现，又足够抽象以忽略无关细节。

**DDD 的适用性判断**：

| 系统特征 | 是否适合 DDD | 替代方案 |
|----------|-------------|----------|
| 业务规则复杂、状态流转多、领域概念丰富 | ✅ 强烈推荐 | — |
| 简单 CRUD、无复杂业务逻辑 | ❌ 过度设计 | 事务脚本 / Active Record |
| 数据驱动、以报表查询为主 | ⚠️ 有限使用 | CQRS + 简单模型 |
| 强一致性要求极高、事务跨多聚合频繁 | ⚠️ 需谨慎设计 | 适当放宽聚合边界或引入 Saga |

---

##### 8.7.1 战略设计：划分业务边界

战略设计的本质是**在问题空间（Problem Space）中识别子域，并在解决方案空间（Solution Space）中将其映射为限界上下文**。这是 DDD 最具价值的部分，也是最容易被忽视的——许多团队急于使用战术构建块（聚合、实体等），却未先澄清战略边界，最终导致"模型很漂亮，但上下文串台"。

**子域（Subdomain）——问题空间的分解**：

Evans 将业务领域划分为三类子域，以指导资源分配优先级：

| 类型 | 定义 | 战略优先级 | 典型示例 | 建设策略 |
|------|------|-----------|----------|----------|
| **核心域（Core Domain）** | 企业的核心竞争力所在，是最具差异化价值的业务部分 | 最高 | 电商的订单履约引擎、金融的风控模型、物流的路径规划算法 | 投入最精锐团队，应用完整 DDD 战术设计，持续精炼模型 |
| **支撑域（Supporting Subdomain）** | 业务必需但非核心，无差异化竞争优势 | 中等 | 商品类目管理、营销活动配置、内容审核流程 | 可内部开发，但不必过度设计；或外包定制 |
| **通用域（Generic Subdomain）** | 行业通用能力，已有成熟解决方案 | 最低 | 用户认证、权限管理、消息通知、文件存储 | 直接采购 SaaS 或开源方案（如 Keycloak、MinIO），避免重复造轮子 |

> **关键洞察**：一个组织的核心域很少超过 2-3 个。识别核心域的试金石是："如果竞争对手在这部分做得比我们好，我们是否会被颠覆？"

**限界上下文（Bounded Context）——解决方案空间的边界**：

限界上下文是 DDD 中最核心的战略模式。它定义了一个**语义一致性边界**，在该边界内：
- 领域模型是统一且自洽的；
- 通用语言有明确且无歧义的定义；
- 模型可以独立演化，不受外部概念干扰。

Evans 提出这一概念的深层动机是对"统一企业模型"神话的批判。在大型系统中，试图建立一个覆盖全公司的统一领域模型是根本不可能的——"用户"在订单上下文是"买家"（关注收货地址、信用等级），在权限上下文是"账户"（关注角色、ACL），在 CRM 上下文是"潜在客户"（关注线索阶段、跟进记录）。强行统一只会得到一个臃肿、矛盾、无人敢改的大泥球。

**限界上下文的识别信号**：
- 同一术语在不同团队口中含义不同；
- 某个业务概念的生命周期与其他概念明显分离；
- 存在自然的组织边界（不同团队负责）；
- 数据库 Schema 中某组表几乎只被特定模块访问。

**限界上下文与微服务的关系**：
> 限界上下文是**概念边界**，微服务是**部署边界**。理想情况下二者一一对应，但现实中受团队规模、运维成本、网络延迟约束，可能将多个紧密关联的限界上下文合并为一个服务（模块化单体），或反之将一个上下文拆分为多个服务（如读写分离）。但**一个微服务绝不应包含多个互不协调的领域模型**——这是分布式单体的典型症状。

**通用语言（Ubiquitous Language）——领域知识的显性化**：

通用语言是开发团队与领域专家共享的词汇表，它必须**直接体现在代码中**。Evans 强调："如果领域专家无法理解代码中的类名和方法名，那模型就是错的。"

通用语言的实践原则：
1. **无处不在**：从需求文档、站会讨论、白板草图到单元测试命名，使用同一套术语。
2. **拒绝翻译**：不在业务语言与技术语言之间建立映射层。如果业务方说"核销优惠券"，代码中就应该是 `Coupon.verify()`，而非 `CouponService.updateStatus()`。
3. **随模型演进**：当发现现有词汇无法精确描述业务时，应与领域专家共同创造新词或精确定义，而非在代码中"绕过去"。
4. **代码即文档**：类的命名、方法的命名、异常的命名都应承载业务语义。如 `OrderCannotBeCancelledException` 优于 `BusinessException(1001)`。

**上下文映射（Context Mapping）——边界间的协作模式**：

限界上下文不是孤岛。上下文映射描述了它们之间的**团队协作关系**与**技术集成模式**。Evans 与后续研究者（如 Vaughn Vernon）共归纳出 9 种模式，可分为两大类：

| 类别 | 模式 | 定义 | 团队协作特征 | 技术实现 | 适用场景 |
|------|------|------|-------------|----------|----------|
| **团队协作模式** | **合作关系（Partnership）** | 两个上下文紧密协作，一荣俱荣 | 双方团队同步计划、联合发布，存在循环依赖风险 | 同仓库或紧耦合接口，频繁联调 | 两个新构建的上下文，业务上强耦合 |
| | **共享内核（Shared Kernel）** | 两个上下文共享一部分核心模型和代码 | 极高信任度，任何变更需双方评审 | 共享 Maven 模块 / 公共包 | 紧密关联的子域，如订单与支付共享 `Money` 值对象 |
| | **客户-供应商（Customer-Supplier）** | 上游（供应商）为下游（客户）提供能力 | 下游有话语权，可向上游提需求；上游有义务满足 | 版本化 API、契约测试 | 内部平台团队与业务团队的典型关系 |
| | **遵奉者（Conformist）** | 下游无条件接受上游模型 | 下游无影响力，只能被动跟随 | 直接消费上游模型，无转换 | 集成外部成熟系统（如 Amazon Marketplace API） |
| | **分离方式（Separate Ways）** | 两个上下文完全独立，无集成 | 无协作成本 | 无技术集成 | 集成成本高于各自重复实现的收益 |
| **技术集成模式** | **防腐层（Anti-Corruption Layer, ACL）** | 在下游建立翻译层，将上游模型转换为本上下文模型 | 下游保护自身模型纯净 | 适配器 + 翻译器 + Facade；如 `ExternalCRMClientAdapter` → `CustomerTranslator` → `Customer` | 集成遗留系统、外部频繁变更系统、模型差异大的场景。**这是战略设计中最常用的模式** |
| | **开放主机服务（Open Host Service, OHS）** | 上游以标准化协议向所有下游提供服务 | 上游主动降低集成门槛 | REST API / gRPC / GraphQL；配合版本控制 | 中台、平台型服务向多个业务方暴露能力 |
| | **发布语言（Published Language, PL）** | 定义上下文间交换数据的明确、稳定契约 | 双方围绕契约协作 | OpenAPI / Protobuf Schema / Avro / 领域事件 Schema | 常与 OHS 联用，或事件驱动架构中的事件契约 |
| | **大泥球（Big Ball of Mud）** | 遗留系统缺乏清晰边界，模型混杂 | 尽量避免或渐进式剥离 | 包裹层（Wrapper）、绞杀者模式（Strangler Fig） | 对接历史遗留系统时的临时状态 |

> **模式选择原则**：没有最好的模式，只有最适合当前团队结构、变更频率和自治需求的模式。防腐层增加了开发成本但保护了演进自由度；遵奉者降低了集成成本但丧失模型自主权。架构决策应记录 ADR。

---

##### 8.7.2 战术设计：边界内落地

战术设计关注在单个限界上下文内部，如何用面向对象技术构造表达力强的领域模型。Evans 定义的**战术构建块（Building Blocks）** 是 DDD 最具辨识度的部分，但它们的真正价值不在于形式上的使用，而在于**对业务不变量（Invariants）的精确封装**。

**实体（Entity）——连续性身份标识**：

实体是 DDD 中最基础的构建块，其本质特征不是"有 ID"，而是**具有跨时间的连续性身份（Identity）**。两个属性完全相同的订单，只要 orderId 不同，就是不同的实体；而两个金额和币种都相同的 `Money`，则是同一个值对象。

实体的设计原则：
- **标识设计**：标识应全局唯一且与业务无关。优先使用 UUID、雪花算法等，避免使用业务字段（如手机号）作为主键——业务规则会变，身份不应变。
- **生命周期管理**：实体通常经历创建 → 活动 → 归档/删除的阶段。在代码中体现为状态机（如 `OrderStatus`：CREATED → PAID → SHIPPED → COMPLETED）。
- **行为封装**：实体不应是"贫血模型"（只有 getter/setter 的数据容器），而应封装与其状态相关的业务规则。如 `order.cancel()` 内部检查 `status == CREATED`，而非在 Service 中写 `if (order.getStatus() == CREATED)`。

```java
// 反模式：贫血实体
public class Order {
    private Long id;
    private OrderStatus status;
    // 只有 getter/setter，无业务行为
}

// 正模式：充血实体，封装不变量
public class Order {
    private final OrderId id;
    private OrderStatus status;
    private List<OrderLine> lines;
    private Money totalAmount;

    public static Order create(OrderId id, List<OrderLine> lines) {
        // 工厂方法，确保创建时即有效
        return new Order(id, lines, OrderStatus.CREATED);
    }

    public void pay(Money amount) {
        if (this.status != OrderStatus.CREATED) {
            throw new OrderStatusException("Only CREATED order can be paid");
        }
        if (!amount.equals(this.totalAmount)) {
            throw new PaymentAmountMismatchException();
        }
        this.status = OrderStatus.PAID;
        // 记录领域事件
        registerEvent(new OrderPaidEvent(this.id, amount, Instant.now()));
    }

    // 内部聚合方法，外部不可直接调用 OrderLine.setQuantity()
    void addLine(Product product, int quantity, Money unitPrice) {
        this.lines.add(new OrderLine(product, quantity, unitPrice));
        recalculateTotal();
    }
}
```

**值对象（Value Object）——属性的不可变组合**：

值对象是没有概念标识、由属性值完全定义的对象。它通常**不可变（Immutable）**——创建后不可修改，需要变更时替换为新实例。值对象是 DDD 中最被低估的构建块，大量本应是值对象的概念被错误建模为实体（受数据库表思维影响）。

值对象的设计原则：
- **不可变性**：所有字段 `final`，无 setter，通过构造函数或工厂方法创建。
- **值相等性**：`equals()` 和 `hashCode()` 基于全部属性，而非对象引用或 ID。
- **自验证**：在构造时验证合法性，确保"无效的值对象无法存在"。如 `Money` 不允许负金额，`Email` 必须符合格式。
- **细粒度复用**：值对象可以嵌入多个实体中。如 `Address` 可同时属于 `Order`（收货地址）和 `Customer`（注册地址），彼此独立修改。

```java
@Value // Lombok 自动生成不可变语义
public class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        this.amount = amount;
        this.currency = Objects.requireNonNull(currency);
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new CurrencyMismatchException();
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    // equals/hashCode 基于 amount + currency
}
```

> **实体 vs 值对象判断法则**："如果更换了所有属性，它还是同一个东西吗？" 如果是（如换了手机号的人还是同一人）→ 实体；如果否（如换了金额和币种的钱就是不同的钱）→ 值对象。

**聚合（Aggregate）——一致性边界与事务边界**：

聚合是 DDD 战术设计中最重要、也最容易被误用的模式。它是一个**由聚合根和内部实体/值对象组成的集群**，被当作单个单元进行数据变更。聚合划定了**事务一致性边界**：在一个事务中，只允许修改一个聚合。

聚合的核心规则（Evans 原始定义 + Vernon 补充）：
1. **聚合根规则**：外部对象只能引用聚合根，不能直接引用聚合内部成员。
2. **事务边界规则**：一个事务只修改一个聚合实例。如果需要修改多个聚合，通过**领域事件**实现最终一致性。
3. **标识全局规则**：聚合根在全局范围内有唯一标识，内部实体只在聚合局部范围内有标识。
4. **删除级联**：删除聚合根必须级联删除聚合内所有对象。

聚合设计的黄金法则——**小聚合原则**：
> Vaughn Vernon 在《实现领域驱动设计》中强调："大聚合是性能杀手和并发瓶颈。" 聚合应尽可能小，只包含在业务上必须同时修改的对象。

聚合设计决策树：
```
A 和 B 是否必须在同一个事务中一起修改？
  ├─ 是 → 它们属于同一个聚合
  │       └─ A 是业务操作的入口？
  │           ├─ 是 → A 是聚合根
  │           └─ 否 → 重新考虑边界
  └─ 否 → 它们属于不同聚合，通过领域事件协作
```

典型误区与修正：

| 误区 | 问题 | 修正方案 |
|------|------|----------|
| "用户及其所有订单"作为一个聚合 | 订单量增长后聚合巨大，加载慢、并发冲突高 | `User` 是一个聚合，`Order` 是另一个聚合，`Order` 通过 `userId`（值对象）关联用户 |
| 聚合根持有大量子实体集合 | 每次加载聚合都全量加载子集合 | 使用延迟加载或拆分为独立聚合；如 `Product` 与 `ProductReview` 分离 |
| 跨聚合直接引用（持有对象引用） | 破坏边界，可能导致意外修改其他聚合 | 跨聚合引用使用 ID（值对象），如 `Order` 持 `List<ProductId>` 而非 `List<Product>` |
| 事务跨越多个聚合 | 违反聚合原则，数据库锁竞争加剧 | 使用 Saga / 事务消息实现最终一致性 |

```java
// 聚合示例：Order Aggregate
public class Order extends AggregateRoot<OrderId> {
    private OrderId id;
    private CustomerId customerId;      // 跨聚合引用：只存 ID
    private List<OrderLine> lines;      // 内部实体，外部不可直接访问
    private ShippingAddress address;    // 值对象
    private OrderStatus status;
    private List<DomainEvent> events;   // 领域事件集合

    // 聚合根提供所有外部可见的操作
    public void changeShippingAddress(ShippingAddress newAddress) {
        if (this.status != OrderStatus.CREATED) {
            throw new OrderModificationException("Cannot change address after payment");
        }
        this.address = newAddress;
    }
}

// OrderLine 是聚合内部实体，无全局标识
class OrderLine {
    private int lineNumber;     // 仅在 Order 聚合内唯一的局部标识
    private ProductId productId; // 跨聚合引用用 ID
    private int quantity;
    private Money unitPrice;

    // 包级可见，防止外部直接实例化
    OrderLine(ProductId productId, int quantity, Money unitPrice) {
        if (quantity <= 0) throw new IllegalArgumentException();
        this.productId = productId;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }
}
```

**领域服务（Domain Service）——跨实体的无状态业务逻辑**：

当业务逻辑**不属于任何一个实体或值对象**时，应建模为领域服务。领域服务是无状态的，只封装纯业务规则。

**与应用服务（Application Service）的严格区分**：

| 维度 | 领域服务（Domain Service） | 应用服务（Application Service） |
|------|--------------------------|------------------------------|
| **职责** | 封装业务规则与领域逻辑 | 编排用例流程、控制事务边界、调用基础设施 |
| **依赖** | 不依赖基础设施（无 `@Autowired` Repository） | 依赖 Repository、MQ、缓存等 |
| **状态** | 无状态 | 无状态 |
| **示例** | `PricingService.calculateDiscount()`、`TransferService.transfer()` | `OrderAppService.createOrder()`（校验 → 调用领域服务 → 保存 → 发布事件） |
| **测试** | 纯单元测试，无需 Mock 基础设施 | 需 Mock 外部依赖，集成测试覆盖 |

```java
// 领域服务：纯业务逻辑
public class PricingService {
    public Money calculateDiscount(Order order, Coupon coupon) {
        // 仅包含计算规则，不涉及数据库或事务
        if (!coupon.isApplicableTo(order)) {
            return Money.zero();
        }
        return coupon.apply(order.getTotalAmount());
    }
}

// 应用服务：用例编排
@Service
@Transactional
public class OrderAppService {
    private final OrderRepository orderRepository;
    private final PricingService pricingService;
    private final EventPublisher eventPublisher;

    public OrderId createOrder(CreateOrderCommand command) {
        Order order = Order.create(/* ... */);
        if (command.getCouponId() != null) {
            Coupon coupon = couponRepository.findById(command.getCouponId());
            Money discount = pricingService.calculateDiscount(order, coupon);
            order.applyDiscount(discount);
        }
        orderRepository.save(order);
        eventPublisher.publish(order.getEvents());
        return order.getId();
    }
}
```

**领域事件（Domain Event）——解耦与最终一致性的基石**：

领域事件是"领域中已发生的有意义的事情"，由聚合在执行业务操作后发布。它是实现**跨聚合解耦**和**最终一致性**的核心机制。Eric Evans 的原著中并未正式引入领域事件，这一概念由 Udi Dahan、Vaughn Vernon 等后续补充进 DDD 生态，现已成为不可或缺的部分。

领域事件的设计原则：
- **过去时命名**：`OrderPaidEvent`、`InventoryDeductedEvent`、`ShipmentDeliveredEvent`，明确表示"已发生"。
- **不可变性**：事件一旦产生不可修改，只可增加属性。
- **包含上下文信息**：至少包含事件源标识、发生时间、关键属性；但不应包含整个聚合状态（保持轻量）。
- **在聚合内创建，在应用层发布**：聚合负责生成事件，应用层负责通过消息基础设施派发。

```java
public class OrderPaidEvent extends DomainEvent {
    private final OrderId orderId;
    private final Money paidAmount;
    private final Instant occurredOn;

    // 构造函数 + getter，无 setter
}

// 聚合内注册事件
public class Order extends AggregateRoot<OrderId> {
    public void pay(Money amount) {
        // ... 业务逻辑 ...
        registerEvent(new OrderPaidEvent(this.id, amount, Instant.now()));
    }
}

// 订阅方：库存上下文通过事件实现最终一致性
@Component
public class OrderPaidEventHandler {
    @EventListener
    @Transactional
    public void on(OrderPaidEvent event) {
        InventoryReservation reservation = inventoryService
            .findReservationByOrderId(event.getOrderId());
        reservation.confirm(); // 确认扣减，非预占
        inventoryRepository.save(reservation);
    }
}
```

> **最终一致性 vs 强一致性**：聚合内保证强一致性（单事务）；聚合间通过领域事件实现最终一致性。不要为了追求全局强一致性而设计巨型聚合——这是 DDD 中最常见的反模式。

**仓储（Repository）——聚合的持久化抽象**：

仓储为聚合提供持久化能力，同时**屏蔽数据存储细节**。领域层定义仓储接口，基础设施层实现。这保证了领域层不依赖具体数据库技术。

仓储设计的两种语义风格：

| 风格 | 特征 | 示例 | 适用场景 |
|------|------|------|----------|
| **面向集合（Collection-Oriented）** | 模拟内存集合，调用方不感知持久化；`add()` 仅用于新增，`remove()` 删除 | `orderRepository.add(order)` | 领域驱动优先，聚合有明确的修改跟踪机制（如 Hibernate 的脏检查） |
| **面向持久化（Persistence-Oriented）** | 显式 `save()` 保存变更，`find()` 查询 | `orderRepository.save(order)` | 显式控制持久化时机，更适合 MyBatis、Spring Data JDBC |

```java
// 领域层：仅定义接口
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    Order save(Order order);
    List<Order> findByCustomerId(CustomerId customerId, PageParam page);
    // 注意：仓储只操作聚合根，没有 findOrderLineById()
}

// 基础设施层：技术实现
@Repository
public class OrderRepositoryImpl implements OrderRepository {
    private final OrderMapper orderMapper;    // MyBatis
    private final OrderLineMapper lineMapper;

    @Override
    public Order save(Order order) {
        if (orderMapper.exists(order.getId())) {
            orderMapper.update(order);
            lineMapper.deleteByOrderId(order.getId());
        } else {
            orderMapper.insert(order);
        }
        for (OrderLine line : order.getLines()) {
            lineMapper.insert(line);
        }
        return order;
    }
}
```

**工厂（Factory）——封装复杂创建逻辑**：

当聚合的创建逻辑复杂（涉及多步校验、默认值计算、关联对象初始化）时，应使用工厂封装，避免在客户端暴露创建细节。

```java
public class OrderFactory {
    private final ProductRepository productRepository;

    public Order createOrder(CustomerId customerId, List<LineItemRequest> items) {
        OrderId orderId = OrderId.generate();
        Order order = new Order(orderId, customerId);
        for (LineItemRequest item : items) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));
            order.addLine(product, item.getQuantity(), product.getCurrentPrice());
        }
        return order;
    }
}
```

---

##### 8.7.3 DDD 分层架构（Java 映射）

Evans 在书中提出了一种参考性的分层架构，后续社区演进出了多种变体（四层、五层、六边形、洋葱架构）。以下是在 Java/Spring 生态中最常用的 DDD 四层架构：

```
┌─────────────────────────────────────────────────────────────┐
│  用户接口层（Interfaces / Presentation）                      │
│  Controller / MQ Listener / RPC Provider / Schedule Job      │
│  职责：协议转换、入参校验、权限预检、返回DTO组装              │
├─────────────────────────────────────────────────────────────┤
│  应用层（Application）                                        │
│  ApplicationService / CommandHandler / QueryHandler          │
│  职责：用例编排、事务边界控制、发布领域事件、发送命令          │
│  原则：薄层，不含业务规则，只负责"调用谁、顺序是什么"         │
├─────────────────────────────────────────────────────────────┤
│  领域层（Domain）  ← 核心，最稳定                             │
│  Entity / ValueObject / Aggregate / DomainService            │
│  Repository Interface / DomainEvent / Policy / Specification │
│  职责：封装全部业务规则、不变量、领域逻辑                      │
│  原则：不依赖其他任何层，只依赖通用库（如 Lombok、Guava）     │
├─────────────────────────────────────────────────────────────┤
│  基础设施层（Infrastructure）                                 │
│  Repository Impl / MQ Producer / Cache Client / Http Client  │
│  Config / Security / Persistence Mapper                      │
│  职责：技术实现、外部系统集成、框架配置                        │
│  原则：依赖倒置——实现领域层定义的接口                         │
└─────────────────────────────────────────────────────────────┘
```

**关键依赖规则**：
- 箭头方向即源码依赖方向：接口层 → 应用层 → 领域层 ← 基础设施层
- 领域层位于最中心，不依赖 Spring、MyBatis 等框架注解（可用少量 Lombok 减少样板代码）。
- 跨层调用通过接口或领域事件解耦。

**Maven 多模块实践映射**：
```
order-service/
├── order-application/      # 应用层，依赖 domain
│   └── src/.../service/OrderAppService.java
├── order-domain/           # 领域层，无外部依赖（除通用库）
│   └── src/.../aggregate/Order.java
│   └── src/.../service/PricingService.java
│   └── src/.../repository/OrderRepository.java
├── order-infrastructure/   # 基础设施层，依赖 domain + application
│   └── src/.../repository/OrderRepositoryImpl.java
│   └── src/.../mapper/OrderMapper.java
└── order-interfaces/       # 接口层，依赖 application
    └── src/.../controller/OrderController.java
```

---

##### 8.7.4 DDD 实践原则与常见反模式

**核心实践原则**：
1. **先战略，后战术**：在未识别限界上下文前，不要急于设计聚合。
2. **事件风暴（Event Storming）优先于代码**：Alberto Brandolini 提出的事件风暴工作坊是识别聚合、领域事件、限界上下文的高效协作工具——比在白板上画类图更接近业务本质。
3. **从聚合开始设计，而非数据库表**：先讨论"业务操作的入口是什么"（聚合根），再推导持久化结构。
4. **测试驱动领域模型**：每个聚合的不变量、每个领域服务的规则，都应有直接触发失败路径的单元测试。否则不变量只是"写在纸上的规则"。
5. **允许模型演化**：DDD 不是一次性完美设计。随着业务理解加深，小的聚合可以合并，大的聚合可以拆分，上下文边界可以调整。

**常见反模式**：

| 反模式 | 表现 | 后果 | 修正 |
|--------|------|------|------|
| **贫血领域模型** | 实体只有 getter/setter，业务逻辑全部在 Service 中 | 业务规则分散，难以复用和测试 | 将状态校验、计算逻辑下沉到实体 |
| **巨型聚合** | 一个聚合包含数十个实体，加载整个对象图 | 性能极差，并发冲突频繁 | 按事务边界拆分，跨聚合用事件 |
| **跨聚合引用对象** | `Order` 直接持有 `Product` 对象引用 | 意外修改其他聚合，事务失控 | 跨聚合只引用 ID |
| **数据库驱动设计** | 先设计表结构，再生成领域对象 | 领域模型被持久化细节绑架，缺乏表达力 | 先建模，再映射（如 JPA 的 `@Embeddable`、`@Embedded`） |
| **万能 Service** | 一个 `OrderService` 上千行，包含所有业务逻辑 | 失去封装，变更影响范围不可控 | 按职责拆分为领域服务、应用服务、策略对象 |
| **过度 DDD** | 在简单 CRUD 系统中强行使用聚合、领域事件 | 引入不必要的复杂度，开发效率降低 | 简单系统用事务脚本，复杂系统才用 DDD |

> **Evans 的忠告**："DDD 是为复杂系统而生的。如果你的系统并不复杂，不要用 DDD 把它变复杂。"

**DDD 与相关模式的协同**：
- **CQRS**：将读模型与写模型分离。DDD 聚合负责写（命令），查询走专门的 Qry Repository 或直接读数据库视图，避免为查询需求污染聚合设计。
- **事件溯源（Event Sourcing）**：以领域事件序列作为聚合状态的单一事实来源。DDD 的 `registerEvent()` 天然为事件溯源奠基，但事件溯源是可选增强而非 DDD 必需。
- **六边形架构 / Clean Architecture**：DDD 分层是逻辑分层，六边形架构强调"技术适配器可插拔"，二者在"领域层独立"这一原则上高度一致，可无缝结合。
- **Saga 模式**：当业务用例必须修改多个聚合时，使用 Saga（编排式或协同式）管理跨聚合的最终一致性，补偿失败操作。

#### 8.8 大型网站技术架构

> 李智慧的《大型网站技术架构》是中国互联网架构演进的经典总结。其核心洞察：**优秀的架构不是设计出来的，而是演进来的**。在不同业务阶段选择最合适的技术方案，拒绝过度设计。

**架构演进路径**：
```
单体应用（LAMP/SSM）
  → 应用与数据分离（动静分离、读写分离）
  → 缓存引入（本地缓存 → 分布式缓存）
  → 应用服务集群（负载均衡、Session 共享）
  → 数据库读写分离与分库分表
  → 分布式服务化（SOA / 微服务）
  → 异步化与消息驱动
  → 云原生（容器化、DevOps、Serverless）
```

**高性能架构**：

| 层级 | 手段 | 具体技术 |
|------|------|----------|
| **前端** | 减少请求、就近访问 | CDN、浏览器缓存、页面压缩、懒加载、合并请求 |
| **接入层** | 反向代理、负载均衡 | Nginx、Spring Cloud Gateway、DNS 轮询 |
| **应用层** | 缓存、异步、代码优化 | Redis/Memcached、Caffeine、MQ 削峰、线程池优化 |
| **数据层** | 索引、分片、读写分离 | MySQL 主从、ShardingSphere、Elasticsearch |

**高可用架构**：
- **冗余**：服务器集群、数据库主从、多机房部署
- **故障转移**：Keepalived VIP、Nacos 健康检查自动摘除、Redis Sentinel
- **降级**：开关控制关闭非核心功能（如推荐、评论），保障下单支付
- **限流**：Sentinel 流量控制、Nginx limit_req、API Gateway 配额
- **熔断**：失败率达到阈值后快速失败，防止雪崩（Sentinel / Hystrix 思想）
- **监控**：Metrics（Prometheus）+ Logging（ELK）+ Tracing（SkyWalking），可观测性三支柱

**伸缩性（Scalability）**：
- **应用层**：无状态设计，请求任意路由；水平扩容只需加机器
- **缓存层**：一致性 Hash 减少节点变动时的缓存失效范围
- **数据库层**：分库分表（水平拆分）、自动分片重平衡

**安全性**：
- XSS：对用户输入转义，CSP 策略
- CSRF：Token 验证、SameSite Cookie
- SQL 注入：预编译语句（`#{}`），禁止字符串拼接 SQL
- DDoS：流量清洗、CDN 抗量、限频

> **反模式**：小团队、低流量阶段盲目上微服务、中台、Service Mesh，导致开发效率暴跌。

#### 8.9 并发与网络架构模式（POSA）

> 《面向模式的软件架构》(Pattern-Oriented Software Architecture, POSA) 系列是并发与网络编程模式的权威参考。以下模式深刻影响了 Java NIO、Netty、Tomcat 等核心基础设施的设计。

**Reactor（反应器）模式**：
- **核心**：同步事件多路分解 + 分发。单个线程监听多路事件，分发给对应的事件处理器
- **组件**：Reactor（事件循环）、Acceptor（接收连接）、Handler（处理读写）
- **Java 映射**：Java NIO Selector、Netty EventLoop、Tomcat NioEndpoint
- **变体**：单 Reactor 单线程、单 Reactor 多线程、主从 Reactor（Main-Sub，Netty 默认）

**Proactor（主动器）模式**：
- **核心**：异步 I/O，操作系统完成读写后通知应用。应用发起异步操作后立即返回，由内核回调
- **对比 Reactor**：Reactor 是"就绪通知"（应用自己读），Proactor 是"完成通知"（内核已读好数据）
- **Java 映射**：Java AIO（AsynchronousChannel）；Linux io_uring 是新一代高性能异步 I/O 代表

**Half-Sync/Half-Async（半同步/半异步）**：
- **分层**：异步层（I/O 事件捕获）→ 队列 → 同步层（业务逻辑处理）
- **优势**：异步层保证高并发不阻塞，同步层保证业务逻辑简单编写
- **Java 映射**：Netty 的 I/O 线程（异步）与业务线程池（同步）分离；Tomcat Nio + 业务线程池

**Leader/Followers（领导者/追随者）**：
- **机制**：线程池中的线程轮流担任 Leader 监听事件，事件发生则推选新 Leader，原 Leader 变为 Worker 处理事件
- **优势**：无锁共享事件队列，减少线程上下文切换和数据拷贝
- **缺点**：仅支持单一事件源集合，无法让每个线程独立管理多个连接
- **Java 映射**：某些高性能网络库（如 ACE、Taf）的线程模型

**Active Object（主动对象）**：
- **核心**：将方法调用（接口）与方法执行（实现）解耦，通过调度器异步执行
- **Java 映射**：`java.util.concurrent.Executor` 框架、`ExecutorService.submit()` 本质就是 Active Object

**在 Java 生态中的综合映射**：
```
Netty 线程模型 = 主从 Reactor + Half-Sync/Half-Async
  - MainReactor（BossGroup）：监听端口，Accept 连接
  - SubReactor（WorkerGroup）：处理 I/O 读写（异步层）
  - 业务线程池：处理编解码与业务逻辑（同步层）
```

#### 8.10 整洁架构与边界（Clean Architecture）

> Robert C. Martin（Bob 大叔）在 2017 年出版的《架构整洁之道》(Clean Architecture: A Craftsman's Guide to Software Structure and Design) 中，综合了 Alistair Cockburn 的六边形架构（Hexagonal Architecture，2005）、Jeffrey Palermo 的洋葱架构（Onion Architecture，2008）以及他自己数十年对 SOLID 原则的研究，提出了整洁架构。三者的核心思想高度一致：**用依赖规则隔离业务逻辑与技术细节，让框架、数据库、UI 都成为可替换的插件**。
>
> Bob 大叔在书中开宗明义：软件架构设计本身就是**划分边界的艺术**。边界的作用是将软件分割成各种元素，以便约束边界两侧之间的依赖关系。一个系统的架构对其行为的影响并不大——架构不能限制太多行为选项——但一个设计良好的架构能在行为上起到一个最重要的作用：**明确和显式地反映系统的设计意图**。

**历史脉络与统一视角**：

| 架构 | 提出者 | 时间 | 核心隐喻 | 独特贡献 |
|------|--------|------|----------|----------|
| **六边形架构** | Alistair Cockburn | 2005 | 六边形 + 端口/适配器 | 明确区分"驱动端口"（入站）与"被驱动端口"（出站），强调测试时可替换适配器 |
| **洋葱架构** | Jeffrey Palermo | 2008 | 同心圆，核心在内 | 细化内部层次：领域模型 → 领域服务 → 应用服务 → 基础设施 |
| **整洁架构** | Robert C. Martin | 2012/2017 | 四层同心圆 + 依赖规则 | 形式化"依赖规则"，将 SOLID 原则从类级别提升到架构级别；提出"尖叫架构" |

> **统一规则**：无论名称和图示如何，三者共享同一条铁律——**外层依赖内层，内层不知道外层的存在**。掌握这条规则，就不需要纠结选"六边形"还是"洋葱"——它们是同一哲学的不同表达。

---

##### 8.10.1 依赖规则（The Dependency Rule）

依赖规则是整洁架构的**唯一且绝对的法则**，所有其他原则都是它的推论：

> **源码中的依赖关系必须只向内，指向更高层的策略。内层的任何东西都不能知道外层的任何东西——包括函数、类、变量、以及任何软件实体。**

这条规则的本质是**依赖倒置原则（DIP）在架构级别的应用**。在类级别，DIP 指导我们"面向接口编程"；在架构级别，DIP 指导我们**在边界处放置抽象，让底层细节依赖高层策略**。

**控制流与依赖方向的分离**：

在传统的分层架构中，控制流与依赖方向一致：Controller → Service → DAO → Database。业务逻辑知道数据库的存在，测试必须启动数据库。

在整洁架构中，控制流仍然是从外向内（用户请求 → Controller → Use Case → Entity），但**源码依赖方向被反转**：
- 内层定义接口（端口），如 `OrderRepository`
- 外层实现接口（适配器），如 `JpaOrderRepository`
- 依赖箭头：`JpaOrderRepository → OrderRepository → Use Case → Entity`

```
控制流方向：     外层 ──→ 内层
                  ↓
源码依赖方向：   外层 ←── 内层（通过接口反转）
```

**跨越边界的法则**：

当数据跨越架构边界（从外层进入内层，或从内层输出到外层）时，必须遵守以下规则：
1. **内层不引用外层的类**：`Entity` 不能 `import org.springframework.*`，`UseCase` 不能 `import javax.servlet.*`。
2. **边界处使用简单的数据结构**：跨边界传递的数据应该是语言原生的数据结构（POJO、Map、基本类型），而非框架特定的对象（如 `HttpServletRequest`、`JPA Entity`）。
3. **外层负责格式转换**：Controller 将 HTTP JSON 转为内层的 `Command`/`DTO`；Repository 实现将内层的 `Entity` 转为数据库记录。

```java
// 错误：内层依赖外层框架
public class OrderUseCase {
    // ❌ 内层 import 了 Spring
    @Autowired  
    private JpaOrderRepository repository; 
}

// 正确：内层只依赖自身和同层接口
public class OrderUseCase {
    private final OrderRepository repository;  // 内层定义的端口
    
    public OrderUseCase(OrderRepository repository) {  // 依赖注入，但注入的是接口
        this.repository = repository;
    }
}
```

---

##### 8.10.2 四层同心圆：从策略到机制

整洁架构的四层不是随意划分，而是按照**策略与机制的分野**组织的。越往内，越接近高层业务策略；越往外，越接近底层技术机制。

```
┌──────────────────────────────────────────────────────────────┐
│  框架与驱动（Frameworks & Drivers）                           │
│  Spring Boot / MySQL / Kafka / Redis / gRPC / 第三方 SDK       │
│  ──────────────────────────────────────────────────────────  │
│  接口适配器（Interface Adapters）                              │
│  Controller / Presenter / Gateway / RepositoryImpl / ORM映射  │
│  职责：将外层格式（JSON/SQL/HTTP）转换为内层格式（Entity/DTO）│
│  ──────────────────────────────────────────────────────────  │
│  用例（Use Cases / Application Business Rules）               │
│  CreateOrderUseCase / ApproveLoanUseCase / TransferUseCase    │
│  职责：编排实体，实现特定应用场景的业务流程                      │
│  ──────────────────────────────────────────────────────────  │
│  实体（Entities / Enterprise Business Rules）                 │
│  Order / Account / Loan / Money / 核心业务规则与不变量         │
│  职责：封装最通用、最稳定、可跨应用复用的企业级业务规则           │
└──────────────────────────────────────────────────────────────┘
                    ↑
              源码依赖只向内指
```

**实体层（Entities）——最内层，最稳定**：

实体封装的是**企业级的核心业务规则**。这些规则不依赖于任何应用场景，可以在多个应用中复用。例如：
- `Money`：金额计算规则（不能为负、币种必须一致才能相加）
- `Account`：账户余额不能为负、转账规则
- `Loan`：贷款利率计算、还款计划生成

实体层完全不依赖框架，只包含纯 Java 类和业务逻辑。它的变更频率最低，因为企业的核心规则很少剧烈变化。

**用例层（Use Cases）——应用特定的业务规则**：

用例层封装的是**特定应用场景下的业务逻辑**。它编排实体来完成一个具体的用户目标。用例层知道实体，但不知道数据库、Web 框架或外部服务的存在。

用例层的命名应直接反映业务意图：
- `CreateOrderUseCase.execute(CreateOrderCommand)`
- `ApproveLoanUseCase.execute(LoanId, ApproverId)`
- `ProcessRefundUseCase.execute(OrderId, Reason)`

> **关键原则**：用例层应该是"薄编排、厚规则"。复杂规则应下沉到实体，用例只负责"调用谁、顺序是什么、事务边界在哪里"。

**接口适配器层（Interface Adapters）——数据格式转换器**：

适配器层的唯一职责是**格式转换**。它不包含业务逻辑，只负责：
- **入站适配器**：将外部输入（HTTP 请求、MQ 消息、定时任务触发）转换为用例层可理解的 `Command`/`Query`
- **出站适配器**：将用例层的输出（`Entity`、`Result`）转换为外部需要的格式（JSON Response、数据库记录、消息体）

```java
// 入站适配器：Controller 不做业务判断，只做协议转换
@RestController
public class OrderController {
    private final CreateOrderUseCase createOrderUseCase;
    
    @PostMapping("/orders")
    public ResponseEntity<OrderResponse> create(@RequestBody CreateOrderRequest request) {
        CreateOrderCommand command = new CreateOrderCommand(
            request.getCustomerId(),
            request.getItems().stream().map(/* ... */).toList()
        );
        OrderResult result = createOrderUseCase.execute(command);
        return ResponseEntity.ok(OrderResponse.from(result));
    }
}

// 出站适配器：Repository 实现将实体映射为数据库记录
@Repository
public class JpaOrderRepository implements OrderRepository {
    private final JpaOrderEntityMapper mapper;
    private final SpringDataOrderJpaRepository jpaRepository;
    
    @Override
    public Order save(Order order) {
        OrderJpaEntity entity = mapper.toJpaEntity(order);
        jpaRepository.save(entity);
        return order;
    }
}
```

**框架与驱动层（Frameworks & Drivers）——最外层，最不稳定**：

这一层包含所有的技术细节：Spring Boot、JPA/Hibernate、MySQL、Redis、Kafka、Feign Client、Swagger 等。框架层的代码量应最少——只写"粘合代码"（Glue Code），将框架与适配器层连接起来。

> **可替换性测试**：如果明天要将 Spring Boot 替换为 Quarkus，或将 MySQL 替换为 MongoDB，应该只需要修改框架层和适配器层的实现，而不需要触碰用例层和实体层的一行代码。如果做不到，说明依赖规则被违反了。

---

##### 8.10.3 尖叫架构（Screaming Architecture）

Bob 大叔在提出整洁架构之前，先提出了**尖叫架构**（Screaming Architecture）的概念：

> **当你查看系统的顶层目录结构时，它应该大声"尖叫"出系统的业务意图，而不是它所使用的技术框架。**

如果你看到一个项目的顶层包名是 `com.company.controller`、`com.company.service`、`com.company.dao`，它在尖叫的是："这是一个 Spring 应用"。但你无法知道它是书店、医院系统还是银行核心。

而尖叫架构的包结构应该是：
```
com.company.bookstore/
├── catalog/          ← 商品目录（核心子域）
│   ├── domain/       ← Book, Category, Price 实体
│   ├── usecase/      ← SearchBooks, AddToCatalog
│   └── adapter/      ← CatalogController, BookRepositoryImpl
├── ordering/         ← 订单（核心子域）
│   ├── domain/       ← Order, OrderLine, Money
│   ├── usecase/      ← CreateOrder, CancelOrder
│   └── adapter/      ← OrderController, OrderRepositoryImpl
├── payment/          ← 支付（核心子域）
└── shared/           ← 通用值对象、工具类
```

**尖叫架构的设计原则**：
1. **先按业务领域分包，再按技术层次分包**：`ordering/domain/Order.java` 优于 `domain/ordering/Order.java`
2. **技术细节是隐性的，业务意图是显性的**：框架名称不应出现在顶层包名中
3. **新成员应能快速定位**：一个负责"订单取消"功能的开发者，应该能直接找到 `ordering/usecase/CancelOrderUseCase.java`

---

##### 8.10.4 边界划分：在哪里画线，如何画线

Bob 大叔指出："边界线应该沿着系统的**变更轴**来画。也就是说，位于边界线两侧的组件应该以不同原因、不同速率变化。"

**变更轴分析**：

| 边界两侧 | 变更原因 | 变更速率 | 是否应画边界 |
|----------|----------|----------|-------------|
| GUI vs 业务规则 | GUI 随用户体验迭代，业务规则随政策/法规变化 | GUI 快，业务规则慢 | ✅ 必须画 |
| 业务规则 vs 数据库 | 业务规则随需求变化，数据库随容量/性能优化变化 | 业务规则中等，数据库慢 | ✅ 必须画 |
| 业务规则 vs DI 框架 | 业务规则随市场变化，DI 框架几乎不变 | 业务规则中等，DI 框架极慢 | ⚠️ 视情况画 |
| 订单用例 vs 支付用例 | 订单随促销变化，支付随渠道接入变化 | 两者都快但不同步 | ✅ 子域边界 |

**插件式架构的实现**：

当边界划分完成后，非核心组件应成为核心业务逻辑的"插件"——可以独立替换、独立升级、独立测试。

```java
// 核心业务定义端口（抽象）
public interface PaymentGateway {
    PaymentResult charge(Money amount, Card card);
    PaymentResult refund(TransactionId id, Money amount);
}

// 插件1：支付宝适配器
public class AlipayGateway implements PaymentGateway { /* ... */ }

// 插件2：Stripe 适配器
public class StripeGateway implements PaymentGateway { /* ... */ }

// 插件3：单元测试中的 Mock 适配器
public class MockPaymentGateway implements PaymentGateway {
    @Override
    public PaymentResult charge(Money amount, Card card) {
        return PaymentResult.success(TransactionId.generate());
    }
}

// 用例层通过接口使用支付能力，完全不知道具体实现
public class ProcessPaymentUseCase {
    private final PaymentGateway paymentGateway;  // 依赖抽象
    
    public ProcessPaymentUseCase(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;  // 运行时注入具体插件
    }
}
```

---

##### 8.10.5 SOLID 原则在架构层面的映射

Bob 大叔在书中明确指出：SOLID 原则不仅是类设计的指导，更是**架构设计的基石**。

| 原则 | 类级别含义 | 架构级别含义 | 架构实践 |
|------|-----------|-------------|----------|
| **SRP** | 一个类只对一个行为者负责 | 一个组件（模块/服务）只对一类变更原因负责 | 按业务子域拆分服务；`ordering` 与 `payment` 分开部署 |
| **OCP** | 对扩展开放，对修改封闭 | 通过新增插件扩展系统行为，不修改已有组件 | 新增支付渠道只需新增 `PaymentGateway` 实现，不改用例层 |
| **LSP** | 子类可替换父类而不影响正确性 | 接口的替代实现不影响系统行为 | `JpaOrderRepository` 替换为 `MongoOrderRepository`，`OrderUseCase` 无感知 |
| **ISP** | 不强迫客户端依赖不需要的接口 | 组件只暴露必要的接口，隐藏内部细节 | `OrderRepository` 不暴露 `findOrderLineById()`；查询走专门的 `OrderQueryService` |
| **DIP** | 高层模块依赖抽象，低层模块实现抽象 | 源码依赖永远向内；框架依赖业务规则 | `Controller` 依赖 `UseCase` 接口；`RepositoryImpl` 依赖 `Repository` 接口 |

---

##### 8.10.6 组件构建原则（Component Principles）

《架构整洁之道》第二部分专门讨论了组件（Component）级别的设计原则。组件是软件的**部署单元**（JAR 包、DLL、Maven Module），其设计质量直接影响编译时间、部署灵活性和团队协作效率。

**组件内聚原则（何时将类放在一起）**：

| 原则 | 定义 | 核心思想 | 实践指导 |
|------|------|----------|----------|
| **REP（复用/发布等价原则）** | 复用的粒度就是发布的粒度 | 一起复用的类应该一起发布 | 如果 `Order` 和 `OrderLine` 总是被一起使用，它们应在同一个 JAR 包中 |
| **CCP（共同封闭原则）** | 一起变更的类应该聚合在一起 | 单一职责原则在组件级别的复用 | 如果 "税率计算规则变更" 会导致 `TaxCalculator`、`TaxRule`、`TaxExemption` 同时修改，它们应在同一组件中 |
| **CRP（共同复用原则）** | 不要强迫组件依赖它们不需要的东西 | 接口隔离原则在组件级别的复用 | `order-service` 不应依赖 `reporting-service` 的全部代码，只应依赖其发布的 API 客户端 JAR |

> **张力图（Tension Diagram）**：REP + CCP + CRP 之间存在张力。REP 和 CCP 倾向于让组件更大（更多类在一起），而 CRP 倾向于让组件更小（避免无关依赖）。架构师需要根据当前阶段的质量属性目标（开发效率 vs 部署灵活性）找到平衡点。

**组件耦合原则（组件间的依赖关系）**：

| 原则 | 定义 | 度量 | 实践指导 |
|------|------|------|----------|
| **ADP（无环依赖原则）** | 组件依赖图中不允许存在循环依赖 | 依赖图必须是 DAG（有向无环图） | 使用 Maven/Gradle 的依赖分析工具检测循环；若出现，引入新组件或依赖倒置 |
| **SDP（稳定依赖原则）** | 依赖应指向更稳定的组件 | 不稳定性 `I = 出向依赖 / (出向 + 入向依赖)` | `order-service`（不稳定，`I ≈ 0.8`）可以依赖 `common-domain`（稳定，`I ≈ 0.1`），反之不行 |
| **SAP（稳定抽象原则）** | 稳定的组件应该更抽象 | 抽象度 `A = 抽象类+接口数 / 总类数` | `common-domain` 应有高抽象度（`A ≈ 0.8`），充满接口和抽象类；`order-application` 可以有低抽象度（`A ≈ 0.2`），充满具体实现 |

```
理想状态的组件分布：
  高抽象度(A)
      │
      │    ● 稳定抽象区（SAP 要求在此）
      │   /  如：common-domain, shared-kernel
      │  /
      │ /  × 痛苦区（稳定但具体，难以修改）
      │/       如：充斥着 SQL 的静态工具类
      ├──────────────
      │\  × 无用区（抽象但不稳定，过度设计）
      │ \      如：无人实现的接口
      │  \
      │   ● 不稳定具体区（正常）
      │      如：Controller, Service 实现
      └──────────────────→ 高不稳定性(I)
```

---

##### 8.10.7 三种架构的统一视角与融合实践

在实际工程中，六边形架构、洋葱架构和整洁架构不是"三选一"的关系，而是**同一核心思想的不同侧重**。常见做法是：用洋葱架构组织代码层次，用六边形架构的端口/适配器概念定义边界，用整洁架构的依赖规则作为强制检查。

**Clean Architecture + DDD 的融合映射**：

| Clean Architecture | DDD 四层架构 | 对应内容 | 说明 |
|-------------------|-------------|----------|------|
| **实体（Entities）** | **领域层（Domain）** | 聚合、实体、值对象、领域服务、领域事件 | 最核心、最稳定 |
| **用例（Use Cases）** | **应用层（Application）** | 应用服务、命令处理器、事件处理器、编排逻辑 | 薄层，只负责编排 |
| **接口适配器（Interface Adapters）** | **接口层 + 基础设施层实现** | Controller、RepositoryImpl、MQ Producer、外部客户端 | 格式转换 + 技术实现 |
| **框架与驱动（Frameworks）** | **基础设施层框架** | Spring Boot、JPA、MyBatis、数据库驱动 | 最外层，可替换 |

**Java/Maven 融合实践结构**：

```
clean-order-service/
├── order-domain/                    # 实体层：纯业务规则，零框架依赖
│   └── src/main/java/
│       └── com/example/order/
│           ├── aggregate/
│           │   └── Order.java       # 聚合根：封装业务不变量
│           ├── vo/
│           │   ├── Money.java       # 值对象
│           │   └── OrderId.java     # 标识值对象
│           ├── service/
│           │   └── PricingService.java  # 领域服务
│           ├── event/
│           │   └── OrderPlacedEvent.java
│           └── port/
│               ├── OrderRepository.java       # 出站端口（仓储接口）
│               └── PaymentGateway.java        # 出站端口（支付网关接口）
│
├── order-application/               # 用例层：编排领域对象，无框架依赖
│   └── src/main/java/
│       └── com/example/order/
│           ├── usecase/
│           │   ├── CreateOrderUseCase.java
│           │   └── CancelOrderUseCase.java
│           └── command/
│               └── CreateOrderCommand.java
│
├── order-adapter/                   # 接口适配器层：技术实现
│   ├── in-web/                      # 入站适配器：Web 接口
│   │   └── OrderController.java
│   ├── out-persistence/             # 出站适配器：持久化
│   │   └── JpaOrderRepository.java
│   └── out-payment/                 # 出站适配器：支付网关
│       └── StripePaymentGateway.java
│
└── order-bootstrap/                 # 框架与驱动层：Spring Boot 启动 + 依赖组装
    └── src/main/java/
        └── OrderServiceApplication.java   # @SpringBootApplication
        └── config/
            └── BeanConfiguration.java      # 手动装配端口与适配器
```

> **依赖规则验证**：`order-domain` 的 `pom.xml` 中不应有任何框架依赖（无 Spring、无 JPA、无 Web）。如果 `order-domain` 能编译通过，说明依赖规则未被违反。可使用 ArchUnit 自动化验证：
> ```java
> @ArchTest
> static final ArchRule domain_should_not_depend_on_frameworks =
>     noClasses().that().resideInAPackage("..domain..")
>         .should().dependOnClassesThat().resideInAnyPackage(
>             "org.springframework..", "javax.persistence..", "org.apache.ibatis..");
> ```

---

##### 8.10.8 常见反模式与诊断

| 反模式 | 症状 | 根因 | 修正方案 |
|--------|------|------|----------|
| **内层 import 外层** | `domain/Order.java` 中出现了 `import javax.persistence.*` 或 `@Entity` | 数据库表驱动设计，ORM 注解污染领域模型 | 分离领域对象与持久化对象，用 Mapper 转换；或使用 `orm.xml` 外部配置映射 |
| **用例层做太多事** | `CreateOrderUseCase.execute()` 超过 200 行，包含参数校验、日志、权限检查、MQ 发送 | 将应用层关注点（横切关注点）与用例编排混在一起 | 用 AOP 或拦截器处理日志/权限；用 ApplicationService 封装横切关注点；用例只保留核心编排 |
| **贫血用例/透传控制器** | Controller 直接调用 Repository，UseCase 只是透传，没有任何编排逻辑 | 未识别业务用例，将 CRUD 误认为用例 | 识别真正的业务流程（如"下单"不是 insert，而是"校验库存→计算价格→创建订单→扣减库存→发送事件"） |
| **边界处传递框架对象** | `UseCase.execute(HttpServletRequest request)` 或 `UseCase.execute(Model model)` | 未在适配器层完成格式转换，让内层感知了 HTTP 协议 | 适配器层提取所需数据组装为 `Command`，用例层只接受 `Command` |
| **过度抽象** | 为每个类都定义接口，产生大量 `IOrderService`、`OrderServiceImpl` | 误解了 DIP，将"依赖抽象"等同于"每个类一个接口" | 只对"可能多实现"或"需要测试隔离"的边界创建接口；稳定内部类无需接口 |
| **循环依赖** | `order-application` 依赖 `payment-application`，反之亦然 | 子域边界划分不清，或共享模型未下沉 | 提取共享内核到独立模块；或通过领域事件解耦同步调用 |

> **Bob 大叔的忠告**："一个架构的好坏，不取决于它用了多少模式，而取决于它**推迟了多少决策**。好的架构让你可以在项目后期再决定使用哪个数据库、哪个 Web 框架、哪个消息队列，而不需要重写核心业务逻辑。"

#### 8.11 架构师的修炼

> 《架构师修炼之道》(The Software Architect Elevator) 强调：架构师 50% 的工作是技术，50% 的工作是沟通与推动变革。架构师必须能在组织的不同层级"上下电梯"。

**架构即决策**：
- 技术选型的核心不是"哪个技术最先进"，而是"哪个技术最匹配当前约束"
- 约束包括：团队技能、遗留系统、交付时间、合规要求、预算
- 每个重大决策应记录 ADR，说明问题、可选方案、决策、后果

**架构演进策略**：
- **Strangler Fig（绞杀榕）模式**：逐步用新系统替换旧系统，而非大爆炸重写。通过代理/网关路由流量，逐模块替换
- **演化式设计**：架构不是一次画好，而是在迭代中涌现。先满足当前需求，再识别坏味道并重构
- **技术雷达**：定期评估技术（采纳、试验、评估、暂缓），避免技术债失控

**软技能与沟通**：
- **可视化**：用 C4 模型与不同受众沟通，避免一张图让所有人头晕
- **讲故事**：用场景和质量属性驱动讨论，而非堆砌技术名词
- **推动变革**：架构师是变革代理（Change Agent），需要建立联盟、从小胜利开始、用数据说话

**架构反模式**：

| 反模式 | 症状 | 解毒剂 |
|--------|------|--------|
| **简历驱动开发** | 为了在个人简历上添加技术而引入不必要的复杂度 | 技术选型匹配业务阶段 |
| **金锤** | 手里有把锤子，看什么都像钉子（滥用熟悉技术） | 多了解几种技术，按需选择 |
| **分析瘫痪** | 过度分析，迟迟不做决策，项目停滞 | 设定决策截止日期，用原型验证 |
| **大泥球** | 没有清晰结构，随意依赖，代码腐烂 | 逐步引入边界，绞杀榕重构 |
| **上帝类/上帝服务** | 一个类/服务做所有事 | 按职责拆分，应用 SRP |

#### 8.12 架构设计方法论

**从需求到架构的推导链**：
```
业务目标 → 质量属性需求（性能/可用性/安全） → 架构战术（缓存/冗余/加密）
   ↓
功能需求 → 用例/领域模型 → 组件划分与接口定义
   ↓
约束（技术/组织/法规） → 限制可选方案空间
   ↓
架构决策 → 4+1视图/C4模型表达 → ADR记录
```

**演进式架构的守护**：
- **架构适应度函数（Fitness Functions）**：用自动化测试守护架构规则。如 ArchUnit 验证"领域层不依赖基础设施层"
- **架构防腐**：定期扫描代码库，发现违反分层规则、循环依赖时告警
- **技术债管理**：有意识、可追踪、有偿还计划。将技术债条目化，在迭代中分配时间偿还

**关键Checklist（设计评审时自查）**：
1. 是否识别了前 3 个最重要的质量属性？是否有明确的度量指标？
2. 架构是否支持团队并行开发？边界是否与团队边界对齐（康威定律）？
3. 数据库/框架是否被视为可替换的插件？核心业务是否依赖具体技术？
4. 是否考虑了故障场景？单点故障在哪里？降级方案是什么？
5. 扩展时是否需要修改现有代码？是否符合开闭原则？
6. 是否记录了关键架构决策（ADR）？上下文和后果是否清晰？
> 综合《重构》、《代码整洁之道》、《Effective Java》、《代码大全》、《阿里巴巴Java开发手册》、《编写可读代码的艺术》、《人月神话》、《程序员职业素养》、《程序员修炼之道》九本书的核心思想。
>
> **详细参考**: 需要深入了解某项规范的理论依据、完整规则、反例正例、重构手法时，阅读 `references/coding-standards.md`。
