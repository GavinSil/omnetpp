## nedxml 子系统分析发现 (2026-03-31)

### 架构特点

1. **自动生成代码模式**：`nedelements.h` 和 `msgelements.h` 由 `dtdclassgen.pl` 从 DTD 文件自动生成，确保 AST 结构与 DTD 定义一致。

2. **GLR 解析器**：NED 和 MSG 解析器都使用 Bison GLR（Generalized LR）解析器，NED 有 9 个预期的 shift-reduce 冲突，MSG 无冲突。

3. **验证分层**：
   - DTD 验证：结构合法性
   - 语法验证：语义正确性
   - 交叉验证：跨文件引用（当前未使用）

4. **代码生成属性系统**：MSG 编译器支持 50+ 属性控制代码生成行为（`@owned`, `@byValue`, `@custom` 等）。

### 关键类

- `ASTNode`：AST 节点基类，DOM 风格树结构，泛型+类型化双接口
- `NedParser` / `MsgParser`：解析器前端
- `MsgCompiler`：MSG 编译主类，协调分析器和代码生成器
- `MsgAnalyzer`：提取 ClassInfo/FieldInfo
- `MsgCodeGenerator`：C++ 代码生成（~1800 行最大文件）
- `NedResourceCache`：NED 文件缓存，类型查找

### 技术债

- `nedcrossvalidator.cc` 有 29 个 TODO（代码库最多）
- 该验证器当前标记为"未使用"

### 工具入口

- `opp_nedtool`：NED 文件工具（转换、验证、美化）
- `opp_msgtool`：MSG 文件工具（转换、验证、代码生成）

---

## envir 子系统分析发现 (2026-03-31)

### 架构特点

1. **三层启动流程**：
   - `main.cc` → `evmain.cc` → `startup.cc::setupUserInterface()`
   - 清晰的职责分离：入口点、主函数、初始化逻辑

2. **插件化设计**：
   - 几乎所有核心组件可通过配置替换（RNG、调度器、输出管理器等）
   - 使用 `Register_Class()` 宏注册可实例化类
   - 配置选项如 `*-class` 控制具体实现

3. **配置系统优化**：
   - `SectionBasedConfiguration` 使用"后缀分桶"优化参数查找
   - 按参数名后缀分组，避免线性扫描所有条目
   - 通配符条目单独存放

4. **参数扫描机制**：
   - `Scenario` 类管理迭代变量
   - `ValueIterator` 支持离散值、范围、并行迭代
   - 约束表达式过滤无效组合

### 核心类

- `EnvirBase`：环境基类，管理 RNG、输出管理器、事件日志
- `cEnvir`：环境接口（~900 行），定义仿真内核与环境的交互
- `cRunnableEnvir`：可运行环境接口，UI 层实现此接口
- `SectionBasedConfiguration`：基于 INI 分节的配置实现
- `InifileReader`：低层 INI 文件读取，支持 include
- `EventlogFileManager`：事件日志记录，支持快照和索引
- `cOmnetAppRegistration`：UI 注册机制，按分数选择最佳 UI

### 命令行参数规范

```
ARGSPEC = "h?f:u:l:c:r:n:x:i:p:q:e:avwsSmX:"
```

关键选项：
- `-f`：指定 INI 文件
- `-u`：选择用户界面
- `-c`：选择配置
- `-r`：运行过滤器
- `-n`：NED 路径
- `-l`：加载库
- `-q`：查询运行信息

### 输出管理器层级

```
cIOutputVectorManager ← OmnetppOutputVectorManager / SqliteOutputVectorManager
cIOutputScalarManager ← OmnetppOutputScalarManager / SqliteOutputScalarManager
cIEventlogManager ← EventlogFileManager
cISnapshotManager ← FileSnapshotManager
```

### 预定义配置变量

- `${configname}`、`${runnumber}`、`${repetition}`
- `${datetime}`、`${processid}`、`${runid}`
- `${iterationvars}`、`${resultdir}`

### 扩展点

| 配置选项 | 基类 |
|---------|------|
| `configuration-class` | `cConfigurationEx` |
| `scheduler-class` | `cScheduler` |
| `rng-class` | `cRNG` |
| `outputvectormanager-class` | `cIOutputVectorManager` |
| `outputscalarmanager-class` | `cIOutputScalarManager` |
| `eventlogmanager-class` | `cIEventlogManager` |

### UI 注册机制

```cpp
Register_OmnetApp("Cmdenv", Cmdenv, 1, "Command-line user interface");
Register_OmnetApp("Qtenv", Qtenv, 2, "Qt-based graphical user interface");
```

分数高的 UI 优先被选择（Qtenv 默认优先于 Cmdenv）。

### 输出文件命名模板

```
${resultdir}/${configname}-${iterationvarsf}#${repetition}.vec
${resultdir}/${configname}-${iterationvarsf}#${repetition}.sca
```

---

## sim/ 子系统分析发现 (2026-03-31)

### 类层次结构
- 根类 `cObject` 无数据成员，纯虚基类
- `cNamedObject` 添加名称字符串
- `cOwnedObject` 添加所有权指针，实现自动内存管理
- `cSoftOwner` 是软所有者，允许对象被其他所有者取走

### 事件系统设计
- `cEvent` 是事件抽象基类，`cMessage` 继承自它
- `cPacket` 继承 `cMessage`，添加长度、错误标志、封装能力
- FES 使用 `cEventHeap`（二叉堆）实现
- 事件按 `(arrivalTime, priority, insertOrder)` 排序

