# OMNeT++ 6.4.0 功能总结

> 来源：项目 AGENTS.md、architecture.md、源码目录结构、公共 API 头文件等

---

## 一、仿真内核 (Simulation Kernel)

| 功能 | 说明 | 关键类/文件 |
|------|------|------------|
| 离散事件仿真引擎 | 基于事件调度的 DES 核心 | `cSimulation`, `cFutureEventSet`, `cEventHeap` |
| 组件模型 | 模块、门、连接的层次化组合 | `cModule`, `cSimpleModule`, `cGate`, `cChannel` |
| 消息与分组 | 事件传递与数据包建模 | `cMessage`, `cPacket`, `cPacketQueue` |
| 事件调度 | 定时、取消、相对调度 | `scheduleAt()`, `cancelEvent()`, `scheduleAfter()` |
| 仿真时间 | 固定点高精度时间表示 | `simtime_t`, `simtime.h` |
| 参数系统 | 类型安全的模块参数，支持隐式转换 | `cPar`, `cParImpl` 及其子类 |
| 随机数生成 | 多种 RNG 实现 | `cRNG`, `cMersenneTwister`, `cLCG32` |
| 统计收集 | 标量、向量、直方图记录 | `cStdDev`, `cHistogram`, `cOutVector`, `cResultRecorder`, `cResultFilter` |
| 直方图策略 | 多种密度估计方法 | `cHistogram`, `cPSquare`, `cKSplit`, `cPrecollectedDensityEst`, `cAbstractHistogram` |
| 信号机制 | 基于发布-订阅的统计采集 | `cResultListener`, `@statistic` NED 属性 |
| 配置系统 | INI 文件配置，节继承与参数化 | `cConfiguration`, `cConfigReader`, `cConfigOption` |
| 拓扑发现 | 网络拓扑遍历与发现 | `cTopology` |
| 有限状态机 | FSM 建模支持 | `cFSM` |
| 指纹校验 | 仿真结果指纹验证 | `cFingerprint` |
| 模型变更通知 | 运行时模型变更回调 | `cModelChange` |
| 生命周期监听 | 仿真阶段生命周期 | `cLifecycleListener` |
| 共享库加载 | 动态加载仿真模型库 | `cModuleType`, `cComponentType` |
| 对象所有权 | 自动内存管理 | `cOwnedObject`, `cSoftOwner` |
| 观察器 | 运行时变量监控 | `cWatch`, `cStlWatch` |
| 协程 | 基于协程的模块行为 | `cCoroutine` |
| 显示字符串 | 模块可视化描述 | `cDisplayString` |
| Canvas/图形 | 2D 图形绘制 API | `cCanvas`, `cOsgCanvas` |
| 属性系统 | NED 属性注解 | `cProperty`, `cProperties` |
| 消息打印 | 自定义消息格式化 | `cMessagePrinter` |
| XML 元素 | XML 配置支持 | `cXMLElement` |
| 单位转换 | 物理单位自动换算 | `unitconversion.h` |
| 哈希工具 | 通用哈希 | `cHasher` |

## 二、NED 语言 (Network Description Language)

| 功能 | 说明 |
|------|------|
| 简单模块定义 (`simple`) | 定义原子行为模块 |
| 复合模块定义 (`module`) | 层次化组合子模块 |
| 网络定义 (`network`) | 顶层仿真网络 |
| 信道定义 (`channel`) | 连接建模 (`cDelayChannel`, `cDatarateChannel`) |
| 参数声明 | `int`, `double`, `string`, `bool`, `xml` 类型 |
| 门声明 (`gates`) | 输入门、输出门、双开门 |
| 连接声明 (`connections`) | 门间连接，支持信道参数 |
| 子模块实例化 (`submodules`) | 参数化子模块创建 |
| 条件连接 (`conditions`) | 基于参数的条件连接 |
| 属性注解 (`@`) | `@statistic`, `@display`, `@class`, `@units` 等 |
| 模块继承 (`extends`) | NED 类型继承与扩展 |
| 内联类型 | 在子模块中直接定义类型 |
| 默认值与单位 | 参数默认值，物理量单位绑定 |
| 包声明 (`package`) | NED 文件的包组织 |
| 导入 (`import`) | 跨包类型导入 |
| 表达式 | 参数支持数学与 NED 内建函数 |

## 三、MSG 语言 (Message Definition Language)

| 功能 | 说明 |
|------|------|
| 消息定义 | 定义 `cMessage`/`cPacket` 子类 |
| 字段声明 | 基本类型与自定义类型字段 |
| 数组字段 | 固定大小数组字段 |
| 继承 | 消息类继承 |
| 属性注解 | `@customize`, `@omitGetVerb` 等 |
| 代码生成 | 自动生成 C++ 头文件与实现 |
| 嵌套类 | 消息内嵌类定义 |

## 四、并行/分布式仿真 (Parallel Simulation)

