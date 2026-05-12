# 表达式编译器设计详解

> 聚焦表达式解析与求值子系统。

## 一、整体架构：三阶段流水线

表达式系统位于 `src/common/` 子系统（库 `oppcommon`），采用**经典编译器三阶段**架构：

```
源码字符串
    │
    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐
│  词法分析     │───▶│  语法分析     │───▶│  AST → ExprNode 树   │
│ (Flex)       │    │ (Bison)       │    │ (AstTranslator)      │
│ expression.lex│    │ expression.y  │    │ expression.cc        │
└──────────────┘    └──────────────┘    └──────────────────────┘
                                                │
                                                ▼
                                       ┌──────────────────────┐
                                       │  常量折叠优化          │
                                       │ (constant folding)   │
                                       └──────────────────────┘
                                                │
                                                ▼
                                         ExprNode 求值树
                                         （可执行结构）
```

关键入口方法：`Expression::parse(text, translator)`，内部依次调用：

```cpp
Expression& Expression::parse(const char *expr, AstTranslator *translator) {
    setExpressionTree(parseAndTranslate(expr, translator));
    return *this;
}

ExprNode *Expression::parseAndTranslate(const char *expr, AstTranslator *translator) {
    if (!translator) translator = &defaultTranslator;
    std::unique_ptr<AstNode> astTree(parseToAst(expr));          // 阶段1+2
    ExprNode *exprTree = translateToExpressionTree(astTree.get(), translator); // 阶段3
    exprTree = performConstantFolding(exprTree);                   // 优化
    return exprTree;
}
```

## 二、阶段一：词法分析（Flex）

**源文件**：`src/common/expression.lex`

词法分析器由 Flex 生成，是**可重入的** (`%option reentrant`)，通过 `yyscan_t` 参数支持多实例并发。

### Token 类型

| Token | 含义 | 示例 |
|-------|------|------|
| `TRUE_`, `FALSE_`, `NAN_`, `INF_` | 布尔/特殊常量 | `true`, `nan`, `inf` |
| `UNDEFINED_`, `NULLPTR_`, `NULL_` | 空值常量 | `undefined`, `nullptr` |
| `NAME` | 标识符（含 `::` 命名空间） | `foo`, `ns::bar` |
| `INTCONSTANT` | 整数字面量 | `42`, `0xFF` |
| `REALCONSTANT` | 浮点字面量 | `3.14`, `1e-5` |
| `STRINGCONSTANT` | 字符串字面量 | `"hello"` |
| `EQ`(`==`), `NE`(`!=`), `LE`(`<=`), `GE`(`>=`) | 比较运算符 | |
| `AND`(`&&`), `OR`(`||`), `XOR`(`##`) | 逻辑运算符 | |
| `SPACESHIP`(`<=>`), `MATCH`(`=~`) | 三路比较/模式匹配 | |
| `SHIFT_LEFT`(`<<`), `SHIFT_RIGHT`(`>>`) | 移位运算符 | |
| `DOUBLECOLON`(`::`) | 命名空间分隔 | |

**特点**：
- 标识符规则：`{L}({L}|{D})*(:{L}({L}|{D})*)*`，支持 `A::B::C` 形式的限定名
- 单位量词直接在词法层面拼接，如 `5s 230ms` 被识别为 `quantity` token
- 单行注释 `//` 被词法层跳过（兼容 NED 表达式）

## 三、阶段二：语法分析（Bison）

**源文件**：`src/common/expression.y`

Bison 语法文件，输出为 `expression.tab.c` / `expression.tab.h`。分析结果为 `AstNode` 树。

### AST 节点定义 (`Expression::AstNode`)

```cpp
struct COMMON_API AstNode {
    enum Type {
        UNDEF,           // 未定义
        CONSTANT,        // 常量（bool/int/double/string/pointer/quantity）
        OP,              // 运算符（一元/二元/三元）
        IDENT,           // 标识符：variable
        IDENT_W_INDEX,   // 带索引标识符：variable[index]
        FUNCTION,        // 函数调用：f(a,b,c)
        MEMBER,          // 成员访问：obj.member
        MEMBER_W_INDEX,  // 带索引成员：obj.member[index]
        METHOD,          // 方法调用：obj.method(a,b)
        OBJECT,          // 对象字面量：TypeName{key: value, ...}
        KEYVALUE,        // 键值对（OBJECT 内部）
        ARRAY            // 数组字面量：[a, b, c]
    };
    Type type = UNDEF;
    ExprValue constant;          // CONSTANT 节点的值
    std::string name;            // 运算符名/标识符名/函数名
    std::vector<AstNode*> children; // 子节点
    bool parenthesized = false;  // 括号标记（用于反序列化 unparse）
};
```

