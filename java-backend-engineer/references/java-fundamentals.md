#### 语言基础与面向对象

> 综合《Java编程思想》(Bruce Eckel, Thinking in Java) 与《On Java 8》的核心思想。前者是理解Java面向对象本质的启蒙经典；后者是前者的现代重构，深度整合了Java 8+的函数式编程范式。本章节提炼两本书中直接指导工程实践的核心理念。

##### 对象导论

**面向对象设计本质**（《TIJ》核心哲学）：
- **万物皆对象**：将问题空间中的元素表示为"对象"，程序是对象的集合，对象间通过发送消息（方法调用）通信
- **程序是对象的集合**：对象持有状态（字段）、具有行为（方法）、能接收和发送消息
- **每个对象都有类型**：类是对象的抽象描述，对象是其具体实例
- **特定类型的所有对象都可接收相同消息**：子类型可替代父类型，这是多态的基础

**抽象过程**：所有编程语言都提供抽象机制。汇编是对机器层的抽象；面向过程是对汇编的抽象；面向对象则提供了对问题空间更直接的建模能力——程序员在解空间中表示问题空间的元素。

**对象创建与生命周期**：
- Java 中对象在堆上创建，通过 `new` 分配。垃圾回收器自动管理内存，消除了C++中内存泄漏的大部分根源
- 但 **GC 不等于不会内存泄漏**：持有无用对象引用（如未清理的集合、未注销的监听器、ThreadLocal 未 remove）仍会导致泄漏

##### 一切都是对象

**引用与对象**：
```java
String s;      // 引用变量，未初始化时为 null
s = "abc";     // 引用指向堆中的 String 对象
```
- Java 中一切都被视为对象，但操纵的是对象的**引用**（reference），而非对象本身
- 引用可独立存在，不必须指向对象；但使用未初始化的引用会抛出 `NullPointerException`

**基本类型与包装类**：
| 基本类型 | 包装类 | 大小 | 默认值 |
|----------|--------|------|--------|
| `boolean` | `Boolean` | — | `false` |
| `char` | `Character` | 16-bit | `\u0000` |
| `byte` | `Byte` | 8-bit | 0 |
| `short` | `Short` | 16-bit | 0 |
| `int` | `Integer` | 32-bit | 0 |
| `long` | `Long` | 64-bit | 0L |
| `float` | `Float` | 32-bit | 0.0f |
| `double` | `Double` | 64-bit | 0.0d |

- 基本类型直接存储值，效率更高；包装类是对象，拥有方法和常量（如 `Integer.MAX_VALUE`）
- **自动装箱/拆箱**（JDK5）：`Integer a = 100; int b = a;`。但注意：`==` 比较包装类时，-128~127 缓存区间外比较的是引用地址，应使用 `.equals()`

**高精度数字**：`BigInteger`（任意精度整数）、`BigDecimal`（任意精度定点数）。金融计算必须用 `BigDecimal`，禁止用 `float/double`。

**数组**：Java 数组是**对象**（有 `length` 字段），创建时大小固定，元素自动初始化为默认值。多维数组是"数组的数组"。

##### 操作符、控制流与字符串

**操作符要点**：
- `==` 与 `!=` 比较的是**引用地址**（基本类型比较值）；对象内容比较用 `.equals()`
- 整数除法截断小数：`int x = 7 / 4; // x = 1`
- 逻辑操作符 `&&`（短路与）、`||`（短路或）：左侧能确定结果时右侧不执行。位操作符 `&`、`|` 无短路特性
- 三元操作符 `boolean ? a : b` 中，a 和 b 会被类型提升为更宽类型

**字符串**（`String`）：
- **不可变性**：`String` 对象一旦创建不可修改。所有修改操作（如 `concat`、`replace`）都返回新对象
- 字符串常量池：字面量 `"abc"` 存放在常量池，相同字面量共享同一对象。`new String("abc")` 则强制在堆上新建对象
- `StringBuilder`（非线程安全）/ `StringBuffer`（线程安全，已少用）：循环拼接字符串时必须使用，避免大量临时对象
- `String.join()`、`String.format()`、`MessageFormat`：格式化首选，避免 `+` 拼接复杂字符串