| 功能 | 说明 | 关键类 |
|------|------|--------|
| MPI 通信 | 基于 MPI 的并行通信 | `cMPIComm`, `cMPICommBuffer` |
| 命名管道通信 | 基于命名管道的进程间通信 | `cNamedPipeComm` |
| 文件通信 | 基于文件的进程间通信 | `cFileComm` |
| 空消息协议 (Null Message Protocol) | 经典保守同步协议 | `cNullMessageProt` |
| 理想仿真协议 | 无延迟的理想同步 | `cIdealSimulationProt` |
| 无同步 | 无同步协调（调试用途） | `cNoSynchronization` |
| 链路延迟前看 | 基于链路延迟的前看优化 | `cLinkDelayLookahead`, `cAdvLinkDelayLookahead` |
| ISP 事件记录 | 并行仿真事件日志 | `cISPEventLogger` |
| 代理门 | 跨分区门映射 | `cProxyGate` |
| 占位模块 | 远端模块占位 | `cPlaceholderMod` |
| 分区管理 | 仿真分区配置 | `cParsimPartition`, `cParsimSynchr` |

## 五、仿真运行环境 (Envir)

| 功能 | 说明 |
|------|------|
| 命令行界面 (Cmdenv) | 纯文本终端运行，批处理仿真 |
| Qt 图形界面 (Qtenv) | 交互式图形界面，动画、检查器 |
| INI 配置系统 | 节继承、通配符、参数覆盖 |
| 结果文件管理 | 标量 (.sca) 和向量 (.vec) 文件输出 |
| SQLite 结果后端 | 结果存储到 SQLite 数据库 |
| 事件日志 | 事件轨迹记录与回放 |
| 配置选项注册 | `Register_PerObjectConfigOption` |
| 仿真启动序列 | 模块初始化、网络构建、事件循环 |
| 环境接口 | `cEnvir` 抽象接口，可嵌入 |

## 六、结果分析 (Scave)

| 功能 | 说明 | 关键文件 |
|------|------|----------|
| 标量结果文件 | `.sca` 文件读写 | `omnetppscalarfilewriter.h` |
| 向量结果文件 | `.vec` 文件读写与索引 | `vectorfileindex*.cc` |
| 结果文件管理 | 统一管理多格式结果 | `resultfilemanager.cc` |
| 数据导出 | CSV、JSON、OMNeT++ 格式导出 | `exporter*.cc` |
| SQLite 结果后端 | 结果存储与查询 | `sqlitescalarfilewriter.h`, `sqlitevectorfilewriter.h` |
| Python API | `omnetpp.scave` 包，数据分析和绘图 | `python/omnetpp/scave/` |

## 七、事件日志处理 (EventLog)

| 功能 | 说明 |
|------|------|
| 事件日志解析 | 事件轨迹文件读取与解析 |
| 事件过滤 | 条件过滤事件记录 |
| 消息依赖追踪 | 消息发送-接收依赖关系 |
| 序列图可视化 | IDE 中的序列图显示 |
| 事件日志表格 | IDE 中的事件日志浏览器 |

## 八、图形布局 (Layout)

| 功能 | 说明 |
|------|------|
| 力导向布局 | 弹簧嵌入器算法 |
| 树形嵌入 | 层次化树形布局 |
| 图形布局 | 网络拓扑自动布局 |

## 九、IDE (集成开发环境)

### 核心功能

| 功能 | 说明 |
|------|------|
| NED 编辑器 | 图形化与文本双模式 NED 编辑 |
| INI 文件编辑器 | 表单与文本双模式配置编辑 |
| MSG 文件编辑器 | 消息定义文件语法高亮与编辑 |
| NED 文档生成器 | 从 NED 生成 HTML 文档 |
| 仿真启动配置 | 多种运行/调试配置管理 |
| CDT 集成 | Eclipse C/C++ 开发工具集成 |
| 分析工具 (Scave) | 结果分析与绘图 |
| Python 图表支持 | 基于 Python 的自定义图表 |
| 事件日志浏览器 | 事件列表查看 |
| 序列图 | 事件序列可视化 |
| 原生库加载 | 平台相关 C++ 库加载 |

### 插件体系

| 组 | 包含插件 | 职责 |
|----|----------|------|
| 核心 | main, common, common.core, nativelibs | IDE 基础与共享工具 |
| NED | ned.core, ned.model, ned.editor, neddoc | NED 语言支持 |
| 分析 | scave, scave.model, scave.builder, scave.pychart, scave.templates | 结果分析 |
| 编辑器 | inifile.editor, msg.editor, eventlogtable, sequencechart, figures | 各类文件编辑器 |
| 启动 | launch, cdt, dsp | 仿真运行与调试 |

## 十、命令行工具

| 工具 | 路径 | 功能 |
|------|------|------|
| `opp_run` | `src/envir/main.cc` | 仿真主运行器 |
| `opp_nedtool` | `src/nedxml/` | NED 文件编译/验证 |
| `opp_msgtool` | `src/nedxml/` | MSG 文件编译器 |
| `opp_scavetool` | `src/scave/` | 结果文件处理与分析 |
| `opp_test` | `src/utils/` | 测试运行器 |
| `opp_makemake` | `src/utils/` | Makefile 生成器 |
| `opp_featuretool` | `src/utils/` | 特性开关管理 |
| `opp_charttool` | `src/utils/` | 图表生成 |
| `opp_neddoc` | `src/utils/` | NED 文档生成器 |