### 语法规则要点

| 规则 | AST 节点类型 | 示例 |
|------|-------------|------|
| `expr '+' expr` | `OP("+", child1, child2)` | `a + b` |
| `'-' expr` (UMIN_) | `OP("-", child)` 或原地取反 | `-5` → `CONSTANT(-5)` |
| `NAME '(' opt_exprlist ')'` | `FUNCTION(name, args...)` | `sqrt(2)` |
| `expr '.' NAME '(' opt_exprlist ')'` | `METHOD(name, obj, args...)` | `a.size()` |
| `expr '.' NAME` | `MEMBER(name, obj)` | `a.length` |
| `NAME '[' expr ']'` | `IDENT_W_INDEX(name, idx)` | `hosts[3]` |
| `qname '{' opt_keyvaluelist '}'` | `OBJECT(typeName, kvPairs)` | `Xml{key: val}` |
| `'[' exprlist ']'` | `ARRAY(elems...)` | `[1, 2, 3]` |
| `expr '?' expr ':' expr` | `OP("?:", cond, then, else)` | `x > 0 ? x : -x` |

### 运算符优先级（从低到高）

```
?:  (右结合)
||  (左结合)
##
&&
== !=
< > <= <= <=>
=~
|  #
&  ~
<< >>
+ -
* / %
^   (右结合)
- ~ !  (一元，右结合)
!   (后缀)
.
```

**特殊优化**：一元减号作用于数字常量时，**不生成运算节点**，而是直接修改常量值：

```cpp
// expression.y, production: '-' expr %prec UMIN_
if (arg->type == AstNode::CONSTANT && arg->constant.getType() == ExprValue::DOUBLE) {
    arg->constant.setPreservingUnit(-arg->constant.doubleValue());
    $$ = $2; // 直接返回常量节点
}
```

## 四、阶段三：AST → ExprNode 树翻译

### 翻译器架构（策略模式 + 责任链）

```
                ┌──────────────────────┐
                │ MultiAstTranslator   │
                │  (责任链调度器)        │
                └─────────┬────────────┘
                          │ 顺序尝试
            ┌─────────────┼─────────────────┐
            ▼             ▼                 ▼
  OperatorAstTranslator  StdMathAstTranslator  ...
  (运算符)              (sin,cos等)             │
                                                 ▼
                                    UnresolvedNameAstTranslator
                                    (变量/函数/方法动态解析)
```

`MultiAstTranslator` 持有一组 `AstTranslator`，按顺序尝试翻译每个 AST 节点，第一个成功即返回：

```cpp
ExprNode *MultiAstTranslator::translateToExpressionTree(AstNode *astNode, AstTranslator *translatorForChildren) {
    for (AstTranslator *translator : translators)
        if (ExprNode *node = translator->translateToExpressionTree(astNode, translatorForChildren))
            return node;
    return nullptr; // 全部失败则报错
}
```

### 默认翻译器链（4 个，按序）

| # | 翻译器 | 职责 | 创建的 ExprNode |
|---|--------|------|------------------|
| 1 | `OperatorAstTranslator` | 运算符 → 对应节点 | `AddNode`, `MulNode`, `EqualNode` 等 |
| 2 | `StdMathAstTranslator` | 标准数学函数 | `MathFunc0/1/2/3/4Node` |
| 3 | `UnitConversionAstTranslator` | 单位转换函数 | `UnitConversionNode` |
| 4 | `UnresolvedNameAstTranslator` | 未解析名 → 动态解析节点 | `DynamicallyResolvedVariableNode/FunctionNode/MethodNode` |

### `BasicAstTranslator` 翻译流程

核心方法 `translateToExpressionTree()` 根据 AST 节点类型分发：