**控制流**：
- `switch`（传统）：支持 `byte/short/char/int` 及其包装类、`enum`、`String`（JDK7）
- `switch` 表达式（JDK12+，标准于JDK14）：可返回值，支持箭头语法 `case L ->`，多标签 `case A, B ->`，无需 `break`
- `for-each`：适用于数组和 `Iterable`，底层通过 `Iterator`。遍历时**禁止**增删元素（抛 `ConcurrentModificationException`）

##### 初始化和清理

**构造器**：
- 与类同名、无返回值。没有显式构造器时，编译器提供默认无参构造器；一旦定义了有参构造器，默认构造器消失
- 构造器是编译期绑定，**不是多态的**：在构造器中调用可重写方法是危险的，子类可能尚未初始化

**this 关键字**：
- `this` 指当前对象引用，用于区分字段与局部变量、调用同类其他构造器（`this(...)` 必须位于构造器第一行）
- 链式调用：`return this;`

**super 关键字**：
- `super.method()` 调用父类实现；`super()` 调用父类构造器（隐式或显式，必须位于子类构造器第一行）
- 构造器调用链：子类构造器 → 父类构造器 → 更上层父类构造器 → `Object` 构造器

**初始化顺序**（极其重要）：
```
父类静态变量/静态代码块（按出现顺序）
  → 子类静态变量/静态代码块
  → 父类实例变量/实例代码块
  → 父类构造器
  → 子类实例变量/实例代码块
  → 子类构造器
```

**清理与垃圾回收**：
- Java 没有 C++ 的析构函数。`finalize()` 方法在 JDK9 标记为废弃，JDK18 正式弃用，**绝不应依赖它**进行资源释放
- 资源释放必须使用 **try-with-resources**（`AutoCloseable`）或 `finally` 块

##### 访问控制

**包（package）**：将类组织在单一命名空间下，解决命名冲突。包名对应目录结构，应与组织反向域名匹配（如 `com.example.project`）。

**访问权限**（从宽到严）：

| 修饰符 | 同类 | 同包 | 子类 | 任何地方 |
|--------|------|------|------|----------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| 包访问（默认）| ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

- **封装原则**：字段优先 `private`，按需通过 getter/setter 暴露。这不仅关乎安全，更关乎**解耦**——可在方法中添加校验、日志、通知，而不影响客户端
- `protected` 还包含包访问权限，不要误以为只有子类能访问

##### 复用类：组合与继承

**组合（Has-A）**：新类中包含已有类的对象。更灵活、更松耦合，是复用的首选方式。
**继承（Is-A）**：新类是已有类的特殊类型。强耦合，子类依赖父类实现细节。

**《Effective Java》+《TIJ》共识**：
> **优先使用组合，而非继承**。继承是白箱复用（需了解父类内部），组合是黑箱复用（只需了解接口）。仅在真正的子类型关系（LSP 成立）时使用继承。

**继承中的要点**：
- `@Override` 注解：重写方法时必须加，编译器会检查签名匹配，防止笔误
- 向上转型（Upcasting）：子类引用赋给父类引用，安全、自动。`Parent p = new Child();`
- 向下转型（Downcasting）：父类引用转回子类，需显式转换，可能抛 `ClassCastException`。先用 `instanceof` 检查

**final 关键字**：
- `final` 类：不可被继承（如 `String`、`System`）
- `final` 方法：不可被重写
- `final` 变量：基本类型值不可变，引用类型引用不可变（但对象内部状态可变）

##### 多态

> 多态（Polymorphism）是面向对象编程的核心，也是《TIJ》全书最深入探讨的主题。

**方法调用绑定**：
- **前期绑定（静态绑定）**：编译期确定调用哪个方法。`private`、`static`、`final` 方法都是前期绑定
- **后期绑定（动态绑定）**：运行期根据对象实际类型确定调用方法。普通实例方法默认后期绑定

**多态的本质**：发送消息给对象，让对象根据自身的类型决定如何响应。

**多态的陷阱**：
1. **构造器中的多态**：在构造器中调用重写方法，子类构造器尚未执行，子类字段可能处于默认状态。避免在构造器中调用非 `final`/`private` 方法
2. **协变返回类型**（JDK5）：重写方法可返回原返回类型的子类型
3. **静态方法不参与多态**：静态方法根据引用类型调用，不是对象实际类型

##### 接口与抽象类