## 十一、Python 绑定与工具

| 模块 | 功能 |
|------|------|
| `omnetpp.scave` | 结果分析与绘图（主要 Python API） |
| `omnetpp.ned` | NED 文件处理 |
| `omnetpp.nedast` | NED AST 表示 |
| `omnetpp.nedlinter` | NED 代码风格检查 |
| `omnetpp.repl` | 交互式 Python REPL |
| `omnetpp.test` | 测试工具 |
| `omnetpp.lldb` | LLDB 调试器集成 |
| `omnetpp.llmtool` | LLM 代码辅助工具（C++ 编码指南、代码改进模板、补丁格式） |

## 十二、通用工具库 (Common)

| 功能 | 说明 | 关键文件 |
|------|------|----------|
| 表达式解析器 | 数学与逻辑表达式求值 | `expression.cc/h`, `exprnodes.cc/h` |
| 字符串工具 | `opp_isempty`, `opp_streq` 等 | `stringutil.cc/h` |
| 文件 I/O | 文件读写、行读取 | `fileutil.cc/h`, `filereader.cc/h` |
| JSON 输出 | JSON 格式写入 | `jsonwriter.cc/h` |
| CSV 输出 | CSV 格式写入 | `csvwriter.cc/h` |
| SQLite 集成 | 嵌入式 SQLite 数据库 | `sqlite3.c/h`, `sqlitedatabase.cc/h` |
| 模式匹配 | glob 风格模式匹配 | `patternmatcher.cc/h`, `matchexpression.cc/h` |
| BigDecimal | 高精度十进制运算 | `bigdecimal.cc/h` |
| 单位转换 | 物理单位换算 | `unitconversion.cc/h` |
| SAX 解析器 | XML 解析（libxml2 / yxml） | `saxparser.h`, `saxparser_libxml.cc`, `saxparser_yxml.cc` |
| 线程工具 | 读写锁 | `rwlock.cc/h` |
| 数量格式化 | 物理量格式化 | `quantityformatter.cc/h` |
| 异常处理 | 仿真异常类 | `exception.cc/h` |
| 错误存储 | 编译验证错误收集 | `errorstore.cc/h` |

## 十三、测试框架

| 功能 | 说明 |
|------|------|
| 自定义测试格式 | `.test` 文件格式（`%activity`, `%contains`, `%not-contains` 等） |
| 测试运行器 `opp_test` | 从 .test 生成 C++ 代码，编译、运行、比较输出 |
| 内核测试 | 750+ .test 文件覆盖仿真内核 |
| 指纹回归测试 | 仿真结果不变性验证 |
| 多测试套件 | core, common, envir, scave, anim, models, build, IDE 等 |
| 快速测试 | `make test_quick` 快速子集 |

## 十四、构建系统

| 功能 | 说明 |
|------|------|
| `./configure` | 自动检测编译器、路径、标志 |
| `Makefile.inc` | 生成的构建配置 |
| `opp_makemake` | 自动生成 Makefile |
| 多构建模式 | release (默认), debug, sanitize, coverage, profile |
| 库命名规范 | `$(LIB_PREFIX)opp<subsystem>$D$(LIB_SUFFIX)` |
| 依赖链构建 | common → sim → envir → cmdenv/qtenv |
| IDE 原生库 | `make ui` 构建 IDE 所需 JNI 库 |
| 安装脚本 | `install.sh` 一键构建安装 |
| Docker 构建 | `ghcr.io/omnetpp/distrobuild` 容器化构建 |

## 十五、示例仿真

| 示例 | 展示功能 |
|------|----------|
| `tictoc/` | 入门教程（15步渐进式） |
| `aloha/` | ALOHA 协议、参数研究 |
| `queueinglib/` | 可复用排队库（63文件，最大示例） |
| `routing/` | 网络路由算法 |
| `fifo/` | 基本 FIFO 队列仿真 |
| `dyna/` | 动态模块创建/删除 |
| `cqn/` | 闭环排队网络 |
| `canvas/` | Canvas API 使用 |
| `osg-*` | OpenSceneGraph 3D 可视化 |
| `petrinets/` | Petri 网建模 |
| `sockets/` | 真实 socket 通信 |
| `resultfiles/` | 结果记录示例 |
| `neddemo/` | NED 语言特性演示 |
| `embedding/`, `embedding2/` | OMNeT++ 嵌入其他应用 |

## 十六、CI/CD

| 工作流 | 说明 |
|--------|------|
| `build_release.yml` | 发布构建 |
| `build_tests.yml` | Ubuntu 24.04, Clang, Qt6 构建+测试 |
| `main_tests.yml` | 主测试套件 |

---

*文件生成时间：2026-04-24*