```cpp
ExprNode *BasicAstTranslator::translateToExpressionTree(AstNode *astNode, AstTranslator *translatorForChildren) {
    switch (astNode->type) {
    case AstNode::CONSTANT:   return new ConstantNode(astNode->constant);
    case AstNode::OP:         return createOperatorNode(name, argCount);
    case AstNode::FUNCTION:   return createFunctionNode(name, argCount);
    case AstNode::METHOD:     return createMethodNode(name, argCount);
    case AstNode::IDENT/IDENT_W_INDEX: return createIdentNode(name, withIndex);
    case AstNode::MEMBER/MEMBER_W_INDEX: return createMemberNode(name, withIndex);
    case AstNode::OBJECT:     return createObjectNode(name, keys);
    case AstNode::ARRAY:      return createArrayNode(argCount);
    }
}
```

每个 `createXxxNode()` 返回 `nullptr` 表示"无法处理"，交由下一个翻译器。

## 五、ExprNode 求值树（运行时表示）

AST 经过翻译后，变成一棵**可自求值的树**。节点继承体系：

```
ExprNode                     (抽象基类: evaluate(), getName(), getPrecedence())
├── LeafNode                 (无子节点)
│   ├── ConstantNode         (保存 ExprValue 常量)
│   ├── MathFunc0Node        (零参数数学函数)
│   └── ValueNode            (变量抽象基类)
│       ├── VariableNode     (抽象: getValue(context))
│       ├── LambdaVariableNode   (lambda 提供变量值)
│       └── DynamicallyResolvedVariableNode  (运行时由 DynamicResolver 提供)
├── UnaryNode                (1 个子节点)
│   ├── NegateNode           (-)
│   ├── NotNode              (!)
│   ├── BitwiseNotNode       (~)
│   ├── IntCastNode          (int())
│   ├── DoubleCastNode       (double())
│   ├── UnitConversionNode   (单位转换)
│   ├── MathFunc1Node        (一元数学函数)
│   ├── IndexedVariableNode  (arr[i])
│   ├── MemberNode           (obj.field)
│   └── ...
├── BinaryNode               (2 个子节点)
│   ├── AddNode, SubNode, MulNode, DivNode, ModNode
│   ├── PowNode              (^)
│   ├── CompareNode系列      (==, !=, <, >, <=, >=)
│   ├── ThreeWayComparisonNode (<=>)
│   ├── MatchNode             (=~)
│   ├── ShiftNodes           (<<, >>)
│   ├── LogicalInfixNodes    (&&, ||, ##)
│   └── BitwiseInfixNodes    (&, |, #)
├── TernaryNode              (3 个子节点)
│   ├── InlineIfNode         (?:)
│   └── MathFunc3Node
├── NaryNode                 (N 个子节点)
│   ├── FunctionNode         (函数调用, N 个参数)
│   ├── MethodNode           (方法调用, object + N 个参数)
│   ├── ObjectNode           (对象字面量)
│   ├── ArrayNode            (数组字面量)
│   ├── MathFunc4Node
│   └── DynamicallyResolvedXxxNode 系列
```

每个节点通过 `evaluate(Context*)` 执行求值，`Context` 携带运行时信息：

```cpp
struct Context {
    const Expression *expression = nullptr;
    cObject *simContext = nullptr;  // 仿真上下文（模块、参数等）
};
```

### 优先级枚举 (`ExprNode::Precedence`)

用于 `str()` 反序列化和括号控制：

```cpp
enum Precedence {
    ELEM = 0,           // 常量/变量/函数
    MEMBER = 1,         // . (方法/成员)
    UNARY = 2,          // - ~ !
    POW = 3,            // ^
    MULDIV = 4,         // * / %
    ADDSUB = 5,         // + -
    SHIFT = 6,          // << >>
    BITWISE_AND = 7,    // &
    BITWISE_XOR = 8,    // #
    BITWISE_OR = 9,     // |
    MATCH = 10,         // =~
    RELATIONAL_LT = 11, // < <= > >=
    RELATIONAL_EQ = 12, // == !=
    LOGICAL_AND = 13,   // &&
    LOGICAL_XOR = 14,   // ##
    LOGICAL_OR = 15,    // ||
    IIF = 16,           // ?:
};
```

## 六、常量折叠优化

`parseAndTranslate()` 最后一步做常量折叠：

```cpp
Expression::performConstantFolding(exprTree)
```