**抽象类**：
- 可包含抽象方法和具体方法、字段。是"不完全的类"，为子类提供部分实现
- 单继承限制：一个类只能继承一个抽象类

**接口**（Java 8 之前）：
- 完全抽象，所有方法隐式 `public abstract`，所有字段隐式 `public static final`
- 多重实现：一个类可实现多个接口，弥补了Java单继承的限制

**接口演进**（Java 8+）：
- **默认方法**（`default`）：在接口中提供方法实现，实现类可不重写。用于接口的向后兼容扩展
- **静态方法**（`static`）：接口级别的工具方法
- **私有方法**（Java 9+）：接口内部的辅助方法，供默认方法或静态方法调用

**接口 vs 抽象类选择**：
- 需要多重继承能力 → 接口
- 只是想定义契约（what），不关心实现（how）→ 接口
- 需要共享代码实现，且是"is-a"关系 → 抽象类
- Java 8+ 后，接口能力大增，优先倾向接口（《Effective Java》第20条）

##### 内部类

**四种内部类**：
1. **成员内部类**：定义在类内部，如同字段。可访问外部类所有成员（包括 `private`）
2. **局部内部类**：定义在方法内部，作用域仅限该方法
3. **匿名内部类**：没有名字，在创建时定义。常用于事件监听器、Runnable。JDK8 后大量被 Lambda 替代
4. **静态嵌套类**：用 `static` 修饰，不持有外部类引用，是最接近普通顶层类的形式。优先使用静态嵌套类，避免隐式持有外部引用导致内存泄漏

**内部类的本质**：编译器生成独立的 `.class` 文件（如 `Outer$Inner.class`），通过合成方法访问外部类私有成员。成员内部类持有指向外部类的隐式引用 `Outer.this`。

##### 异常体系

**Java 异常层次**：
```
Throwable
├── Error（严重系统错误，不应捕获）
│     └── OutOfMemoryError, StackOverflowError...
└── Exception
      ├── RuntimeException（非受检异常，编程错误）
      │     └── NullPointerException, IllegalArgumentException...
      └── 其他受检异常（外部不可控条件）
            └── IOException, SQLException...
```

**受检异常（Checked）vs 非受检异常（Unchecked）**：
- **受检异常**：调用者必须处理（`try-catch` 或 `throws`）。代表外部不可控问题（IO失败、网络中断）
- **非受检异常**：不强制处理。代表编程错误（空指针、非法参数）

**异常处理原则**（《Effective Java》+《Clean Code》）：
1. **只在异常情况下使用异常**，不要用异常做正常控制流
2. **对可恢复的情况使用受检异常**，对编程错误使用运行时异常
3. **优先使用标准异常**：`IllegalArgumentException`、`IllegalStateException`、`NullPointerException`
4. **不要忽略异常**：空的 catch 块是代码的毒瘤。至少记录日志
5. **异常转换**：捕获底层异常，抛出自定义业务异常，保留原始异常作为 cause
6. **finally 与 try-with-resources**：资源释放用 `try (Resource r = ...) { ... }`，简洁且安全

**常见异常**：
| 异常 | 场景 | 避免方法 |
|------|------|----------|
| `NullPointerException` | 调用 null 引用方法/字段 | 使用 `Optional`、防御性检查、`Objects.requireNonNull()` |
| `ClassCastException` | 错误的向下转型 | 先用 `instanceof` 检查 |
| `ConcurrentModificationException` | 遍历中修改集合 | 使用 `Iterator.remove()` 或并发容器 |
| `IllegalArgumentException` | 参数不合法 | 在方法入口校验参数 |
| `IndexOutOfBoundsException` | 数组/集合越界 | 检查索引范围 |

##### 泛型

**泛型的本质**：参数化类型，实现"类型安全的容器"。编译期类型检查 + 自动类型转换，避免运行时 `ClassCastException`。

**核心概念**：
- **类型参数**：`class Box<T> { T item; }`。T 是类型形参，实例化时替换为具体类型
- **泛型擦除**：Java 泛型通过擦除实现，编译后所有类型参数替换为边界（默认 `Object`）。运行时没有泛型类型信息（`List<String>` 和 `List<Integer>` 运行时是同一类型）
- **边界**：`extends` 上界（`T extends Number`）、`super` 下界（`? super Integer`）

