# OMNeT++ 公共 API 分析文档

## 概述

本文档分析 OMNeT++ 6.4.0 版本的公共 API，涵盖 `include/omnetpp/` 目录下的 124 个头文件。这些头文件定义了仿真模型开发者可使用的公共接口。

## API 类别分类

根据 `index.h` 中的 Doxygen 分组定义，公共 API 分为以下主要类别：

### 1. 基础类 (Fundamentals)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cObject` | `cobject.h` | OMNeT++ 类层次结构的根类，提供命名、克隆、所有权管理等基础机制 |
| `cOwnedObject` | `cownedobject.h` | 支持所有权追踪的对象基类 |
| `cNamedObject` | `cnamedobject.h` | 具有名称的对象类 |
| `cCoroutine` | `ccoroutine.h` | 协程类，用于 activity() 方法的实现 |
| `cRuntimeError` | `cexception.h` | 运行时错误异常类 |
| `cClassDescriptor` | `cclassdescriptor.h` | 反射信息提供类 |

### 2. 模型组件类 (Model Components)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cModule` | `cmodule.h` | 模块基类，表示仿真中的模块 |
| `cSimpleModule` | `csimplemodule.h` | 简单模块基类，用户通过继承此类实现仿真逻辑 |
| `cChannel` | `cchannel.h` | 通道基类，表示模块间的连接 |
| `cIdealChannel` | `cchannel.h` | 理想通道（零延迟、无限带宽） |
| `cDatarateChannel` | `cdataratechannel.h` | 数据率通道，支持传输延迟建模 |
| `cDelayChannel` | `cdelaychannel.h` | 固定延迟通道 |
| `cGate` | `cgate.h` | 门类，表示模块的连接点 |
| `cPar` | `cpar.h` | 参数类，表示模块和通道的参数 |
| `cProperties` | `cproperties.h` | 属性集合类 |

### 3. 消息类 (Simulation Programming)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cMessage` | `cmessage.h` | 消息类，表示仿真事件和消息 |
| `cPacket` | `cpacket.h` | 数据包类，继承自 cMessage，增加长度和封装能力 |
| `cEvent` | `cevent.h` | 事件基类 |
| `cQueue` | `cqueue.h` | 队列类，FIFO 或优先级队列 |
| `cPacketQueue` | `cpacketqueue.h` | 数据包专用队列 |
| `cTopology` | `ctopology.h` | 拓扑工具类，用于发现模型拓扑和寻找最短路径 |

### 4. 仿真核心类 (Simulation Core)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cSimulation` | `csimulation.h` | 仿真管理器类，存储网络模型和事件调度 |
| `cFutureEventSet` | `cfutureeventset.h` | 未来事件集合接口 |
| `cEventHeap` | `ceventheap.h` | 基于堆的事件集合实现 |
| `cScheduler` | `cscheduler.h` | 事件调度器接口 |

### 5. 时间类 (Simulation Time)

| 类型/类名 | 头文件 | 说明 |
|-----------|--------|------|
| `simtime_t` | `simtime_t.h` | 仿真时间类型（SimTime 的别名） |
| `SimTime` | `simtime.h` | 仿真时间类，64位定点表示 |

**仿真时间宏：**
- `SIMTIME_MAX` - 最大可表示的仿真时间
- `SIMTIME_ZERO` - 零仿真时间
- `SIMTIME_STR(t)` - 转换为 C 字符串
- `SIMTIME_DBL(t)` - 转换为 double（有精度损失）

### 6. 随机数生成类 (Random Numbers)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cRNG` | `crng.h` | 随机数生成器接口 |
| `cMersenneTwister` | `cmersennetwister.h` | Mersenne Twister RNG 实现 |
| `cLCG32` | `clcg32.h` | 32位线性同余生成器 |
| `cRandom` | `crandom.h` | 随机变量生成器基类 |
| `cUniform` | `crandom.h` | 均匀分布 |
| `cExponential` | `crandom.h` | 指数分布 |
| `cNormal` | `crandom.h` | 正态分布 |
| `cTruncNormal` | `crandom.h` | 截断正态分布 |
| `cGamma` | `crandom.h` | Gamma 分布 |
| `cBeta` | `crandom.h` | Beta 分布 |
| `cErlang` | `crandom.h` | Erlang 分布 |
| `cChiSquare` | `crandom.h` | 卡方分布 |
| `cStudentT` | `crandom.h` | Student's T 分布 |
| `cCauchy` | `crandom.h` | Cauchy 分布 |
| `cTriang` | `crandom.h` | 三角分布 |
| `cWeibull` | `crandom.h` | Weibull 分布 |
| `cParetoShifted` | `crandom.h` | 平移 Pareto 分布 |
| `cIntUniform` | `crandom.h` | 离散均匀分布 |
| `cIntUniformExcl` | `crandom.h` | 排除边界的离散均匀分布 |
| `cBernoulli` | `crandom.h` | 伯努利分布 |
| `cBinomial` | `crandom.h` | 二项分布 |
| `cGeometric` | `crandom.h` | 几何分布 |
| `cNegBinomial` | `crandom.h` | 负二项分布 |
| `cPoisson` | `crandom.h` | 泊松分布 |

