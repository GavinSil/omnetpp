# Draft: Cross-Process Model Loading Architecture

## Requirements (confirmed)
- [core-need]: User wants to load simulation models across processes instead of current shared library mechanism
- [stated-date]: 2026-04-03
- [driver]: "某些仿真模型有自己的进程，不能以共享库方式提交" - models exist as independent processes
- [scope]: "单一主机内跨进程" - single machine IPC, no distributed cross-node

## Current Understanding (Confirmed 2026-04-03)
- **场景**: 存在现有第三方仿真程序，以独立进程运行，需要作为OMNeT++模块参与仿真
- **接口可调**: 外部模型接口可以调整适配OMNeT++
- **交互深度**: 需要完整模块接口代理
  - 消息传递（事件调度）
  - 模块生命周期（initialize/handleMessage/finish）
  - Kernel API访问（参数、统计、日志）
  - 仿真上下文访问（其他模块、拓扑结构）
- **性能特征**: 中等事件频率 (100-10K/s)，IPC延迟约1-10µs可接受（占事件时间1-10%）
- **架构范围**: 单一主机，本地IPC机制

---

## Oracle Consultation (2026-04-03)

### Recommended Architecture

**Proxy Pattern**:
- `cExternalModule` extending `cSimpleModule` — intercepts lifecycle calls, translates to IPC
- OMNeT++ kernel treats external models identically to native modules
- Synchronous IPC for lifecycle (initialize/finish blocks until response)
- Asynchronous message queue for handleMessage (pipelined events)

**IPC Mechanism**:
- **Primary**: Unix domain sockets (1-5µs latency, simple implementation)
- **Optimization path**: Shared memory ring buffer if profiling shows >20% IPC overhead
- **Serialization**: Leverage existing `cCommBuffer` from parsim infrastructure

**Interface Contract**:
- C-style IPC interface (~12 functions) — avoids C++ ABI issues
- Language-agnostic design enables Python, Rust, other bindings
- Functions: initialize, handleMessage, finish, par access, emit, log, send, context queries
- Versioning: `uint32_t interface_version` field in all structs for compatibility

**Registration Mechanism**:
- `Define_External_Module()` macro similar to `Define_Module()`
- Registers factory producing `cExternalModule` proxies
- Configuration: `external-process` parameter in NED for executable path + args

**Error Handling**:
- Watchdog thread for crash detection
- IPC timeout with abort + diagnostic dump
- All functions return error codes, kernel logs and can abort simulation

**Effort Estimate**: Large (3-5 weeks core implementation, 1-2 weeks testing/integration)

### Critical Decisions Requiring User Input

**Decision 1: Context Access Scope** ✓ CONFIRMED
- User choice: **Read-only context queries** (par access, module existence checks)
- Scope: No write operations (topology changes, sendDirect) in v1
- Future path: Defer write operations until use cases emerge

**Decision 2: Message Ownership** ✓ CONFIRMED
- User choice: **Kernel copies** message data
- Behavior: External process sends → kernel serializes → kernel copies to internal → external process relinquishes ownership
- Trade-off: Safe, simpler, measurable overhead. Zero-copy path available if profiling shows bottleneck

**Decision 3: Concurrency Model** ✓ CONFIRMED
- User choice: **Single-threaded kernel, external process async**
- Lifecycle: Synchronous IPC calls (initialize/finish block until response)
- Messages: Asynchronous queue for handleMessage pipelining
- Rationale: Matches current OMNeT++ design semantics

### Guardrails (Scope Boundaries)

- **IPC simplicity**: Unix sockets only for v1. Shared memory ONLY if profiling shows >20% IPC overhead
- **Single-machine scope**: No MPI, no network RPC. Design hooks for future, don't implement
- **Kernel API whitelist**: Reject "just one more API" additions without use case
- **Serialization reuse**: Use parsim's `cCommBuffer`. Extend only when required
- **Language bindings deferred**: C interface first. Python/other bindings are separate projects

### Evolution Path

```
Phase 1 (v1): Single-machine external processes
├── cExternalModule proxy
├── Unix socket IPC
├── C IPC interface
└── Basic lifecycle + messaging

Phase 2: Performance optimization
├── Shared memory ring buffer for messages
├── Batched IPC calls
└── Async handleMessage pipelining

Phase 3: Language bindings
├── Python adapter library
├── Generated IPC stubs from interface definition
└── Example models in Python

Phase 4: Distributed simulation (future)
├── Leverage existing parsim infrastructure
├── External processes on remote nodes
└── Network-transparent IPC (reuse cCommBuffer)
```

### Risks and Mitigation