**通配符**：
- `?`：无界通配符，代表任意类型。`List<?>` 是任何 `List` 的父类型，但除了 `null` 不能添加元素
- `? extends T`：上界通配符，可安全**读取** T 及子类。Producer（生产者）用 `extends`
- `? super T`：下界通配符，可安全**写入** T 及子类。Consumer（消费者）用 `super`
- **PECS 原则**：`Producer Extends, Consumer Super`

**泛型限制**（擦除导致的）：
- 不能创建泛型数组：`new T[10]` 非法
- 不能对泛型类型使用 `instanceof`：`if (x instanceof T)` 非法
- 不能创建泛型类型的对象：`new T()` 非法
- 静态字段不能引用类型参数

**泛型方法**：`<T> void method(T t)`。类型参数列表位于返回类型之前。泛型方法可在普通类中定义。

##### Lambda 表达式与函数式接口

> 《On Java 8》将 Lambda 视为Java从"面向对象"走向"函数式编程"的转折点。

**Lambda 语法**：`(参数) -> { 主体 }`。单参数可省略括号；单表达式可省略大括号和 `return`。

**函数式接口**：只有一个抽象方法的接口。可用 `@FunctionalInterface` 标注，编译器会检查。

| 核心函数式接口 | 方法签名 | 用途 |
|---------------|----------|------|
| `Predicate<T>` | `boolean test(T t)` | 断言/过滤 |
| `Consumer<T>` | `void accept(T t)` | 消费/接收 |
| `Function<T, R>` | `R apply(T t)` | 转换/映射 |
| `Supplier<T>` | `T get()` | 供给/生成 |
| `UnaryOperator<T>` | `T apply(T t)` | 一元操作 |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | 二元操作 |

**方法引用**：`ClassName::methodName`，是 Lambda 的简写形式。分四类：
- 静态方法：`String::valueOf`
- 实例方法（特定对象）：`System.out::println`
- 实例方法（任意对象）：`String::length`
- 构造器：`ArrayList::new`

**闭包与变量捕获**：Lambda 可捕获外部局部变量，但变量必须是**事实最终变量**（effectively final）。修改外部变量会编译错误。

##### Stream API

> 《On Java 8》的核心章节。Stream 不是数据结构，而是对数据源的"高级迭代器"，支持声明式数据处理。

**Stream 特点**：
- **不存储数据**：从集合、数组、IO 等源获取，通过管道操作处理
- **函数式**：操作不改变源数据，产生新结果
- **惰性求值**：中间操作（`filter`、`map`）不会立即执行，直到遇到终端操作（`collect`、`forEach`）
- **可消费性**：一个 Stream 只能消费一次，再次使用会抛 `IllegalStateException`

**操作分类**：

| 中间操作（惰性） | 终端操作（立即） |
|----------------|----------------|
| `filter(Predicate)` | `forEach(Consumer)` |
| `map(Function)` | `collect(Collector)` |
| `flatMap(Function)` | `reduce(BinaryOperator)` |
| `distinct()` | `count()` / `min()` / `max()` |
| `sorted()` | `anyMatch()` / `allMatch()` / `noneMatch()` |
| `peek(Consumer)` | `findFirst()` / `findAny()` |
| `limit(long)` / `skip(long)` | `toArray()` / `iterator()` |

**常用收集器**：
- `Collectors.toList()` / `toSet()` / `toMap(keyMapper, valueMapper)`
- `Collectors.groupingBy(Function)`：分组
- `Collectors.partitioningBy(Predicate)`：分区（true/false 两组）
- `Collectors.joining(CharSequence)`：字符串连接
- `Collectors.summarizingInt(ToIntFunction)`：统计（count/sum/min/max/avg）

**并行流**：`stream.parallel()` 或 `list.parallelStream()`。使用 Fork/Join 框架在多核上并行处理。
- **慎用并行流**：数据量小、源分割成本高、涉及 IO、需要严格顺序保证时，并行反而更慢
- 避免在并行流中修改共享可变状态

**Optional**（JDK8）：
- 表示可能为空的容器对象，避免 `null` 扩散
- `Optional.ofNullable(value)` / `map()` / `filter()` / `orElse(default)` / `orElseThrow()` / `ifPresent()`
- **最佳实践**：作为返回值类型使用；**不要**用作字段或方法参数（增加包装开销，且语义不清晰）