### 7. 统计类 (Statistics)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cStatistic` | `cstatistic.h` | 统计基类 |
| `cStdDev` | `cstddev.h` | 标准统计（均值、标准差、最小/最大值） |
| `cHistogram` | `chistogram.h` | 直方图类 |
| `cPSquare` | `cpsquare.h` | P² 算法分位数计算 |
| `cOutVector` | `coutvector.h` | 输出向量记录器 |
| `cAbstractHistogram` | `cabstracthistogram.h` | 抽象直方图基类 |
| `cKSplit` | `cksplit.h` | K-split 直方图 |
| `cPrecollDensityEst` | `cprecolldensityest.h` | 密度估计器 |

### 8. 信号类 (Signals)

| 类名/类型 | 头文件 | 说明 |
|-----------|--------|------|
| `simsignal_t` | `clistener.h` | 信号句柄类型 |
| `cIListener` | `clistener.h` | 监听器接口 |
| `cListener` | `clistener.h` | 监听器默认实现 |
| `cResultFilter` | `cresultfilter.h` | 结果过滤器基类 |
| `cResultRecorder` | `cresultrecorder.h` | 结果记录器基类 |

### 9. 表达式类 (Expressions)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cExpression` | `cexpression.h` | 表达式基类 |
| `cDynamicExpression` | `cdynamicexpression.h` | 运行时解析的表达式 |
| `cMatchExpression` | `cmatchexpression.h` | 匹配表达式 |
| `cValue` | `cvalue.h` | 变体值类 |
| `cValueMap` | `cvaluemap.h` | 值映射（JSON 对象风格） |
| `cValueArray` | `cvaluearray.h` | 值数组（JSON 数组风格） |
| `cValueHolder` | `cvalueholder.h` | 值持有器 |
| `cXmlElement` | `cxmlelement.h` | XML 元素 |

### 10. 画布类 (Canvas)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cCanvas` | `ccanvas.h` | 2D 画布 |
| `cFigure` | `ccanvas.h` | 图形基类 |
| `cGroupFigure` | `ccanvas.h` | 分组图形 |
| `cLineFigure` | `ccanvas.h` | 线段图形 |
| `cArcFigure` | `ccanvas.h` | 弧形图形 |
| `cPolylineFigure` | `ccanvas.h` | 折线图形 |
| `cRectangleFigure` | `ccanvas.h` | 矩形图形 |
| `cOvalFigure` | `ccanvas.h` | 椭圆图形 |
| `cRingFigure` | `ccanvas.h` | 环形图形 |
| `cPieSliceFigure` | `ccanvas.h` | 饼图切片图形 |
| `cPolygonFigure` | `ccanvas.h` | 多边形图形 |
| `cPathFigure` | `ccanvas.h` | 路径图形（SVG 风格） |
| `cTextFigure` | `ccanvas.h` | 文本图形 |
| `cLabelFigure` | `ccanvas.h` | 标签图形（非缩放） |
| `cImageFigure` | `ccanvas.h` | 图像图形 |
| `cIconFigure` | `ccanvas.h` | 图标图形（非缩放） |
| `cPixmapFigure` | `ccanvas.h` | 像素图图形 |

### 11. OSG 3D 支持类 (OSG)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cOsgCanvas` | `cosgcanvas.h` | OpenSceneGraph 3D 画布 |

### 12. 有限状态机类 (FSM)

| 宏/类 | 头文件 | 说明 |
|-------|--------|------|
| `FSM_Switch()` | `cfsm.h` | FSM 主宏 |
| `cFSM` | `cfsm.h` | 有限状态机类 |

### 13. 日志类 (Logging)

| 宏 | 头文件 | 说明 |
|----|--------|------|
| `EV_LOG()` | `clog.h` | 日志宏 |
| `EV_STATICCONTEXT` | `clog.h` | 静态上下文宏 |

### 14. 环境与扩展类 (Envir and Extensions)

| 类名 | 头文件 | 说明 |
|------|--------|------|
| `cEnvir` | `cenvir.h` | 仿真环境接口 |
| `cConfiguration` | `cconfiguration.h` | 配置接口 |
| `cNullEnvir` | `cnullenvir.h` | 空环境实现 |
| `cConfigOption` | `cconfigoption.h` | 配置选项类 |
| `cConfigReader` | `cconfigreader.h` | 配置读取器 |

---