核心逻辑：递归检查子树，如果所有子节点都是可折叠的（常量 + 算术运算符 + 标准数学函数），则在编译期直接计算结果，替换为 `ConstantNode`：

```cpp
bool Expression::isFoldableNode(ExprNode *node) const {
    return
        dynamic_cast<ConstantNode*>(node) ||
        dynamic_cast<UnaryOperatorNode*>(node) ||
        dynamic_cast<BinaryOperatorNode*>(node) ||
        dynamic_cast<TernaryOperatorNode*>(node) ||
        dynamic_cast<DoubleCastNode*>(node) ||
        dynamic_cast<IntCastNode*>(node) ||
        (ExprNodeFactory::supportsStdMathFunction(node->getName()) &&
            (dynamic_cast<MathFunc1/2/3Node*>(node)));
}
```

例如：`"2+3*4"` → 编译时计算为 `ConstantNode(14)`。

折叠算法分两步：
1. `findFoldableSubtrees()` — 自底向上标记可折叠子树
2. `foldSubtrees()` — 尝试对标记子树调用 `tryEvaluate(nullptr)` 求值，成功则替换为 `ConstantNode`

## 七、动态解析机制

对于变量、函数、方法等**无法在编译期确定**的符号，`Expression` 类提供了运行时解析接口：

### `Expression::DynamicResolver`（common 层）

```cpp
class DynamicResolver {
public:
    virtual ExprValue readVariable(Context *ctx, const char *name);
    virtual ExprValue readVariable(Context *ctx, const char *name, intval_t index);
    virtual ExprValue callFunction(Context *ctx, const char *name, ExprValue argv[], int argc);
    virtual ExprValue readMember(Context *ctx, const ExprValue& obj, const char *name);
    virtual ExprValue callMethod(Context *ctx, const ExprValue& obj, const char *name, ExprValue argv[], int argc);
};
```

`DynamicallyResolvedXxxNode` 在 `evaluate()` 时调用这些 resolver。上层 `cDynamicExpression` 作为 `cExpression` 的仿真内核子类，通过 `IResolver` 接口将仿真变量（如 NED 参数）绑定为可求值表达式。

### 仿真内核使用 (`cDynamicExpression`)

`cDynamicExpression` 继承 `cExpression`，内部持有 `common::Expression*` 和 `IResolver*`：

```cpp
class cDynamicExpression : public cExpression {
    common::Expression *expression = nullptr;
    IResolver *resolver = nullptr;
    // ...
    virtual void parse(const char *text) override;
    virtual void parse(const char *text, IResolver *resolver);
    virtual cValue evaluate(Context *context) const override;
};
```

`IResolver` 接口与底层 `DynamicResolver` 类似但使用 `cValue`（仿真内核值类型），用于解析 NED 参数引用（如 `host.delay`）、NED 函数（如 `uniform()`, `intuniform()`）等。

### `SymbolTable` 便捷解析器

`cDynamicExpression` 内嵌 `SymbolTable` 类，从 `std::map<string, cValue>` 提供变量值：

```cpp
class SymbolTable : public IResolver {
    std::map<std::string, cValue> variables;
    std::map<std::string, std::vector<cValue>> arrays;
    virtual cValue readVariable(Context *context, const char *name) override;
    virtual cValue readVariable(Context *context, const char *name, intval_t index) override;
};
```

## 八、ExprValue 值系统

```cpp
class ExprValue {
    enum Type { UNDEF=0, BOOL='B', INT='L', DOUBLE='D', STRING='S', POINTER='O' };
    Type type;
    union { bool bl; intval_t intv; double dbl; const char *s; };
    any_ptr ptr;              // POINTER 类型
    opp_staticpooledstring unit; // 量纲单位（如 "s", "mW"）
};
```

特点：
- 支持**量纲**（unit）：`5s`（5秒）、`10mW`（10毫瓦），编译和运算时进行量纲一致性检查
- `POINTER` 类型用于传递 `cObject*` 等仿真对象指针
- `UNDEF` 表示未定义值
- 量纲检查机制：`ensureNoLogarithmicUnit()` 拒绝对数单位参与运算；`bringToCommonTypeAndUnit()` 自动做单位换算

## 九、NED 表达式的特殊处理