### 模块系统
- `cModule` 是抽象基类，`cSimpleModule` 是具体实现
- 支持 `handleMessage()` 和 `activity()` 两种编程风格
- `activity()` 使用协程 (`cCoroutine`)，不推荐用于大型仿真

### 所有权机制
- 防止常见的内存管理错误
- 硬所有者（cQueue）不允许对象被取走
- 软所有者（cModule）允许对象转移

### 信号系统
- 使用 `emit()`/`subscribe()` 模式
- 信号向上传播到祖先模块
- `@statistic` NED 属性自动创建记录器链

### 关键模式
- 模块注册：`Define_Module(ClassName)`
- 类注册：`Register_Class(ClassName)`
- 多阶段初始化：`numInitStages()` + `initialize(stage)`

---

## Common 子系统发现（2026-03-31）

### 表达式引擎架构

1. **四阶段处理流水线**
   - 词法分析（Flex）→ 语法分析（Bison）→ 语义翻译 → 求值执行
   - AST 作为中间表示，支持多种翻译器

2. **动态解析机制**
   - `AstTranslator` 接口支持自定义语义翻译
   - `DynamicResolver` 支持运行时变量/函数解析
   - 允许表达式上下文绑定延迟到求值时

3. **ExprValue 类型系统**
   - 支持动态类型（BOOL/INT/DOUBLE/STRING/POINTER）
   - 内置物理单位支持（自动转换）
   - `any_ptr` 实现类型安全的 cObject 指针

### 模式匹配设计

1. **PatternMatcher 特点**
   - Glob 风格语法（`*`, `?`, `**`, `{a-z}`, `{n..m}`）
   - dottedpath 模式支持层级路径匹配（`**.mac[*].retries`）
   - `covers()` 方法判断模式覆盖关系

2. **MatchExpression 扩展**
   - 支持 AND/OR/NOT 逻辑组合
   - 支持多字段匹配（`className =~ TCP*`）
   - `Matchable` 接口解耦匹配逻辑

### 文件 I/O 性能优化

1. **FileReader 大文件策略**
   - 256KB 默认缓冲区
   - 支持正向/反向遍历
   - 文件变化检测（追加/覆盖）
   - 惰性打开/关闭

2. **输出写入器统一模式**
   - `open()` / `close()`
   - `beginRecordingForRun()` / `endRecordingForRun()`
   - `record*()` 系列方法
   - 缓冲写入 + `flush()`

### 单位转换系统

1. **UnitConversion 特性**
   - 内置物理单位表（时间、功率、频率等）
   - 支持线性/对数单位转换（W ↔ dBW）
   - `getBestUnit()` 智能选择人可读单位

2. **表达式中的单位**
   - 支持 `5s 230ms` 复合单位语法
   - 求值时自动单位转换
   - 类型检查确保单位兼容

### 命名约定

- 全局工具函数：`opp_` 前缀（`opp_streq`, `opp_strdup`）
- 类方法：camelCase
- 节点类：`XxxNode` 后缀
- 命名空间：`omnetpp::common`

---

## 公共 API 分析发现（2026-03-31）

### API 结构

1. **头文件数量**：124 个公共头文件，113 个使用 SIM_API 导出宏
2. **导出点**：379 处 SIM_API 使用

### 关键设计模式

1. **cObject 层次结构**
   - 所有仿真对象继承自 cObject
   - 提供统一的命名、所有权、反射机制
   - cOwnedObject 添加所有权追踪

2. **组件模型**
   - cComponent → cModule/cChannel 分支
   - cModule → cSimpleModule（用户实现）
   - cChannel → cIdealChannel/cDatarateChannel/cDelayChannel

3. **消息模型**
   - cEvent → cMessage → cPacket 层次
   - cPacket 添加长度、错误标志、封装能力

### 命名约定确认

- 类：c-前缀（cModule, cMessage）
- 方法：camelCase（handleMessage, scheduleAt）
- C 函数：opp_ 前缀（opp_isempty, opp_streq）
- 宏：大写下划线或 Register_ 前缀

### API 分组结构（index.h 定义）

```
Fundamentals     - 基础类
ModelComponents  - 模型组件
SimProgr         - 仿真编程
SimCore          - 仿真核心
SimTime          - 仿真时间
RandomNumbers    - 随机数
Statistics       - 统计
ResultFiltersRecorders - 结果过滤/记录
Expressions      - 表达式
Canvas           - 2D 画布
OSG              - 3D 画布
FSM              - 有限状态机
Signals          - 信号
Logging          - 日志
Misc             - 杂项
WatchMacros      - WATCH 宏
RegMacros        - 注册宏
StringFunctions  - 字符串函数
UtilityFunctions - 工具函数
EnvirAndExtensions - 环境与扩展
Internals        - 内部类（排除）
ParsimBrief      - 并行仿真
```

### 重要发现

1. **SendOptions 结构**（csimplemodule.h）：
   - 支持延迟发送、传播延迟、传输持续时间
   - 支持传输更新（abort/preempt 传输）

2. **simtime_t 类型**：
   - 64 位定点表示
   - 全局共享指数
   - 类型别名：`typedef SimTime simtime_t`

3. **随机数框架**：
   - cRNG 接口 + 多种实现
   - cRandom 抽象基类 + 20+ 分布类
   - 同时提供函数式 API（distrib.h）

### 排除项

- `@ingroup Internals` 标记的类不在公共 API 范围
- 内部实现细节不记录