## 命名约定

### 类命名

OMNeT++ 公共 API 类使用 **c-前缀** 命名约定：

```
cObject        - 基础对象类
cModule        - 模块类
cMessage       - 消息类
cSimulation    - 仿真类
cGate          - 门类
cChannel       - 通道类
```

### 方法命名

方法使用 **camelCase** 驼峰命名法：

```cpp
getName()           // 获取名称
getFullName()       // 获取全名
getFullPath()       // 获取完整路径
handleMessage()     // 处理消息
scheduleAt()        // 调度自消息
```

### C 函数命名

自由函数使用 **opp_** 前缀：

```cpp
opp_isempty(s)      // 检查字符串是否为空
opp_streq(a, b)     // 字符串比较
opp_typename()      // 获取类型名
```

### 宏命名

宏使用 **大写下划线** 或 **Register_** 前缀：

```cpp
SIM_API                // 导出宏
SIMTIME_ZERO           // 仿真时间常量
Register_Class()       // 注册类
Define_Module()        // 定义模块
Register_Enum()        // 注册枚举
```

---

## API 导出宏

### SIM_API

`SIM_API` 是公共 API 的导出宏，用于控制符号的可见性：

- 在构建仿真库时，`SIM_API` 定义为导出符号
- 在使用仿真库时，`SIM_API` 定义为导入符号
- 所有公共 API 类都使用此宏声明

**示例：**
```cpp
class SIM_API cObject { ... };
class SIM_API cModule : public cComponent { ... };
```

**统计：** 113 个文件中使用 `SIM_API`，共 379 处。

---

## 关键公共接口详解

### cObject - 对象基类

`cObject` 是 OMNeT++ 类层次结构的根类，提供以下核心机制：

**名称管理：**
- `getName()` - 获取对象名称
- `getFullName()` - 获取完整名称（含索引）
- `getFullPath()` - 获取完整路径

**对象操作：**
- `dup()` - 克隆对象
- `str()` - 获取对象描述字符串
- `forEachChild()` - 遍历子对象

**所有权管理：**
- `getOwner()` - 获取所有者
- `take()` - 获取对象所有权
- `drop()` - 释放对象所有权

**反射支持：**
- `getDescriptor()` - 获取类描述符

### cModule - 模块基类

`cModule` 表示仿真中的模块，提供：

**模块信息：**
- `getParentModule()` - 获取父模块
- `getModuleType()` - 获取模块类型
- `getIndex()` - 获取模块向量索引
- `getVectorSize()` - 获取模块向量大小

**子模块管理：**
- `hasSubmodules()` - 检查是否有子模块
- `getSubmodule()` - 获取子模块
- `addSubmodule()` - 添加子模块
- `SubmoduleIterator` - 子模块迭代器

**门管理：**
- `gate()` - 获取门
- `addGate()` - 添加门
- `addGateVector()` - 添加门向量
- `GateIterator` - 门迭代器

### cSimpleModule - 简单模块

`cSimpleModule` 是用户实现仿真逻辑的核心类：

**生命周期方法（用户重写）：**
- `initialize()` - 初始化
- `handleMessage(cMessage*)` - 消息处理
- `activity()` - 活动方法（不推荐）
- `finish()` - 结束处理

**消息发送：**
- `send(cMessage*, cGate*)` - 发送消息
- `sendDirect(cMessage*, cModule*, ...)` - 直接发送
- `scheduleAt(simtime_t, cMessage*)` - 调度自消息
- `cancelEvent(cMessage*)` - 取消事件

**SendOptions 结构：**
- `after(delay)` - 延迟发送
- `propagationDelay(delay)` - 传播延迟
- `duration(dur)` - 传输持续时间
- `transmissionId(id)` - 传输 ID
- `updateTx(id)` - 更新传输
- `finishTx(id)` - 完成传输

### cMessage - 消息类

`cMessage` 表示仿真中的事件和消息：

**消息属性：**
- `getKind()` / `setKind()` - 消息类型
- `getTimestamp()` / `setTimestamp()` - 时间戳
- `getContextPointer()` / `setContextPointer()` - 上下文指针
- `getControlInfo()` / `setControlInfo()` - 控制信息

**发送/到达信息：**
- `isSelfMessage()` - 是否为自消息
- `getSenderModule()` - 获取发送模块
- `getArrivalModule()` - 获取到达模块
- `getSendingTime()` - 获取发送时间
- `getArrivalTime()` - 获取到达时间

**动态附件：**
- `addPar()` - 添加参数
- `par()` - 获取参数
- `addObject()` - 添加对象

### cPacket - 数据包类

`cPacket` 继承自 `cMessage`，增加：

**数据包属性：**
- `getBitLength()` / `setBitLength()` - 长度（位）
- `getByteLength()` / `setByteLength()` - 长度（字节）
- `hasBitError()` / `setBitError()` - 比特错误标志

