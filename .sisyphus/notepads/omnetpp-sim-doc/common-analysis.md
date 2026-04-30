# OMNeT++ src/common/ 子系统架构分析

## 1. 概述

`src/common/` 是 OMNeT++ 的基础共享库，编译为 `liboppcommon`（约 11MB），被所有其他子系统依赖。该库提供通用工具函数、表达式解析引擎、模式匹配、文件 I/O、输出写入器等核心功能。

### 1.1 文件统计

| 类别 | 头文件 | 实现文件 | 说明 |
|------|--------|----------|------|
| 表达式引擎 | 8 | 7 | expression.*, exprnode*, exprvalue.*, matchexpression.* |
| 字符串处理 | 4 | 4 | stringutil.*, stringtokenizer.*, stringpool.*, pooledstring.* |
| 文件 I/O | 5 | 5 | fileutil.*, filereader.*, fileglobber.*, filelock.*, fnamelisttokenizer.* |
| 输出写入器 | 8 | 8 | jsonwriter.*, csvwriter.*, sqlite*writer.*, omnetpp*writer.* |
| XML 解析 | 5 | 3 | saxparser*.* |
| 数值类型 | 3 | 3 | bigdecimal.*, statistics.*, unitconversion.* |
| 异常/工具 | 其余 | 其余 | exception.*, commonutil.*, intutil.*, lcgrandom.* 等 |
| 第三方 | 2 | 2 | sqlite3.*, yxml.* |

**总计**：57 个头文件，45 个实现文件，约 110 个文件（含第三方库）。

## 2. 工具类别分类

### 2.1 字符串处理

| 文件 | 核心类/函数 | 功能 |
|------|-------------|------|
| `stringutil.h/cc` | `opp_*` 函数族 | 字符串操作核心工具（60+ 函数） |
| `stringtokenizer.h/cc` | `StringTokenizer` | 字符串分词器 |
| `stringpool.h/cc` | `StringPool` | 字符串池（内存优化） |
| `pooledstring.h/cc` | `opp_staticpooledstring` | 静态池化字符串 |

**关键函数**（stringutil.h）：
- `opp_isempty()`, `opp_streq()`, `opp_strcmp()` — 安全字符串检查/比较
- `opp_strdup()`, `opp_strndup()` — 字符串复制
- `opp_trim()`, `opp_split()`, `opp_join()` — 字符串处理
- `opp_quotestr()`, `opp_parsequotedstr()` — 引号处理
- `opp_stringf()`, `opp_vstringf()` — 格式化字符串
- `opp_atol()`, `opp_atof()`, `opp_strtod()` — 安全数值转换
- `opp_xmlquote()`, `opp_latexquote()` — 特殊格式转义

### 2.2 表达式解析引擎

表达式引擎是 common 库最复杂的子系统，采用经典的三阶段架构：

```
源代码 → 词法分析 → 语法分析(AST) → 语义翻译 → 表达式树 → 求值
```

#### 2.2.1 核心组件

| 文件 | 核心类 | 功能 |
|------|--------|------|
| `expression.y` | Bison 文法 | 语法定义（运算符优先级、AST 构建） |
| `expression.lex` | Flex 词法 | 词法分析（token 识别） |
| `expression.h/cc` | `Expression` | 主接口类 |
| `exprvalue.h/cc` | `ExprValue` | 表达式值类型（动态类型） |
| `exprnode.h/cc` | `ExprNode` | 表达式树节点基类 |
| `exprnodes.h/cc` | 各种节点类 | 具体节点实现（运算符、函数、变量） |

#### 2.2.2 架构层次

**第 1 层：词法分析**
- 输入：字符流
- 输出：token 流（NAME, INTCONSTANT, REALCONSTANT, STRINGCONSTANT 等）
- 文件：`expression.lex` → `expression.lex.cc`

**第 2 层：语法分析**
- 输入：token 流
- 输出：抽象语法树（AST）
- 文件：`expression.y` → `expression.tab.cc`
- AST 节点类型：`AstNode::Type`（CONSTANT, OP, IDENT, FUNCTION, MEMBER, METHOD, OBJECT, ARRAY 等）