`cDynamicExpression::parseNedExpr()` 在通用表达式解析基础上，额外支持 NED 特有名称解析（如 NED 函数注册表 `cNedFunction`）。NED 翻译器链在默认 4 个翻译器之前或之间插入，用于解析 NED 特有函数（`uniform`, `exponential`, `intuniform`, `bernoulli` 等）和 NED 参数引用。

## 十、扩展机制：自定义 AstTranslator

用户可通过 `Expression::installAstTranslator()` 向默认翻译器链追加自定义翻译器：

```cpp
static void installAstTranslator(AstTranslator *translator);
```

这使得表达式编译器可扩展支持新的语法元素（自定义函数、新的运算符语义等），无需修改 Flex/Bison 语法文件。

## 十一、数据流全景

```
"sin(2*pi*freq) + offset"

     ┌───────────────────────────────────────────────────────┐
     │                    expression.lex                      │
     │  NAME("sin") → FUNCTION  INT(2) → INT  ...          │
     └───────────────────────────┬───────────────────────────┘
                                 │ tokens
     ┌───────────────────────────▼───────────────────────────┐
     │                    expression.y                       │
     │  构建 AstNode 树:                                     │
     │  OP("+",                                              │
     │    FUNCTION("sin", MUL(CONSTANT(2), IDENT("pi"))),  │
     │    IDENT("offset"))                                   │
     └───────────────────────────┬───────────────────────────┘
                                 │ AstNode*
     ┌───────────────────────────▼───────────────────────────┐
     │           MultiAstTranslator (4级链)                  │
     │  Operator → StdMath → UnitConv → UnresolvedName      │
     │                                                       │
     │  sin → MathFunc1Node(sin)                             │
     │  pi  → DynamicallyResolvedVariableNode("pi")          │
     │  *   → MulNode                                        │
     │  +   → AddNode                                        │
     │  freq   → DynamicallyResolvedVariableNode("freq")     │
     │  offset → DynamicallyResolvedVariableNode("offset")   │
     └───────────────────────────┬───────────────────────────┘
                                 │ ExprNode*
     ┌───────────────────────────▼───────────────────────────┐
     │           performConstantFolding()                    │
     │  sin(2*pi) → 无法折叠（pi 运行时解析）                  │
     │  常量子表达式如 "1+2" → ConstantNode(3)               │
     └───────────────────────────┬───────────────────────────┘
                                 │ ExprNode* (最终求值树)
     ┌───────────────────────────▼───────────────────────────┐
     │           Expression::evaluate(context)               │
     │  递归求值: AddNode.evaluate() →                       │
     │    left= MathFunc1Node.evaluate() → sin(context)      │
     │    right= DynamicallyResolvedVarNode.evaluate() →     │
     │          resolver->readVariable(context, "offset")     │
     └───────────────────────────────────────────────────────┘
```

## 十二、关键源文件索引

| 文件 | 位置 | 作用 |
|------|------|------|
| `expression.lex` | `src/common/` | Flex 词法分析器 |
| `expression.y` | `src/common/` | Bison 语法分析器（生成 AST） |
| `expression.tab.h` | `src/common/` | Bison 生成的 Token 定义与 YYSTYPE |
| `expression.h` | `src/common/` | `Expression` 类 + `AstNode` 定义 |
| `expression.cc` | `src/common/` | 编译、翻译、求值、常量折叠、翻译器链 |
| `exprnode.h` | `src/common/` | `ExprNode` 基类及 LeafNode/UnaryNode/BinaryNode 等 |
| `exprnodes.h` | `src/common/` | 全部具体 ExprNode 子类（ConstantNode, AddNode 等） |
| `exprvalue.h` | `src/common/` | `ExprValue` 值类型（含量纲支持） |
| `cexpression.h` | `include/` | 仿真内核抽象基类 `cExpression` |
| `cdynamicexpression.h` | `include/` | `cDynamicExpression` — 仿真内核表达式实现 |
| `cnedfunction.h` | `include/` | NED 函数注册接口 |
| `expressionfilter.h` | `src/sim/` | NED/参数过滤器 |

## 十三、设计局限性分析

### 13.1 类型系统：运行时动态类型，无编译期类型检查

`ExprValue` 是动态类型联合体（`UNDEF/BOOL/INT/DOUBLE/STRING/POINTER`），编译器在 AST→ExprNode 翻译阶段**不做类型推断或类型检查**。所有类型错误只在求值时抛出。