**封装支持：**
- `encapsulate(cPacket*)` - 封装数据包
- `decapsulate()` - 解封装数据包
- `getEncapsulatedPacket()` - 获取封装的数据包

**传输状态：**
- `getDuration()` - 传输持续时间
- `getRemainingDuration()` - 剩余传输时间
- `isReceptionStart()` / `isReceptionEnd()` - 接收状态

### cSimulation - 仿真管理类

`cSimulation` 是仿真的中央管理类：

**仿真信息：**
- `getSimTime()` - 获取当前仿真时间
- `getEventNumber()` - 获取事件序号
- `getWarmupPeriod()` - 获取预热期

**模块访问：**
- `getModule(id)` - 按 ID 获取模块
- `getModuleByPath(path)` - 按路径获取模块
- `getSystemModule()` - 获取系统模块

**全局函数：**
- `simTime()` - 获取当前仿真时间
- `getSimulation()` - 获取当前仿真对象
- `getEnvir()` - 获取环境对象

---

## 用户可用的全局函数

### 仿真时间

```cpp
simtime_t simTime();  // 返回当前仿真时间
```

### 模块查找

```cpp
cSimulation* getSimulation();  // 获取仿真对象
cEnvir* getEnvir();            // 获取环境对象
```

### 字符串工具

```cpp
bool opp_isempty(const char* s);   // 检查字符串是否为空
bool opp_streq(const char* a, const char* b);  // 字符串比较
```

---

## 注册宏

### 模块注册

```cpp
Define_Module(MyModule);  // 注册简单模块类
```

### 通道注册

```cpp
Define_Channel(MyChannel);  // 注册通道类
```

### 类注册

```cpp
Register_Class(MyClass);  // 注册类（支持按名称创建实例）
```

### 枚举注册

```cpp
enum State { IDLE, BUSY, SLEEPING };
Register_Enum(State, (IDLE, BUSY, SLEEPING));
```

### 函数注册

```cpp
Define_NED_Function(myFunc, "double myFunc(double x)");
Define_NED_Math_Function(sin, 1);
```

---

## 内部类排除说明

标记为 `@ingroup Internals` 的类属于内部实现，不在公共 API 文档范围内。这些类：

- 用于仿真内核内部
- 可能频繁变更
- 不保证向后兼容性

---

## 头文件完整列表

公共 API 头文件（`include/omnetpp/*.h`）共 124 个，包括：

**核心头文件：**
- `index.h` - API 索引和分组定义
- `simkerneldefs.h` - 内核定义
- `cobject.h`, `cownedobject.h`, `cnamedobject.h` - 基础类
- `cmodule.h`, `csimplemodule.h` - 模块类
- `cmessage.h`, `cpacket.h` - 消息类
- `csimulation.h` - 仿真类
- `simtime.h`, `simtime_t.h` - 时间类
- `cgate.h`, `cchannel.h` - 连接类
- `cpar.h` - 参数类

**工具头文件：**
- `cqueue.h`, `cpacketqueue.h` - 队列
- `ctopology.h` - 拓扑
- `carray.h` - 数组
- `cstringtokenizer.h` - 字符串分词
- `cpatternmatcher.h` - 模式匹配

**统计头文件：**
- `cstatistic.h`, `cstddev.h`, `chistogram.h` - 统计类
- `coutvector.h` - 输出向量
- `resultfilters.h`, `resultrecorders.h` - 结果处理

**随机数头文件：**
- `crng.h`, `crandom.h` - RNG 基类
- `distrib.h` - 分布函数

**信号头文件：**
- `clistener.h` - 监听器
- `cresultfilter.h`, `cresultrecorder.h` - 结果过滤器/记录器

**其他头文件：**
- `regmacros.h` - 注册宏
- `cwatch.h` - 监视宏
- `clog.h` - 日志
- `ccanvas.h`, `cosgcanvas.h` - 画布
- `cfsm.h` - 有限状态机
- `cexception.h` - 异常

---

## 总结

OMNeT++ 公共 API 提供了完整的离散事件仿真框架接口：

1. **基础架构**：cObject 层次结构提供命名、克隆、所有权管理
2. **模型组件**：cModule、cChannel、cGate、cPar 构建仿真模型
3. **消息机制**：cMessage、cPacket 实现事件驱动
4. **仿真核心**：cSimulation、cScheduler 管理仿真执行
5. **统计收集**：cStatistic、cOutVector 记录仿真结果
6. **随机数**：cRNG 和多种分布类支持随机建模
7. **扩展机制**：注册宏、信号机制支持灵活扩展

API 遵循一致的命名约定（c-前缀类、camelCase 方法），通过 SIM_API 导出宏确保跨平台兼容性。