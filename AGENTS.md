# AGENTS.md — OMNeT++ 6.4.0

> See also `.agent/rules/architecture.md` for detailed directory-to-component mapping and IDE plugin table.

## Project Identity

OMNeT++ is a C++ discrete event simulation framework. Version 6.4.0, branch `omnetpp-6.x`.
License: Academic Public License.

## Language Policy

本项目中的所有文档、代码注释、git提交消息、AI会话统一使用中文。

## Quick Start

```bash
source setenv                    # MUST run first — sets PATH, LD_LIBRARY_PATH
./configure && make -j$(nproc)   # Build everything
make tests                       # Run all tests (from root)
```

## Repository Structure

```
src/          C++ core (10 subsystems — see src/AGENTS.md)
include/      Public API headers (123 files, all c-prefixed)
test/         Regression tests (890 .test files — see test/AGENTS.md)
samples/      Example simulations (20+ — see samples/AGENTS.md)
python/       Python bindings and tools (see python/AGENTS.md)
ui/           Eclipse IDE plugins (see ui/AGENTS.md)
doc/          Manuals, API docs, guides
images/       Icons and graphics for simulations
misc/         Third-party integrations (gdb, emacs, octave)
releng/       Release engineering scripts
```

## Build System

- `./configure` → generates `Makefile.inc` (detected compilers, paths, flags)
- `make` builds all: common → sim → envir → cmdenv/qtenv, plus nedxml, scave, etc.
- Build modes: `MODE=release` (default), `MODE=debug`, sanitize, coverage, profile
- Library naming: `$(LIB_PREFIX)opp<subsystem>$D$(LIB_SUFFIX)` where `$D` = mode suffix
- Each src/ subsystem has its own Makefile including `../../Makefile.inc`
- IDE native libs: `make ui`

### Dependency Chain

```
common ← layout, eventlog, scave, nedxml, sim, envir, cmdenv, qtenv
sim ← nedxml + common
envir ← sim
cmdenv, qtenv ← envir
qtenv ← layout
```

## C++ Coding Conventions

> Full reference: `python/omnetpp/llmtool/cpp-coding-guidelines.txt` (376 lines)

### Naming

- **Classes**: `c`-prefix for kernel classes (`cModule`, `cMessage`, `cSimulation`)
- **Methods**: camelCase (`handleMessage`, `getFullPath`)
- **C functions**: `opp_` prefix (`opp_isempty`, `opp_streq`)
- **Macros**: `Register_Class()`, `Define_Module()`, `Register_PerObjectConfigOption()`

### Namespaces

- All code lives in `namespace omnetpp { }` (open-brace style)
- `src/common/` uses nested `namespace omnetpp { namespace common { } }`
- `.cc` files: `using namespace omnetpp::common;`

### Braces and Formatting

- **Functions/classes**: Opening brace on NEW line
- **Control blocks**: Egyptian braces (same line)
- **Single-statement blocks**: NO braces
- **else**: `}\nelse {` pattern

```cpp
// Function — opening brace on new line
void cModule::handleMessage(cMessage *msg)
{
    if (msg->isSelfMessage())
        handleTimer(msg);    // Single statement — no braces
    else {
        processPacket(msg);
        delete msg;
    }
}
```

### Headers

- Include guards: `#ifndef __OMNETPP_<SUBSYSTEM>_<FILE>_H`
- Banner: `//===...===//` separator, copyright block
- Public API: `#include "omnetpp/cmodule.h"` — uses `SIM_API` export macro
- Internal: `#include "common/stringutil.h"` (relative from `src/`)
- Use `<cstring>` not `<string.h>`

### Pointers, Types, Casts

- Raw pointers or `shared_ptr` — **NO `unique_ptr`**
- Always `nullptr`, never `NULL`/`0`
- Check pointers explicitly: `!= nullptr`, `== nullptr`
- **Numeric casts**: C-style `(int)x`, NOT `static_cast<int>(x)`
- `dynamic_cast` must always have nullptr check
- `check_and_cast<T*>()` never returns nullptr — don't check its result

### Strings

- Prefer `std::string` over C-string operations
- C-string empty check: `opp_isempty(s)` not `strlen(s) == 0`
- C-string comparison: `opp_streq(a, b)` not `strcmp(a, b) == 0`

### Conditions and auto

- No `? true : false`, no `foo == true`
- Explicit bool: `!= 0`, `!= nullptr` — avoid implicit conversions
- Ternary only for assignments/arguments, never standalone
- `auto` only for STL iterators/pairs/lengthy template types — NOT for primitives

### OMNeT++ Specifics

- `initialize()`, `handleMessage()` etc. always with `override`
- Module params: `par("x")` (implicit conversion), NOT `par("x").doubleValue()`
- Read param strings into `std::string`, not `const char*`
- `using namespace omnetpp;` always present in simulation code
- `simtime_t`: always init to zero. `simsignal_t`: do NOT zero-init

### Data Members

- Data members precede member functions in class declarations
- Prefer inline member initialization with `= 0`, not `{}`
- Remove empty/default destructors
- No `override` on destructors
- Remove redundant initializer-list entries when inline-initialized

## CI/CD

Three GitHub workflows in `.github/workflows/`:
- `build_release.yml` — Release builds
- `build_tests.yml` — Build + test on push/PR to master/omnetpp-6.x (ubuntu-24.04, clang, Qt6)
- `main_tests.yml` — Main test suite

CI steps: `cp configure.user.dist configure.user` → `source setenv` → `cd test` → `make -j4 test_build`

## Key Entry Points

| Tool | Location | Purpose |
|------|----------|---------|
| `opp_run` | `src/envir/main.cc` | Main simulation runner |
| `opp_nedtool` | `src/nedxml/` | NED file compiler/validator |
| `opp_msgtool` | `src/nedxml/` | MSG file compiler |
| `opp_scavetool` | `src/scave/` | Result file processor |
| `opp_test` | `src/utils/` | Test runner tool |
| `opp_makemake` | `src/utils/` | Makefile generator |
| `opp_featuretool` | `src/utils/` | Feature toggle manager |
| `opp_charttool` | `src/utils/` | Chart generation |

## Known Technical Debt

263 TODO/FIXME items across 99 .cc files. Hotspots:
- `src/nedxml/nedcrossvalidator.cc` (29)
- `src/envir/eventlogfilemgr.cc` (15)
- `src/qtenv/mainwindow.cc` (12)
- `src/qtenv/qtenv.cc` (12)

## Navigation Tips

- Kernel internals: `src/sim/` + `include/omnetpp/`
- Tools: `src/nedxml/`, `src/scave/`, `src/utils/`
- Common utilities: `src/common/`
- IDE: `ui/` (Eclipse plugin structure)
- Python: `python/omnetpp/`
- Public API defines what simulation models can use: `include/omnetpp/`