**第 3 层：语义翻译**
- 输入：AST
- 输出：表达式求值树（`ExprNode` 树）
- 关键类：`AstTranslator`, `BasicAstTranslator`, `MultiAstTranslator`

**第 4 层：求值执行**
- 输入：`ExprNode` 树 + `Context`
- 输出：`ExprValue`
- 支持动态解析：`DynamicResolver` 接口

#### 2.2.3 ExprValue 类型系统

```cpp
enum Type { UNDEF=0, BOOL='B', INT='L', DOUBLE='D', STRING='S', POINTER='O' };
```

- 支持带单位的数值：`setQuantity(double, const char* unit)`
- 支持指针类型（`any_ptr`）用于 cObject
- 自动类型转换和单位转换

#### 2.2.4 运算符优先级（从低到高）

| 优先级 | 运算符 |
|--------|--------|
| IIF (16) | `?:` |
| LOGICAL_OR (15) | `\|\|` |
| LOGICAL_XOR (14) | `##` |
| LOGICAL_AND (13) | `&&` |
| RELATIONAL_EQ (12) | `==`, `!=` |
| RELATIONAL_LT (11) | `<`, `<=`, `>`, `>=` |
| MATCH (10) | `=~` |
| BITWISE_OR (9) | `\|` |
| BITWISE_XOR (8) | `#` |
| BITWISE_AND (7) | `&` |
| SHIFT (6) | `<<`, `>>` |
| ADDSUB (5) | `+`, `-` |
| MULDIV (4) | `*`, `/`, `%` |
| POW (3) | `^` |
| UNARY (2) | `-`, `~`, `!` |
| MEMBER (1) | `.` |
| ELEM (0) | 常量、变量、函数 |

### 2.3 模式匹配

| 文件 | 核心类 | 功能 |
|------|--------|------|
| `patternmatcher.h/cc` | `PatternMatcher` | Glob 风格模式匹配 |
| `matchexpression.h/cc` | `MatchExpression` | 复杂布尔表达式匹配 |

#### PatternMatcher 语法

- `?` — 匹配任意非 `.` 字符
- `*` — 匹配零或多个非 `.` 字符
- `**` — 匹配零或多个任意字符
- `{a-z}` — 字符范围
- `{^a-z}` — 排除字符范围
- `{32..255}` — 数值范围
- `[32..255]` — 方括号数值范围

#### MatchExpression 语法

- 支持 AND、OR、NOT 逻辑运算
- 支持 `<field> =~ <pattern>` 字段匹配
- 示例：`"node* or host* or className =~ StandardHost*"`

### 2.4 文件 I/O

| 文件 | 核心类/函数 | 功能 |
|------|-------------|------|
| `fileutil.h/cc` | 全局函数 | 文件路径操作 |
| `filereader.h/cc` | `FileReader` | 高效大文件行读取 |
| `fileglobber.h/cc` | `FileGlobber` | 文件名 glob 匹配 |
| `filelock.h/cc` | `FileLock` | 文件锁 |
| `linetokenizer.h/cc` | `LineTokenizer` | 行分词器 |

#### FileReader 特性

- 支持大文件（GB 级别）
- 缓冲区管理，避免频繁 I/O
- 支持正向/反向遍历
- 检测文件变化（追加/覆盖）
- 支持文件锁定

### 2.5 输出写入器

| 文件 | 核心类 | 输出格式 | 用途 |
|------|--------|----------|------|
| `jsonwriter.h/cc` | `JsonWriter` | JSON | 通用数据导出 |
| `csvwriter.h/cc` | `CsvWriter` | CSV (RFC 4180) | 表格数据导出 |
| `omnetppscalarfilewriter.h/cc` | `OmnetppScalarFileWriter` | OMNeT++ 标量文件 | 仿真结果输出 |
| `omnetppvectorfilewriter.h/cc` | `OmnetppVectorFileWriter` | OMNeT++ 向量文件 | 仿真结果输出 |
| `sqlitescalarfilewriter.h/cc` | `SqliteScalarFileWriter` | SQLite 标量表 | 数据库结果存储 |
| `sqlitevectorfilewriter.h/cc` | `SqliteVectorFileWriter` | SQLite 向量表 | 数据库结果存储 |

#### 写入器架构模式