```cpp
// 以下表达式在编译期全部成功，运行时才报错
Expression e;
e.parse("3 + \"hello\"");   // 编译期不报错，求值时抛 runtime_error
e.parse("true[5]");          // 编译期不报错，求值时才崩溃
```

这在 NED 参数声明中影响最大——参数声明的类型（`int`, `double`, `bool`, `string`）和表达式实际求值类型之间的不匹配，只能到仿真初始化阶段才暴露。

### 13.2 `isEmpty()` 语义疑似反转

```cpp
// expression.h:179
bool isEmpty() const {return tree != nullptr;}
```

注释说"Returns true if the expression is empty. An empty expression cannot be evaluated."，但实现是 `tree != nullptr`——当树存在时返回 `true`（即"空"），与注释语义**截然相反**。这应是一个 bug，`tree == nullptr` 时表达式才真正为"空"。

### 13.3 AST `unparse()` 依赖括号标记，而非优先级自动推导

```cpp
// expression.cc:79-80 注释
// Note: This code is oblivious of operator precedence, it solely relies 
// on the "parenthesized" flag in each AST node to insert parens where needed.
// If those flags are not set correctly, the unparsed expression may have 
// a different meaning from the original!
```

`AstNode::parenthesized` 是在语法分析时由 Bison 产生式 `'(' expr ')'` 设置的。如果 AST 是**程序构造而非解析得到**的，`parenthesized` 默认为 `false`，反序列化后的表达式可能因优先级不同而含义改变。

**示例**：手动构造 `OP("*", OP("+", a, b), c)` 而未标记 `a+b` 带括号，则 `unparse()` 输出 `a+b*c`（语义错误，应为 `(a+b)*c`）。

### 13.4 常量折叠范围有限

`isFoldableNode()` 只认可以下节点：

- `ConstantNode`
- `UnaryOperatorNode / BinaryOperatorNode / TernaryOperatorNode`
- `DoubleCastNode / IntCastNode`
- `MathFunc1/2/3Node`（仅限 `ExprNodeFactory::supportsStdMathFunction` 注册的）

**不折叠的**：
- `FunctionNode`（自定义 NED 函数如 `uniform()` 不可折叠——参数相同但每次调用结果不同）
- `MethodNode`（方法调用有副作用）
- `VariableNode / MemberNode`（变量值运行时才确定）

**缺失的优化**：无公共子表达式消除、无代数化简（如 `x*1→x`、`x+0→x`）、无短路求值常量传播。

### 13.5 `=` 运算符语义模糊

语法文件允许：

```yacc
expr '=' expr    { $$ = newOp("=", $1, $3); }
```

但在默认翻译器链中，**没有 `createOperatorNode("=", 2)` 的处理**——`OperatorAstTranslator::createOperatorNode()` 只处理一元/二元/三元运算符，不区分运算符名称，`ExprNodeFactory::createBinaryOperator("=")` 返回 `nullptr`。

这意味着 `a = 5` **能解析但不能翻译**，会抛出 "Unknown operator '='" 错误。`=` 的存在是因为 NED 参数赋值语法需要，但它作为表达式运算符是**不可求值的**，语法与语义不一致。

### 13.6 后缀 `!` 运算符的歧义

```yacc
expr '!'    { $$ = newOp("_!", $1); /*!!!*/ }
```

这用于**单位后缀表示**（如 `5dBm!`），但语法上任何表达式后跟 `!` 都合法。`_!` 运算符在 `OperatorAstTranslator` 中无对应节点，会导致翻译失败。这个语法规则本质上只为 NED 的量纲解析服务，但在通用表达式语法中创建了**一个合法但不可求值的语法结构**。

### 13.7 原始指针管理

AST 和 ExprNode 树全靠**裸指针 + 析构函数递归 delete**：

```cpp
~AstNode() {for (AstNode *child : children) delete child;}
~UnaryNode() {delete child;}
~BinaryNode() {delete child1; delete child2;}
~NaryNode() {for (ExprNode *child : children) delete child;}
```

风险：
- `FunctionNode` 和 `MethodNode` 有 `mutable ExprValue *values = nullptr` 预分配缓冲区，析构用 `delete[]`，但需要与子节点生命周期同步
- `parseAndTranslate()` 中用 `std::unique_ptr<AstNode>` 管理根节点，但翻译后的 ExprNode 用裸指针传出
- 拷贝构造和赋值运算符依赖 `dupTree()` 做深拷贝，遗漏即泄漏

