# AGENTS.md — OMNeT++ 6.4.0

> See also `.agent/rules/architecture.md` for detailed directory-to-component mapping and IDE plugin table.

## Project Identity

OMNeT++ is a C++ discrete event simulation framework. Version 6.4.0, branch `omnetpp-6.x`.
License: Academic Public License.

## Language Policy

本项目中的所有文档、代码注释、git提交消息、AI会话统一使用中文。

## Quick Start

```bash
source setenv                              # MUST run first — sets PATH, LD_LIBRARY_PATH
```

For a fresh clone, the simplest path is `./install.sh -y` (Linux/macOS), which installs system deps and builds. For manual build:

```bash
cp configure.user.dist configure.user       # Copy default build config
source setenv                               # MUST run first
./configure && make -j$(nproc)              # Build everything
```

## Running Tests

```bash
make tests              # Full suite from repo root (builds + runs all)
cd test && make test_quick                  # Fast subset only (core, envir, common, etc.)
cd test/core && ./runtest <specific>.test  # Single test
cd test/core && ./runtest                   # All tests in one directory
```

Key test suites: `test_core` (kernel, ~750 files), `test_envir`, `test_common`, `test_fingerprint`, `test_scave_*`. See `test/AGENTS.md` for the `.test` file format and full suite list.

## Repository Structure

```
src/          C++ core (10 subsystems — see src/AGENTS.md)
include/      Public API headers (123 files, all c-prefixed)
test/         Regression tests — see test/AGENTS.md
samples/      Example simulations — see samples/AGENTS.md
python/       Python bindings and tools — see python/AGENTS.md
ui/           Eclipse IDE plugins — see ui/AGENTS.md
doc/          Manuals, API docs, guides
images/       Icons and graphics for simulations
misc/         Third-party integrations (gdb, emacs, octave)
releng/       Release engineering scripts
```

## Build System

- `./configure` → generates `Makefile.inc` (detected compilers, paths, flags)
- `cp configure.user.dist configure.user` — required before first `./configure`
- `configure.user` controls optional features (QtEnv, Python, OSG, SystemC, etc.)
- Build modes: `MODE=release` (default), `MODE=debug`, `MODE=sanitize`, `MODE=coverage`, `MODE=profile`
- Library naming: `$(LIB_PREFIX)opp<subsystem>$D$(LIB_SUFFIX)` where `$D` = mode suffix
- Each src/ subsystem has its own Makefile including `../../Makefile.inc`
- IDE native libs: `make ui`
- Build order enforced by Makefile dependency chain (not parallel-safe at top level)

### Dependency Chain

```
common ← layout, eventlog, scave, nedxml, sim, envir, cmdenv, qtenv, utils
sim ← nedxml + common
envir ← sim
cmdenv, qtenv ← envir
qtenv ← layout
utils (standalone tools, built first as prerequisite)
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

## Python Environment

Optional Python bindings and tools require a venv:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r python/requirements.txt   # matplotlib, numpy, pandas
```
`setenv` auto-activates `.venv/` if present. Required for `WITH_SCAVE_PYTHON_BINDINGS=yes` and `opp_charttool`.

## CI/CD

Three GitHub workflows in `.github/workflows/`:
- `build_release.yml` — Full release builds (Linux, macOS, Windows)
- `build_tests.yml` — Build verification on push/PR (ubuntu-24.04, clang, Qt6)
- `main_tests.yml` — Full test suite on push/PR

CI build steps:
```bash
cp configure.user.dist configure.user
source setenv
./configure WITH_LIBXML=yes WITH_QTENV=yes WITH_OSG=yes WITH_OSGEARTH=no
make -j4
cd test && make test_quick   # runs each test suite
```

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

## Navigation Tips

- Kernel internals: `src/sim/` + `include/omnetpp/`
- Tools: `src/nedxml/`, `src/scave/`, `src/utils/`
- Common utilities: `src/common/`
- IDE: `ui/` (Eclipse plugin structure)
- Python: `python/omnetpp/`
- Public API defines what simulation models can use: `include/omnetpp/`
- Makefile dependency chain: all `$(BASE)` targets require `utils` built first (see root `Makefile`)