```
Writer 基类
├── open(filename) / close()
├── beginRecordingForRun() / endRecordingForRun()
├── record*() 系列方法
└── flush()
```

### 2.6 XML 解析

| 文件 | 核心类 | 底层实现 |
|------|--------|----------|
| `saxparser.h` | `SaxParser`, `SaxHandler` | 抽象接口 |
| `saxparser_libxml.h/cc` | `LibxmlSaxParser` | LibXML2 |
| `saxparser_yxml.h/cc` | `YxmlSaxParser` | YXML（轻量） |
| `saxparser_default.h/cc` | 默认实现 | 条件编译选择 |

**SAX 事件模型**：
- `startElement(name, atts)`
- `endElement(name)`
- `characterData(s, len)`
- `processingInstruction(target, data)`

### 2.7 数值与统计

| 文件 | 核心类 | 功能 |
|------|--------|------|
| `bigdecimal.h/cc` | `BigDecimal` | 高精度十进制数 |
| `statistics.h/cc` | `Statistics` | 描述性统计（加权/非加权） |
| `unitconversion.h/cc` | `UnitConversion` | 单位转换工具 |
| `quantityformatter.h/cc` | `QuantityFormatter` | 量值格式化 |
| `intutil.h/cc` | 工具函数 | 整数操作工具 |

#### UnitConversion 功能

- 内置物理单位表（时间、功率、频率、带宽等）
- 支持线性/对数单位（W vs dBW）
- `parseQuantity()` — 解析带单位数值
- `convertUnit()` — 单位转换
- `getBestUnit()` — 智能单位选择

#### Statistics 支持的统计量

- `count`, `min`, `max`
- `sumWeights`, `sumWeightedValues`
- `mean`, `variance`, `stddev`

### 2.8 其他工具类

| 文件 | 核心类/函数 | 功能 |
|------|-------------|------|
| `exception.h/cc` | `opp_runtime_error` | 通用异常类 |
| `commonutil.h/cc` | `Assert()`, 工具函数 | 通用工具 |
| `lcgrandom.h/cc` | `LCGRandom` | 线性同余随机数生成器 |
| `colorutil.h/cc` | 颜色工具 | 颜色解析/转换 |
| `enumstr.h/cc` | 枚举字符串工具 | 枚举与字符串映射 |
| `rwlock.h/cc` | `RWLock` | 读写锁 |
| `histogram.h` | `Histogram` | 直方图数据结构 |
| `any_ptr.h/cc` | `any_ptr` | 类型安全指针包装 |
| `formattedprinter.h/cc` | `FormattedPrinter` | 格式化输出 |
| `progressmonitor.h` | `ProgressMonitor` | 进度监控接口 |
| `backward.h/cc` | 堆栈追踪 | 调试支持 |

## 3. 关键函数列表

### 3.1 stringutil.h 关键函数

```cpp
// 字符串检查
bool opp_isempty(const char *s);
bool opp_isblank(const char *txt);
bool opp_streq(const char *s1, const char *s2);
int opp_strcmp(const char *s1, const char *s2);
bool opp_stringbeginswith(const char *s, const char *prefix);
bool opp_stringendswith(const char *s, const char *ending);

// 字符串操作
char *opp_strdup(const char *s);
char *opp_strndup(const char *s, int len);
std::string opp_trim(const std::string& text);
std::string opp_quotestr(const std::string& txt, char quot='"');
std::string opp_parsequotedstr(const char *txt, char quot='"');
std::string opp_replacesubstring(const std::string& text, ...);
std::vector<std::string> opp_split(const std::string& text, const std::string& separator);
std::string opp_join(const std::vector<std::string>& strings, const char *separator);

// 格式化
std::string opp_stringf(const char *fmt, ...);
std::string opp_abbreviate(const std::string& text, int maxlen);

// 数值转换（带溢出检查）
long opp_atol(const char *s);
double opp_atof(const char *s);
long long opp_atoll(const char *s);

// 特殊格式
std::string opp_xmlquote(const std::string& str);
std::string opp_latexquote(const std::string& str);
```

### 3.2 Expression 关键方法