| Risk | Mitigation |
|------|------------|
| External process crash deadlocks simulation | Watchdog thread, IPC timeout, abort with diagnostic dump |
| Pointer-based kernel API breaks via IPC | Define serializable API subset. Document unsupported operations |
| IPC overhead dominates simulation time | Profile early. Design for shared memory fallback. Performance budget |
| Memory leaks in long-running simulations | RAII for IPC channels. Valgrind integration in tests |
| ABI compatibility across compiler versions | C interface with struct versioning. Test multiple compilers |
| Complex type serialization (cObject pointers) | Restrict to built-in types + strings initially. Proxy complex objects via IDs |

---

## Test Infrastructure Assessment (2026-04-03)

**Infrastructure Exists**: YES
**Framework**: Custom `opp_test` tool (Python-based) with 890+ `.test` files
**CI Integration**: GitHub Actions (main_tests.yml, build_tests.yml)
**Coverage**: `MODE=coverage` build mode supported (clang llvm-cov)
**TDD Feasibility**: EXCELLENT

**Test Pattern**:
- Declarative `.test` format: `%description`, `%activity`, `%contains`, `%exitcode`
- Generates C++ code → compiles → runs simulation → validates output
- Location: `test/envir/` suite for environment/infrastructure work
- Run: `./runtest <testname>.test` or `make test_envir`

**Recommendation**: Adopt TDD workflow — write failing tests first, implement features, verify pass

## Open Questions (Refined)
1. 外部进程模型的特征？
   - 这些模型是现有的第三方程序？还是需要新开发的模型？
   - 它们如何定义自己的行为？有标准接口吗？
   - 是否可以修改这些模型的代码？
   
2. 交互深度？
   - 仅需要消息传递（事件调度）？
   - 还是完整模块生命周期（initialize、handleMessage、finish）？
   - 需要访问OMNeT++ kernel功能吗（参数、统计、日志）？
   
3. 性能要求？
   - 事件处理频率？每秒多少事件？
   - 可接受的IPC延迟？毫秒级还是需要更低？
   
4. 兼容性策略？
   - 需要让外部模型在NED文件中声明使用吗？
   - 现有仿真配置能否无缝支持？
   - 还是需要新的配置方式？

## Research Findings (2026-04-03)

### OMNeT++ Current Architecture (explore agent bg_5de7ee32)

**Shared Library Loading**:
- Entry point: `src/envir/fsutils.cc:45-67` — `opp_loadlibrary()` using `dlopen()`/`LoadLibrary()`
- Registration: `include/omnetpp/regmacros.h:117-159` — `Define_Module()` → `EXECUTE_ON_STARTUP()` → global constructor
- Factory: `src/sim/cobjectfactory.cc` — `cObjectFactory::createOne(classname)` via registered creator function
- Lifecycle: `src/sim/cmodule.cc` — `initialize()` → `activity()/handleMessage()` → `finish()` → `deleteModule()`
- Global state: `cSimulation` singleton manages module hierarchy, FES, RNGs

**Key Constraints**:
1. Pointer-based references (messages, modules) — cannot cross process boundary
2. No serialization built-in for IPC
3. Global singletons assume single-process
4. Registration lists are in-memory, not shared

**Existing IPC Infrastructure**:
- `src/sim/parsim/cnamedpipecomm.cc` — Named pipes (Unix FIFO)
- `src/sim/parsim/cfilecomm.cc` — File-based message passing
- `cCommBuffer` — Serialization base class (used in parallel simulation)
- `cPlaceholderMod` — Placeholder modules for remote partitions

### Cross-Process Simulation Patterns (librarian agent bg_90e21478)

**Production Frameworks**:
- ROSS/WARPED2: MPI + Time Warp optimistic synchronization
- MATSim: Distributed message-passing with millions of agents
- PARSIR: NUMA-optimized shared memory (40+ CPU machines)

**IPC Mechanisms**:
- MPI: Latency 1-10µs intra-node, 10-100µs inter-node, throughput 1-10 GB/s
- Shared Memory: Latency <1µs same NUMA, throughput 10-100 GB/s
- Named Pipes/Unix Sockets: Latency ~1-5µs, simple programming model

**Performance Guidelines**:
- Event granularity >100µs: IPC overhead negligible
- Event granularity 10-100µs: Hybrid approach optimal
- Event granularity <10µs: Shared memory required, batch processing

**Recommended Architecture**:
- Phase 1: Shared memory parallelism (4-8x speedup on 8-16 cores)
- Phase 2: MPI distribution (linear scaling to hundreds of cores)
- Phase 3: Hybrid optimization (near-linear to thousands of cores)

## Scope Boundaries
- INCLUDE: Architecture design for cross-process model loading
- EXCLUDE: Actual implementation (planning phase only)

---
Last updated: 2026-04-03 (initial draft)