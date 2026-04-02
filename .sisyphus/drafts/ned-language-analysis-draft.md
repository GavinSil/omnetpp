# OMNeT++ NED语言详细分析草案

> **版本**: OMNeT++ 6.4.0
> **生成日期**: 2026-04-02
> **状态**: 草案文档 - 基于官方文档和示例代码分析

---

## 目录

1. [NED语言概述](#1-ned语言概述)
2. [核心概念](#2-核心概念)
3. [语法规范](#3-语法规范)
4. [模块定义方法](#4-模块定义方法)
5. [参数系统](#5-参数系统)
6. [门(Gate)系统](#6-门gate系统)
7. [连接系统](#7-连接系统)
8. [继承与接口](#8-继承与接口)
9. [属性(Properties)](#9-属性properties)
10. [完整示例分析](#10-完整示例分析)
11. [最佳实践](#11-最佳实践)

---

## 1. NED语言概述

### 1.1 定义

NED (Network Description) 是OMNeT++的拓扑描述语言，用于声明仿真模型的结构。NED描述模块的接口（参数、门）、模块间的连接关系，以及网络拓扑。

**关键特点：**
- **分层化** - 通过模块嵌套管理复杂度
- **组件化** - 模块可复用，支持框架库（如INET）
- **继承** - 模块/通道可继承扩展
- **接口** - 模块/通道接口作为占位符
- **包机制** - Java风格的包结构避免命名冲突
- **内部类型** - 复合模块内定义局部类型
- **元数据注解** - Properties提供额外信息

### 1.2 NED语言 vs UML

NED和UML在设计目标和领域特异性上有本质差异。NED是领域特定语言(DSL)，专为离散事件仿真设计；UML是通用建模语言(GML)，用于软件系统全方位建模。

#### 设计目标差异

| 方面 | NED | UML |
|------|-----|-----|
| **设计目标** | 离散事件仿真拓扑 | 通用软件系统建模 |
| **领域特异性** | 高 - 专为仿真设计 | 低 - 通用建模语言 |
| **可执行性** | 直接编译运行 | 需代码生成转换 |
| **标准化程度** | OMNeT++专有 | ISO/IEC 19505国际标准 |

#### 功能对比

##### NED优势（仿真领域）

| 特性 | NED | UML | 说明 |
|------|-----|-----|------|
| **门向量** | `gate[10]` 原生支持 | 需手动标注 | 模块多端口场景常见 |
| **循环连接** | `for i=0..n-1` | 无原生支持 | 星型/环形拓扑简洁表达 |
| **通道语义** | `delay`, `datarate` 内置 | 需自定义profile | 通信延迟建模核心需求 |
| **参数绑定** | 关联omnetpp.ini | 无配置文件机制 | 实验场景切换便捷 |
| **接口多态** | `<> like IApp` | 接口+依赖注入 | 运行时类型灵活选择 |
| **表达式求值** | `size = 2*N+1` | 属性值仅为字符串 | 动态拓扑生成 |
| **单位系统** | `@unit(s)` | 无 | 量纲错误编译检查 |

##### UML优势（通用建模）

| 特性 | UML | NED | 说明 |
|------|-----|-----|------|
| **图表类型** | 14种图表 | 仅结构定义 | 行为/交互/部署等全方位 |
| **可视化工具** | Enterprise Architect等 | OMNeT++ IDE有限支持 | 专业建模工具生态 |
| **代码生成** | 多语言支持 | 仅C++ | Java/C#/Python等 |
| **文档价值** | 行业标准文档 | 项目内部使用 | 利于团队沟通/评审 |
| **团队协作** | 广泛认知 | 小众领域 | 减少学习成本 |
| **MDD框架** | EMF, MetaEdit+ | 无 | 模型驱动开发生态 |

#### 适用场景划分

```
┌─────────────────────────────────────────────────────────────┐
│                     系统建模需求                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌───────────────┐         ┌───────────────┐               │
│   │  架构文档     │  UML    │  静态结构     │               │
│   │  (类图/包图)  │ ──────→ │  评审沟通     │               │
│   └───────────────┘         └───────────────┘               │
│                                                             │
│   ┌───────────────┐         ┌───────────────┐               │
│   │  消息交互     │  UML    │  时序分析     │               │
│   │  (序列图)     │ ──────→ │  协议设计     │               │
│   └───────────────┘         └───────────────┘               │
│                                                             │
│   ┌───────────────┐         ┌───────────────┐               │
│   │  状态行为     │  UML    │  模块逻辑     │               │
│   │  (状态图)     │ ──────→ │  算法设计     │               │
│   └───────────────┘         └───────────────┘               │
│                                                             │
│   ┌───────────────┐         ┌───────────────┐               │
│   │  拓扑定义     │  NED    │  可执行仿真   │  ← 核心差异    │
│   │  (网络结构)   │ ──────→ │  参数绑定     │               │
│   └───────────────┘         └───────────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 实际案例对比

**场景：设计一个10节点的Ring网络**

**NED实现（可执行）：**
```ned
network RingNetwork {
    parameters:
        int numNodes = 10;
        double delay = 10ms @unit(s);
    submodules:
        node[numNodes]: Node {
            @display("p=100,100,ring");
        }
    connections:
        for i=0..numNodes-1 {
            node[i].out --> { delay = delay; } --> node[(i+1)%numNodes].in;
        }
}
```

**UML类图（文档性）：**
```
┌─────────────────┐
│    RingNetwork  │
├─────────────────┤
│ numNodes: int   │
│ delay: Time     │
├─────────────────┤
│ nodes: Node[10] │
└─────────────────┘
        │
        │ contains 10
        ▼
┌─────────────────┐
│      Node       │
├─────────────────┤
│ in: Gate        │◄───┐
│ out: Gate       │───►│ ring connection
└─────────────────┘    │
                       │
            (association with delay constraint)
```

**关键差异：**
- NED直接运行仿真，UML需转换为代码
- NED的`for`循环自动生成10条连接，UML需手动标注或注释说明
- NED参数可覆盖，UML属性为静态值

#### 协同使用建议

| 模型阶段 | 推荐工具 | 产出物 |
|----------|----------|--------|
| 需求分析 | UML用例图 | 系统边界定义 |
| 架构设计 | UML类图/包图 | 模块类型层次 |
| 交互设计 | UML序列图 | 消息流规范 |
| 行为设计 | UML状态图 | 模块状态机 |
| **拓扑实现** | **NED** | 可执行网络定义 |
| 参数配置 | omnetpp.ini | 实验场景配置 |

#### 结论

| 选择 | 场景 |
|------|------|
| **必用NED** | 仿真拓扑定义、参数绑定、可执行模型 |
| **必用UML** | 团队沟通文档、行为建模、非仿真系统 |
| **协同使用** | 大型仿真项目 - UML设计，NED实现 |

**本质差异：NED是DSL（领域特定语言），UML是GML（通用建模语言）。DSL在特定领域效率更高，GML在跨领域沟通上更强。**

### 1.3 NED语言 vs XML

NED文件可以无损转换为XML格式，反之亦然。但NED作为主要建模语言具有显著优势。

#### 对比表

| 方面 | NED | XML |
|------|-----|-----|
| **可读性** | 高 - 类C语法，简洁直观 | 低 - 冗长标签嵌套 |
| **编写效率** | 高 - 紧凑语法 | 低 - 大量标签开销 |
| **学习曲线** | 平缓 - 类似编程语言 | 陡峭 - 需理解DTD结构 |
| **继承表达** | `extends` 关键字，直观 | 需XML机制或手动复制 |
| **循环连接** | `for i=0..n-1 { }` 直接表达 | 需展开或预处理 |
| **条件连接** | `if condition` 内联 | 无法直接表达 |
| **表达式** | 原生支持算术/逻辑表达式 | 仅文本，需外部求值 |

#### 代码对比示例

**NED（简洁）：**
```ned
network Network {
    submodules:
        node[10]: Node;
    connections:
        for i=0..9 {
            node[i].out --> { delay = 10ms; } --> node[(i+1)%10].in;
        }
}
```

**XML等价表示（冗长）：**
```xml
<ned-file>
  <network name="Network">
    <submodules>
      <submodule name="node" vector-size="10" type="Node"/>
    </submodules>
    <connections>
      <connection-group>
        <loop var="i" from="0" to="9">
          <connection src="node[i].out" dest="node[(i+1)%10].in">
            <channel type="ned.DelayChannel">
              <param name="delay" value="10ms"/>
            </channel>
          </connection>
        </loop>
      </connection-group>
    </connections>
  </network>
</ned-file>
```

#### NED独有特性

| 特性 | 说明 | 示例 |
|------|------|------|
| **表达式求值** | 参数可含动态表达式 | `capacity = 2 * numNodes + 1` |
| **模式赋值** | 通配符批量配置 | `host[*].queue.capacity = 10` |
| **接口多态** | 运行时确定具体类型 | `<> like IInterface` |
| **参数推导** | 自动获取向量大小 | `sizeof(port)` |
| **单位检查** | 编译时量纲验证 | `@unit(s)` |

#### XML适用场景

| 场景 | 原因 |
|------|------|
| 工具生成拓扑 | 程序生成，无需人工编辑 |
| 跨系统交换 | XML是通用数据格式 |
| 批量转换 | NED ↔ XML 无损互转 |
| 版本控制 | 差异比较更精确（行级变化） |

#### 官方立场

> NED files can be converted to XML and back without any data loss, including comments. This makes it easier to programmatically manipulate NED files.
> — OMNeT++ Manual

**结论：NED为人类设计，XML为工具设计。** 日常建模使用NED，程序化处理时用XML。

### 1.4 文件组织

```
项目结构：
├── package.ned          # 包定义文件（可选）
├── Node.ned             # 每个模块类型一个文件（推荐）
├── Network.ned          # 网络定义
└── omnetpp.ini          # 配置文件（参数赋值）
```

**命名约定：**
- 模块类型名：大写开头 (`Queue`, `Node`)
- 参数/门名：小写开头 (`capacity`, `in`, `out`)
- 文件名：与模块名相同 (`Queue.ned`)

---

## 2. 核心概念

### 2.1 模块类型层级

```
┌─────────────────────────────────────────┐
│           Module Interface              │  <- 抽象接口
│   (moduleinterface INode {...})         │
└─────────────────────────────────────────┘
                    ↓ implements
┌─────────────────────────────────────────┐
│           Simple Module                 │  <- 基础模块（C++实现）
│   (simple Queue {...})                  │
└─────────────────────────────────────────┘
                    ↓ extends
┌─────────────────────────────────────────┐
│         Compound Module                 │  <- 组合模块（子模块+连接）
│   (module Host {...})                   │
└─────────────────────────────────────────┘
                    ↓ extends
┌─────────────────────────────────────────┐
│            Network                      │  <- 顶层网络（compound的特殊形式）
│   (network Network {...})               │
└─────────────────────────────────────────┘
```

### 2.2 组件关系图

```
┌──────────────┐         ┌──────────────┐
│  NED文件     │ import  │  其他NED文件  │
│  (定义)      │───────→ │  (类型引用)   │
└──────────────┘         └──────────────┘
       ↓                      ↓
       │ 使用                  │ 使用
       ↓                      ↓
┌──────────────┐         ┌──────────────┐
│  C++类       │ 实现    │  omnetpp.ini │
│  (行为)      │←─────── │  (配置)      │
└──────────────┘         └──────────────┘
```

---

## 3. 语法规范

### 3.1 基本语法规则

```ned
// 注释：单行注释
/* 注释：多行注释 */

// 关键字（小写）
simple, module, network, channel
parameters, gates, submodules, connections, types
input, output, inout
extends, like, import, package

// 标识符规则
- 大小写敏感
- 字母开头，包含字母、数字、下划线
- 类型名首字母大写
- 参数/门名首字母小写
```

### 3.2 完整BNF语法摘要（关键部分）

```bnf
nedfile ::= definitions

definition ::=
    | packagedeclaration
    | import
    | channeldefinition
    | simplemoduledefinition
    | compoundmoduledefinition
    | networkdefinition
    | moduleinterfacedefinition
    | propertydecl
    | ';'

simplemoduledefinition ::=
    SIMPLE NAME opt_inheritance '{'
        opt_paramblock
        opt_gateblock
    '}'

compoundmoduledefinition ::=
    MODULE NAME opt_inheritance '{'
        opt_paramblock
        opt_gateblock
        opt_typeblock      // 内部类型定义
        opt_submodblock    // 子模块
        opt_connblock      // 连接
    '}'

networkdefinition ::=
    NETWORK NAME opt_inheritance '{'
        [同compound module]
    '}'

param ::=
    | param_typename '=' paramvalue ';'
    | param_typename ';'
    | parampattern '=' paramvalue ';'  // pattern赋值

paramtype ::=
    | DOUBLE | INT | STRING | BOOL | OBJECT | XML

gate ::=
    gatetype NAME ';'
    | gatetype NAME '[' ']' ';'       // 向量门（大小未定）
    | gatetype NAME '[' expr ']' ';'  // 向量门（大小固定）

gatetype ::= INPUT | OUTPUT | INOUT

connection ::=
    | leftgatespec '-->' rightgatespec
    | leftgatespec '-->' channelspec '-->' rightgatespec
    | leftgatespec '<-->' rightgatespec  // 双向连接
    | leftgatespec '<-->' channelspec '<-->' rightgatespec

submodule ::=
    | submodulename ':' dottedname opt_condition ';'
    | submodulename ':' likeexpr LIKE dottedname opt_condition  // parametric type
    | [带参数块的完整形式]
```

---

## 4. 模块定义方法

### 4.1 简单模块 (Simple Module)

简单模块是仿真模型的基本构建块，所有行为逻辑都在C++中实现。NED只声明接口。

**基本结构：**

```ned
simple ModuleName
{
    parameters:
        @display("i=block/queue");  // 显示字符串（图标）
        int capacity = default(-1);  // 参数定义
        volatile double serviceTime @unit(s);  // volatile参数

    gates:
        input in[];   // 输入门向量
        output out;   // 输出门（单一）
}
```

**示例：最简单的简单模块**

```ned
// 文件：samples/tictoc/tictoc1.ned
simple Txc1
{
    gates:
        input in;
        output out;
}
```

**示例：带参数的简单模块**

```ned
// 文件：samples/queueinglib/Queue.ned
simple Queue
{
    parameters:
        @group(Queueing);
        @display("i=block/activeq;q=queue");
        @signal[dropped](type="long");
        @signal[queueLength](type="long");
        @statistic[queueLength](title="queue length";record=vector,timeavg,max);

        int capacity = default(-1);  // -1表示无限制
        bool fifo = default(true);
        volatile double serviceTime @unit(s);

    gates:
        input in[];
        output out;
}
```

**C++类关联规则：**

```ned
// 默认：C++类名 = NED类型名
simple Queue { ... }  // → C++类 Queue

// 指定C++类名
simple MyQueue {
    @class(mylib::CustomQueue);  // → C++类 mylib::CustomQueue
}

// 包级namespace
@namespace(mylib);
simple Queue { ... }  // → C++类 mylib::Queue
```

### 4.2 复合模块 (Compound Module)

复合模块将多个子模块组合成更大的单元。它本身无行为逻辑，只定义结构。

**基本结构：**

```ned
module CompoundModule
{
    types:
        // 内部类型定义（可选）
        channel LocalChannel extends ned.DatarateChannel {
            datarate = 1Mbps;
        }

    parameters:
        int numNodes;
        @display("i=misc/node_vs");

    gates:
        inout port[];  // 双向门

    submodules:
        node[numNodes]: Node;  // 子模块向量
        router: Router {
            parameters:
                routingTable = "table.txt";  // 参数赋值
            gates:
                in[sizeof(port)];  // 设置门向量大小
        }

    connections:
        for i=0..numNodes-1 {
            node[i].port <--> LocalChannel <--> router.in[i];
        }
}
```

**示例：Phy复合模块**

```ned
// 文件：samples/wiredphy/Phy.ned
module Phy
{
    parameters:
        @display("i=block/rxtx;bgb=300,200");

    gates:
        input upperLayerIn;
        output upperLayerOut;
        input mediumIn;
        output mediumOut;
        input cutthroughIn @loose;   // 允许未连接
        output cutthroughOut @loose;

    submodules:
        rx: <> like IRx {  // parametric submodule type
            @display("p=200,100");
        }
        tx: <> like ITx {
            @display("p=100,100");
        }

    connections:
        upperLayerIn --> tx.upperLayerIn;
        upperLayerOut <-- rx.upperLayerOut;
        mediumIn --> rx.mediumIn;
        mediumOut <-- tx.mediumOut;
}
```

### 4.3 网络 (Network)

网络是特殊的复合模块，表示完整的仿真模型。使用`network`关键字声明。

```ned
// 文件：samples/tictoc/tictoc1.ned
network Tictoc1
{
    submodules:
        tic: Txc1;
        toc: Txc1;

    connections:
        tic.out --> { delay = 100ms; } --> toc.in;
        tic.in <-- { delay = 100ms; } <-- toc.out;
}

// 配置文件 omnetpp.ini:
[General]
network = Tictoc1
```

**示例：带内部类型定义的网络**

```ned
// 文件：samples/wiredphy/Network.ned
network Network
{
    types:
        channel Link extends ned.DatarateChannel {
            delay = 1ms;
        }

    submodules:
        source: SourceHost { @display("p=50,80"); }
        switch: SwitchNode { @display("p=150,80"); }
        sink: SinkHost { @display("p=250,80"); }

    connections:
        source.out --> Link --> switch.in1;
        switch.out1 --> Link --> source.in;
        switch.out2 --> Link --> sink.in;
        sink.out --> Link --> switch.in2;
}
```

---

## 5. 参数系统

### 5.1 参数类型

| 类型 | 关键字 | 说明 | 示例 |
|------|--------|------|------|
| 整数 | `int` | 整数值 | `int numJobs = 10;` |
| 浮点 | `double` | 浮点值 | `double delay @unit(s) = 1ms;` |
| 字符串 | `string` | 字符串 | `string protocol = "UDP";` |
| 布尔 | `bool` | 真值 | `bool fifo = true;` |
| XML | `xml` | XML文档 | `xml config = xmldoc("cfg.xml");` |
| 对象 | `object` | JSON对象 | `object routes = [{...}];` |

### 5.2 参数修饰符

**volatile修饰符：**

```ned
volatile double sendInterval @unit(s) = default(exponential(1s));

// volatile参数每次读取时重新计算表达式
// 用于随机值、时间相关值等
```

**@unit属性：**

```ned
double delay @unit(s);    // 单位：秒
int packetLength @unit(byte);  // 单位：字节
double bitrate @unit(bps);     // 单位：比特/秒

// 支持的单位：s, ms, us, ns, b, kb, Mb, Gb, byte, KiB, MiB, bps, kbps, Mbps, Gbps, m, km, cm, W, mW, kW, Hz, kHz, MHz, GHz 等
```

**@mutable属性：**

```ned
int maxLength @mutable;  // 运行时可修改

// 模块的C++实现必须支持handleParameterChange()
```

### 5.3 参数赋值方式

**1. NED文件内赋值：**

```ned
// 继承赋值
simple BoundedQueue extends Queue {
    capacity = 10;  // 固定值，不可覆盖
}

// 子模块赋值
module Host {
    submodules:
        queue: Queue {
            capacity = 5;  // 固定值
            serviceTime = default(100ms);  // 仅修改默认值
        }
}

// Pattern赋值（从父模块）
network Network {
    parameters:
        host[*].queue.capacity = default(10);  // 批量赋值
        host[0..9].queue.serviceTime = 50ms;

    submodules:
        host[100]: Host;
}
```

**2. 配置文件赋值：**

```ini
[General]
**.serviceTime = 100ms              # 全局赋值
Network.host[5].queue.capacity = 20 # 特定模块赋值
**.sendInterval = exponential(1s)   # 随机表达式
```

**3. 默认值定义：**

```ned
int capacity = default(-1);     // 默认值
int ttl = default;              // 使用类型默认值
int value = ask;                // 运行时询问用户
```

### 5.4 表达式语法

```ned
// 数值表达式
numNodes * 2 + 1
sizeof(port)
uniform(0, 10)
exponential(1s)

// 条件表达式
simTime() < 1000s ? 1s : 2s

// 字符串匹配
name =~ "P*"

// 运算符（部分与C不同）
^   // 幂运算（不是XOR）
#   // 二进制XOR
##  // 逻辑XOR
<=> // 三向比较（返回-1/0/1/nan）

// 特殊关键字
true, false, nan, inf, null, nullptr, undefined
```

### 5.5 Object参数（JSON风格）

```ned
// 数组
object array1 = [2, 5, 3, -1];
object array2 = [3, 24.5mW, "Hello", false];

// 对象
object obj = { foo: 100, bar: "Hello" };

// 复合结构（路由表）
object routes = [
    { dest: "10.0.0.0", netmask: "255.255.0.0", interf: "eth0", metric: 10 },
    { dest: "10.1.0.0", netmask: "255.255.0.0", interf: "eth1", metric: 20 },
    { dest: "*", interf: "eth2" }
];

// 类型化对象（C++类实例）
volatile object packetToSend = default(cPacket {
    name: "data",
    kind: 10,
    byteLength: intuniform(64, 4096)
});
```

---

## 6. 门(Gate)系统

### 6.1 门类型

| 类型 | 关键字 | 方向 | 用途 |
|------|--------|------|------|
| 输入 | `input` | 接收消息 | 从其他模块接收消息 |
| 输出 | `output` | 发送消息 | 向其他模块发送消息 |
| 双向 | `inout` | 收发 | 物理连接、双向通信 |

### 6.2 门声明

```ned
gates:
    input in;           // 单一输入门
    output out;         // 单一输出门
    inout port;         // 单一双向门

    input in[];         // 输入门向量（大小未定）
    output out[5];      // 输出门向量（大小=5）
    inout port[sizeof(neighbors)];  // 双向门向量（大小由参数决定）
```

### 6.3 门属性

```ned
gates:
    input radioIn @directIn;   // 用于sendDirect()
    inout port[] @loose;       // 允许未连接
```

**@directIn** - 用于直接消息发送（无线传输等）
**@loose** - 允许门未连接（边缘节点等）

### 6.4 门向量操作

```ned
// 声明时设置大小
output out[10];

// 继承时设置大小
simple BinaryTreeNode extends TreeNode {
    gates:
        children[2];  // 设置继承的门向量大小
}

// 子模块中设置大小
submodules:
    node: TreeNode {
        gates:
            children[2];
    }

// 动态扩展（++操作符）
node[i].port++  // 使用下一个可用门，若全部连接则扩展+1
```

---

## 7. 连接系统

### 7.1 连接语法

```ned
// 单向连接（输出→输入）
a.out --> b.in;
a.out --> { delay = 10ms; } --> b.in;
a.out --> ChannelType --> b.in;

// 单向连接（输入←输出）
b.in <-- a.out;

// 双向连接
a.port <--> b.port;
a.port <--> { datarate = 100Mbps; } <--> b.port;
a.port <--> ChannelType <--> b.port;

// 带名称的通道
a.out --> link1: EthernetChannel --> b.in;
```

### 7.2 内置通道类型

| 通道类型 | 包名 | 参数 | 说明 |
|----------|------|------|------|
| IdealChannel | `ned.IdealChannel` | 无 | 无延迟、无错误 |
| DelayChannel | `ned.DelayChannel` | `delay`, `disabled` | 固定延迟 |
| DatarateChannel | `ned.DatarateChannel` | `datarate`, `delay`, `ber`, `per` | 数据率、错误率 |

```ned
// 隐式通道类型（根据参数推断）
a.out --> { delay = 10ms; } --> b.in;  // → DelayChannel
a.out --> { datarate = 1Mbps; } --> b.in;  // → DatarateChannel
a.out --> { } --> b.in;  // → IdealChannel

// 自定义通道
channel Ethernet100 extends ned.DatarateChannel {
    datarate = 100Mbps;
    delay = 100us;
    ber = 1e-10;
}
```

### 7.3 连接循环与条件

```ned
// for循环连接
connections:
    for i=0..count-2 {
        node[i].port[1] <--> node[i+1].port[0];
    }

// 嵌套循环
for i=0..N-1, for j=0..N-1 {
    node[i].out[j] --> node[j].in[i] if i != j;
}

// 条件连接
node[i].left <--> node[2*i+1].parent if 2*i+1 < count;

// allowunconnected修饰符
connections allowunconnected:
    ...  // 允许部分门未连接
```

### 7.4 双向门的子门访问

```ned
// inout门包含两个子门：$i（输入侧）和$o（输出侧）
port$i    // 输入侧
port$o    // 输出侧
port$i[3] // 输入侧向量索引3
port$o++  // 输出侧扩展

// 可单独连接（不推荐）
a.port$o --> b.port$i;
a.port$i <-- b.port$o;
```

---

## 8. 继承与接口

### 8.1 模块继承

```ned
// 基础模块
simple Queue {
    int capacity;
    volatile double serviceTime @unit(s);
    gates:
        input in[];
        output out;
}

// 扩展模块（固定参数）
simple BoundedQueue extends Queue {
    capacity = 10;  // 锁定为10
}

// 扩展模块（更换C++类）
simple PriorityQueue extends Queue {
    @class(PriorityQueue);  // 必须指定新类名！
}
```

**继承规则：**
- 可添加新参数、门
- 可设置参数值、门向量大小
- Compound模块可添加子模块、连接
- 不可删除/修改继承的元素（除`@reconnect`外）

### 8.2 模块接口 (Module Interface)

```ned
// 接口定义
moduleinterface IRx {
    gates:
        output upperLayerOut;
        input mediumIn;
        output cutthroughOut @loose;
}

// 实现接口（simple module）
simple Rx like IRx {
    gates:
        output upperLayerOut;
        input mediumIn;
        output cutthroughOut @loose;
}

// parametric submodule type
module Phy {
    submodules:
        rx: <> like IRx;  // 类型由参数决定
}
```

**接口用途：**
- 作为子模块类型的占位符
- 实现可替换组件（如不同的移动模型）
- 通过`typename`参数确定具体类型

```ini
# omnetpp.ini中指定具体类型
**.rx.typename = "RxAtStart"
**.rx.typename = "RxAtEnd"
```

---

## 9. 属性(Properties)

### 9.1 属性语法

```ned
// 基本形式
@property(value)
@property[key1=value1; key2=value2]
@property[]  // 索引属性

// 位置
@display("i=block/queue");  // 模块属性
int capacity @unit(byte);   // 参数属性
input in @directIn;         // 门属性
a.out --> { @display("ls=red"); } --> b.in;  // 连接属性
```

### 9.2 常用属性

| 属性 | 位置 | 用途 |
|------|------|------|
| `@display` | 模块、门、连接 | 图形显示配置 |
| `@unit` | 参数 | 测量单位 |
| `@class` | 模块 | C++类名 |
| `@namespace` | 文件/包 | C++命名空间 |
| `@group` | 模块 | 模块分组（IDE中） |
| `@signal` | 模块 | 声明信号 |
| `@statistic` | 模块 | 声明统计 |
| `@mutable` | 参数 | 运行时可修改 |
| `@loose` | 门 | 允许未连接 |
| `@directIn` | 门 | 直接消息接收 |
| `@reconnect` | 连接 | 允许重连 |
| `@license` | 文件 | 许可证声明 |
| `@prompt` | 参数 | 交互式提示 |

### 9.3 显示字符串 (@display)

```ned
// 模块图标
@display("i=block/queue");  // 使用图标
@display("i=misc/node_vs,gold");  // 带颜色变体
@display("bgb=300,200");  // 背景大小

// 门显示
@display("p=100,100");  // 位置（子模块）

// 连接显示
@display("ls=red,2");  // 线条：红色、宽度2
@display("m=n");  // 线条方向标记

// 组合
@display("i=block/activeq;q=queue;bgb=300,200");
```

### 9.4 信号与统计

```ned
simple Queue {
    parameters:
        // 声明信号
        @signal[dropped](type="long");
        @signal[queueLength](type="long");
        @signal[queueingTime](type="simtime_t");

        // 声明统计
        @statistic[dropped](
            title="drop event";
            record=vector?,count;
            interpolationmode=none
        );
        @statistic[queueLength](
            title="queue length";
            record=vector,timeavg,max;
            interpolationmode=sample-hold
        );
}
```

---

## 10. 完整示例分析

### 10.1 TicToc基础示例

```ned
// samples/tictoc/tictoc1.ned
// 最简单的仿真模型

// 1. 简单模块定义
simple Txc1
{
    gates:
        input in;   // 接收消息的门
        output out; // 发送消息的门
}

// 2. 网络定义
network Tictoc1
{
    submodules:
        tic: Txc1;  // 创建Txc1实例，命名为tic
        toc: Txc1;  // 创建Txc1实例，命名为toc

    connections:
        // 单向连接，带延迟
        tic.out --> { delay = 100ms; } --> toc.in;
        tic.in <-- { delay = 100ms; } <-- toc.out;
}
```

**分析要点：**
- 简单模块只声明接口，行为在C++中
- 网络是模块实例的组合
- `{ delay = 100ms; }`创建隐式DelayChannel

### 10.2 队列库示例

```ned
// samples/queueinglib/Queue.ned
// 实际应用中的模块定义

package org.omnetpp.queueing;  // 包声明

simple Queue
{
    parameters:
        @group(Queueing);  // IDE分组
        @display("i=block/activeq;q=queue");  // 显示：图标+队列可视化

        // 信号声明（用于统计记录）
        @signal[dropped](type="long");
        @signal[queueLength](type="long");
        @signal[queueingTime](type="simtime_t");
        @signal[busy](type="bool");

        // 统计声明
        @statistic[dropped](
            title="drop event";
            record=vector?,count;
            interpolationmode=none
        );
        @statistic[queueLength](
            title="queue length";
            record=vector,timeavg,max;
            interpolationmode=sample-hold
        );

        // 参数定义
        int capacity = default(-1);  // -1表示无限
        bool fifo = default(true);
        volatile double serviceTime @unit(s);  // volatile+单位

    gates:
        input in[];  // 可连接多个上游模块
        output out;
}
```

**分析要点：**
- 包组织避免命名冲突
- `@signal/@statistic`声明统计系统
- `volatile`参数每次读取重新计算
- `@unit`确保单位一致性
- 门向量`in[]`允许多连接

### 10.3 模块向量与循环连接

```ned
// samples/tictoc/tictoc10.ned
// 模块向量示例

simple Txc10
{
    parameters:
        @display("i=block/routing");
    gates:
        input in[];   // 输入门向量
        output out[]; // 输出门向量
}

network Tictoc10
{
    submodules:
        tic[6]: Txc10;  // 创建6个Txc10实例

    connections:
        // ++操作符：使用下一个可用门
        tic[0].out++ --> { delay = 100ms; } --> tic[1].in++;
        tic[0].in++ <-- { delay = 100ms; } <-- tic[1].out++;

        tic[1].out++ --> { delay = 100ms; } --> tic[2].in++;
        tic[1].in++ <-- { delay = 100ms; } <-- tic[2].out++;

        // tic[1]有多个连接（门向量自动扩展）
        tic[1].out++ --> { delay = 100ms; } --> tic[4].in++;
        tic[1].in++ <-- { delay = 100ms; } <-- tic[4].out++;
}
```

**分析要点：**
- `tic[6]`创建6个实例的向量
- `out++`自动扩展门向量
- 同一模块可有多个连接（门向量）

### 10.4 二叉树拓扑

```ned
// samples/neddemo/BinaryTree.ned
// for循环+条件连接

simple BinaryTreeNode extends Node
{
    gates:
        inout toParent;
        inout leftChild;
        inout rightChild;
}

network BinaryTree1
{
    parameters:
        int height @prompt("Height of the tree") = default(5);

    submodules:
        node[2^height-1]: BinaryTreeNode;  // 2^height-1个节点

    connections allowunconnected:
        // 双重for循环 + 条件连接
        for i=0..2^height-2, for j=0..2^height-2 {
            node[i].leftChild <--> node[j].toParent
                if j == 2*i+1;  // 左子节点条件
            node[i].rightChild <--> node[j].toParent
                if j == 2*i+2;  // 右子节点条件
        }
}
```

**分析要点：**
- `extends`继承基础模块
- 模块数量用表达式：`2^height-1`
- `@prompt`交互式询问参数值
- 嵌套`for`循环遍历所有组合
- `if`条件筛选有效连接
- `allowunconnected`允许叶子节点门未连接

### 10.5 接口与Parametric Submodule

```ned
// samples/wiredphy/

// 接口定义
moduleinterface IRx {
    gates:
        output upperLayerOut;
        input mediumIn;
        output cutthroughOut @loose;
}

// 实现接口
simple Rx like IRx {
    parameters:
        double bitrate @unit(bps) = default(1Mbps);
    gates:
        output upperLayerOut;
        input mediumIn;
        output cutthroughOut @loose;
}

// 使用parametric type
module Phy {
    submodules:
        rx: <> like IRx {  // <>表示类型由参数决定
            @display("p=200,100");
        }
}

// 配置文件指定具体类型
// omnetpp.ini:
**.rx.typename = "RxAtStart"
```

**分析要点：**
- `moduleinterface`定义接口规范
- `like`关键字实现接口
- `<>`占位符（parametric submodule type）
- 具体类型通过ini文件的`typename`参数指定
- 实现可替换组件的设计模式

---

## 11. 最佳实践

### 11.1 文件组织

```
推荐结构：
project/
├── package.ned           # 包定义 + @namespace
├── src/
│   ├── Node.ned          # 简单模块
│   ├── Queue.ned
│   ├── Host.ned          # 复合模块
│   └── Network.ned       # 网络
│   ├── Node.cc           # C++实现
│   └── Queue.cc
└── simulations/
    └── omnetpp.ini       # 配置文件
```

### 11.2 命名约定

| 元素 | 规则 | 示例 |
|------|------|------|
| 模块类型 | 大驼峰，首字母大写 | `Queue`, `MobileHost` |
| 参数 | 小驼峰，首字母小写 | `capacity`, `serviceTime` |
| 门 | 小驼峰，描述性 | `in`, `out`, `upperLayerIn` |
| 子模块 | 小驼峰 | `routing`, `phy` |
| 文件 | 与模块名相同 | `Queue.ned` |

### 11.3 参数设计原则

```ned
// ✅ 推荐：明确单位
double delay @unit(s) = default(1ms);

// ✅ 推荐：volatile用于随机值
volatile double sendInterval @unit(s) = default(exponential(1s));

// ✅ 推荐：default()提供默认值
int capacity = default(-1);

// ❌ 避免：无单位的数值
double delay = 1;  // 什么单位？

// ❌ 避免：volatile用于常量
volatile int constantValue = 10;  // 无意义
```

### 11.4 模块设计建议

**简单模块：**
- 保持单一职责
- 参数控制在合理数量（<20）
- 为所有参数提供默认值或文档
- 声明必要的信号和统计

**复合模块：**
- 合理分层，避免过度嵌套（<5层）
- 使用内部类型避免命名冲突
- 用`for`循环处理规则拓扑
- 用`allowunconnected`处理边界情况

**网络：**
- 作为顶层模型
- 提供参数化拓扑能力
- 考虑多配置场景

### 11.5 调试技巧

```ned
// 未连接门检查
connections allowunconnected:  // 暂时放宽检查

// 门标记
input debugIn @loose;  // 允许未连接用于调试

// 条件连接调试
node[i].out --> node[j].in
    if i != j && uniform(0,1) < connectedness;  // 随机拓扑

// 参数验证
int capacity >= 0;  // 未来版本支持约束表达式
```

---

## 附录A：NED关键字完整列表

```
package, import
simple, module, network, channel
moduleinterface, channelinterface
extends, like
parameters, gates, types, submodules, connections
input, output, inout
volatile, default, ask
allowunconnected
for, if
true, false, nan, inf, null, nullptr, undefined
sizeof, index, exists, typename
this, parent
```

## 附录B：内置函数参考

见 `doc/src/manual/appendix-ned-functions.tex`，包括：
- 数值函数：`abs`, `fabs`, `floor`, `ceil`, `round`, `sin`, `cos`, `exp`, `log`, `pow`, `sqrt`
- 随机函数：`uniform`, `exponential`, `normal`, `lognormal`, `intuniform`, `bernoulli`, `poisson`
- 字符串函数：`length`, `substr`, `strcmp`, `strstr`, `replace`, `parse`
- 时间函数：`simTime`
- 其他：`sizeof`, `index`, `exists`, `typename`, `fullname`, `fullpath`, `parentIndex`

## 附录C：单位完整列表

见 `doc/src/manual/appendix-ned-ref.tex`：

**时间：** `s, ms, us, ns, ps, min, h, d, wk`
**数据：** `b, kb, Mb, Gb, Tb, byte, KiB, MiB, GiB, TiB`
**速率：** `bps, kbps, Mbps, Gbps, Tbps`
**距离：** `m, cm, mm, km, ft, in, mi`
**功率：** `W, mW, kW, MW, dBm, dBW`
**频率：** `Hz, kHz, MHz, GHz`
**其他：** `kg, g, V, A, K, C, F, J, N, Pa, ohm, mol, cd`

---

## 参考资料

1. OMNeT++ Manual - Chapter: The NED Language (`doc/src/manual/ch-ned-lang.tex`)
2. NED Language Grammar (`doc/src/manual/appendix-ned-grammar.tex`)
3. NED Functions (`doc/src/manual/appendix-ned-functions.tex`)
4. NED Reference (`doc/src/manual/appendix-ned-ref.tex`)
5. 示例代码 (`samples/`目录)

---

**文档状态：草案 - 待补充**
- 待补充：nedxml解析器实现细节
- 待补充：更多实际案例（INET Framework等）
- 待补充：IDE编辑器使用细节
- 待补充：NED文件验证与调试技巧

**建议下一步：**
1. 补充nedxml解析器结构分析
2. 收集INET Framework NED示例
3. 添加可视化编辑章节
4. 完善错误处理与调试内容