```cpp
class Expression {
    // 解析
    Expression& parse(const char *text, AstTranslator *translator=nullptr);
    AstNode *parseToAst(const char *text) const;
    ExprNode *translateToExpressionTree(AstNode *astTree, AstTranslator *translator) const;
    
    // 求值
    ExprValue evaluate(Context* context = nullptr) const;
    bool boolValue(Context *context=nullptr) const;
    intval_t intValue(Context *context, const char *expectedUnit=nullptr) const;
    double doubleValue(Context *context, const char *expectedUnit=nullptr) const;
    std::string stringValue(Context *context=nullptr) const;
    
    // 动态解析
    void addDynamicResolver(DynamicResolver *r);
    
    // AST 翻译器
    static void installAstTranslator(AstTranslator *translator);
};
```

### 3.3 UnitConversion 关键方法

```cpp
class UnitConversion {
    static double parseQuantity(const char *str, const char *expectedUnit=nullptr);
    static double convertUnit(double d, const char *unit, const char *targetUnit);
    static const char *getBestUnit(double d, const char *unit);
    static bool isKnownUnit(const char *unit);
    static bool areCompatibleUnits(const char *unit1, const char *unit2);
    static std::vector<const char *> getKnownUnits();
};
```

### 3.4 PatternMatcher 关键方法

```cpp
class PatternMatcher {
    void setPattern(const char *pattern, bool dottedpath, bool fullstring, bool casesensitive);
    bool matches(const char *line) const;
    bool covers(const char *pattern) const;
    static bool containsWildcards(const char *pattern);
};
```

### 3.5 FileReader 关键方法

```cpp
class FileReader {
    FileReader(const char *fileName, size_t bufferSize = 256 * 1024);
    char *getFirstLineBufferPointer();
    char *getLastLineBufferPointer();
    char *getNextLineBufferPointer();
    char *getPreviousLineBufferPointer();
    void seekTo(file_offset_t offset, size_t ensureBufferSizeAround = 0);
    FileChange getFileChange();
    void synchronize(FileChange change);
};
```

## 4. 依赖关系

```
common（本库）
    │
    ├── 被依赖者（所有其他子系统）
    │   ├── sim/      ← 核心仿真内核
    │   ├── nedxml/   ← NED/MSG 编译器
    │   ├── scave/    ← 结果分析
    │   ├── eventlog/ ← 事件日志
    │   ├── envir/    ← 运行环境
    │   ├── qtenv/    ← Qt GUI
    │   ├── cmdenv/   ← 命令行界面
    │   └── layout/   ← 图布局
    │
    └── 外部依赖
        ├── sqlite3   ← SQLite 数据库（内置）
        ├── yxml      ← XML 解析器（内置）
        └── platdep/  ← 平台相关定义
```

## 5. 命名规范

### 5.1 函数命名

- 全局工具函数：`opp_` 前缀（如 `opp_streq`, `opp_strdup`）
- 类方法：camelCase（如 `evaluate`, `parseQuantity`）

### 5.2 类命名

- 类：PascalCase（如 `Expression`, `PatternMatcher`）
- 节点类：`XxxNode` 后缀（如 `ConstantNode`, `AddNode`）

### 5.3 命名空间

```cpp
namespace omnetpp {
namespace common {
    // 所有代码在此
}
}
```

## 6. 第三方库

| 库 | 文件 | 功能 | 许可证 |
|----|------|------|--------|
| SQLite | `sqlite3.h/c` | 嵌入式数据库 | Public Domain |
| YXML | `yxml.h/c` | 轻量 XML 解析器 | MIT |

**注意**：分析时应跳过这些第三方库的内部实现，仅关注 OMNeT++ 对其的封装接口。

## 7. 架构特点总结

1. **高度模块化**：各功能模块独立，低耦合
2. **表达式引擎灵活**：支持 AST 翻译器扩展、动态解析
3. **性能优化**：字符串池、大文件缓冲读取、批量写入
4. **类型安全**：`ExprValue` 动态类型系统，`any_ptr` 类型安全指针
5. **单位支持**：表达式支持物理单位，自动转换
6. **多后端**：XML 解析支持 libxml2 和 yxml 两种后端
7. **多输出格式**：支持文本、JSON、CSV、SQLite 等多种输出格式