##### Java 新特性速查（版本条件化使用）

> **重要原则**：以下新特性只有在判断当前项目 JDK 版本符合时才建议使用。向后兼容的项目应谨慎引入。

**Java 9-17 重要特性**：

| 版本 | 特性 | 说明 | 使用条件 |
|------|------|------|----------|
| 9 | 模块系统（Jigsaw） | `module-info.java` 显式声明依赖和导出 | 大型项目模块化重构时 |
| 10 | `var` 局部变量类型推断 | `var list = new ArrayList<String>();` | JDK 10+，减少冗余类型声明 |
| 9 | 不可变集合工厂 | `List.of()`、`Set.of()`、`Map.of()` | JDK 9+，快速创建小型不可变集合 |
| 11 | 新 HTTP Client | `HttpRequest`/`HttpResponse`，支持异步和 HTTP/2 | JDK 11+，替代 `HttpURLConnection` |
| 11 | `String` 新方法 | `isBlank()`、`lines()`、`strip()`、`repeat()` | JDK 11+ |
| 14 | `switch` 表达式 | 返回值、箭头语法、多标签 `case` | JDK 14+（标准特性）|
| 16 | `record` | `record Point(int x, int y) {}` 自动生成构造器、getter、equals、hashCode、toString | JDK 16+（标准特性）|
| 16 | `instanceof` 模式匹配 | `if (obj instanceof String s) { s.length(); }` | JDK 16+（标准特性）|

**Java 18 特性**（仅在项目使用 JDK 18+ 时建议）：
- **UTF-8 by Default**：默认字符集从平台相关改为 UTF-8，跨平台行为一致。影响 `FileReader`、`FileWriter` 等未指定字符集的 API
- **Simple Web Server**：`jwebserver` 命令行工具，提供极简静态文件服务器，便于原型开发和测试
- **弃用 Finalization**：`finalize()` 方法正式标记为废弃（for removal）。**工程影响**：彻底移除对 `finalize()` 的任何使用，全部替换为 `try-with-resources` 或 `Cleaner`（Java 9+）
- **@snippet 标签**：Javadoc 中 `@snippet` 替代 `<code>`，支持语法高亮和外部代码引用

**Java 21 特性（LTS）**（仅在项目使用 JDK 21+ 时建议）：
- **虚拟线程（Virtual Threads）**：`Thread.startVirtualThread(() -> ...)` 或 `Executors.newVirtualThreadPerTaskExecutor()`
  - 极轻量（可创建数百万个），由 JVM 调度到平台线程上执行
  - 使用 `synchronized` 或 `BlockingQueue` 时，虚拟线程会被**固定**（pinned）到平台线程，降低可伸缩性。优先使用 `ReentrantLock` 替代 `synchronized`
  - 适合高并发 IO 密集型场景（Web 服务、数据库访问），不适合纯计算密集型
- **Record Patterns**（标准特性）：解构 `record` 类型的模式匹配
  ```java
  record Point(int x, int y) {}
  if (obj instanceof Point(int x, int y)) {
      System.out.println(x + y);
  }
  ```
- **Pattern Matching for switch**（标准特性）：`switch` 支持类型模式、null 处理和守卫条件（guarded patterns）
  ```java
  switch (obj) {
      case Integer i when i > 0 -> "positive";
      case Integer i -> "non-positive";
      case String s -> "string: " + s;
      case null -> "null";
      default -> "unknown";
  }
  ```
- **Sequenced Collections**：`SequencedCollection`、`SequencedSet`、`SequencedMap` 接口，统一提供有序集合的头尾操作方法（`addFirst`、`removeLast`、`reversed()` 等）
- **String Templates**（Preview，JDK21）：`STR."Hello \{name}"`，类型安全的字符串插值，替代 `String.format()` 和 `+` 拼接
- **Unnamed Patterns and Variables**（Preview，JDK21）：用 `_` 表示不关心的模式变量，如 `case Point(_, int y)`、`var _ = sideEffect();`

**版本判断与迁移建议**：
- 新项目优先选择 LTS 版本（JDK 17 或 JDK 21）
- 使用 `Runtime.version()` 判断运行期 JDK 版本
- 升级策略：先升级到最近的 LTS，再评估新特性收益。虚拟线程是 JDK 21 引入生产环境的最大亮点

---