### 13.8 Bison YYSTYPE 使用原始 `const char*`

```yacc
%union {
    const char *str;
    AstNode *node;
}
```

语法动作中大量 `delete [] $1` 手动释放字符串内存（如 `delete [] $<str>1`），容易遗漏或重复释放。现代做法应使用 `std::string` 或 Bison 的 `%type` 语义值。

### 13.9 线性翻译器链：无优先级、无冲突检测

```cpp
ExprNode *MultiAstTranslator::translateToExpressionTree(AstNode *astNode, AstTranslator *translatorForChildren) {
    for (AstTranslator *translator : translators)
        if (ExprNode *node = translator->translateToExpressionTree(astNode, translatorForChildren))
            return node;
    return nullptr;
}
```

这是**简单的线性责任链**，第一个成功的翻译器即胜出。问题：
- 如果翻译器 A 和 B 都能处理同一名称（如 `sin` 既可以是 MathFunc 又可以是用户函数），**只由注册顺序决定**，无重载或歧义检测
- 翻译器返回 `nullptr` 表示"不认识"，但如果翻译器**认出了名字但参数不匹配**，应该**抛异常而非返回 nullptr**——`BasicAstTranslator` 的注释明确了这一点，但自定义翻译器可能忽略

### 13.10 表达式不可组合/不可替换

`Expression` 类没有提供：
- **子表达式替换**：无法将 `a+b` 中的 `a` 替换为 `c`
- **表达式组合**：无法将两个 Expression 对象组合成新表达式
- **变量绑定**：无法在不修改 resolver 的情况下临时绑定变量值
- **部分求值**：无法在已知部分变量值时做部分求值/化简

如需这些功能，必须重建表达式或手工操作 ExprNode 树。

### 13.11 错误信息缺乏位置信息

```cpp
void yyerror(yyscan_t scanner, AstNode *&resultAstTree, const char* msg) {
    throw std::runtime_error(opp_removeend(msg, "\n").c_str());
}
```

解析错误只抛 `std::runtime_error`，不含行列号。虽然 Flex scanner 维护了 `lineno` 和 `column`（通过 `yyset_lineno`/`yyset_column`），但错误信息中未使用。与 NED/MSG 解析器（提供精确位置信息）形成对比。

### 13.12 `cDynamicExpression` 与 `Expression` 的双层抽象缝隙

```
cExpression (抽象基类, sim/)
   └── cDynamicExpression (sim/)
         内含 common::Expression (common/)
```

值类型双轨制：
- `Expression` 使用 `ExprValue`（common 层）
- `cDynamicExpression` 使用 `cValue`（sim 层）

两者类型系统不同但需要互相转换。`cDynamicExpression::evaluate()` 内部在 sim 层 Context 和 common 层 Context 之间做适配，增加了复杂度。

### 13.13 线程安全

`Expression` 对象**非线程安全**：
- `defaultTranslator` 是静态全局变量
- `evaluate()` 无锁，内部 `ExprValue` 的单位转换等操作修改临时状态
- 多线程环境需每个线程独立持有 Expression 副本

### 13.14 性能特征

| 方面 | 特性 |
|------|------|
| 解析 | 每次从头解析，无缓存/复用 |
| 求值 | 树遍历解释执行，无 JIT |
| 内存 | 每个节点一个堆对象（`new`），无池分配 |
| 常量折叠 | 仅编译期一次，运行期无优化 |

对 NED 参数求值（每次事件可能重复求同一参数），性能瓶颈在求值而非解析。OMNeT++ 通过 `cPar` 在参数值不变时缓存结果来缓解，但这不是表达式编译器本身的优化。

### 13.15 局限性总结

OMNeT++ 表达式编译器的核心局限是**动态类型 + 无编译期类型检查 + 树遍历解释执行**。这使其灵活但牺牲了安全性和性能。对于仿真配置脚本这种"写一次、求值百万次"的场景，更理想的架构是编译期类型推断 + 字节码/AST 直接执行 + 运行期缓存，但这些权衡是 OMNeT++ 保持通用性和可扩展性的设计选择。