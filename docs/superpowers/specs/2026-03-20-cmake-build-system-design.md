# OMNeT++ CMake Build System Design

**Date**: 2026-03-20
**Author**: AI Agent
**Status**: Draft (Revised after Oracle review)

## Overview

Add CMake build support to OMNeT++ 6.4.0 as a parallel build system alongside the existing Autoconf/Makefile system. Target: modern developer experience (IDE integration, ninja, CLion/VS Code CMake Tools support).

## Goals

- **Primary**: Enable modern build toolchain (ninja, IDE integration)
- **Scope**: Complete developer runtime (libraries + tools + executables)
- **Platform**: Linux only (initial implementation)
- **Coexistence**: CMake and Makefile run in parallel, user selects

## Deliverables (Complete Developer Runtime)

A CMake build must produce a usable OMNeT++ runtime, not just libraries:

### Libraries
| Library | CMake Target | Output File |
|---------|--------------|-------------|
| oppcommon | `OMNeTpp::common` | `liboppcommon.so` |
| opplayout | `OMNeTpp::layout` | `libopplayout.so` |
| oppeventlog | `OMNeTpp::eventlog` | `liboppeventlog.so` |
| oppscave | `OMNeTpp::scave` | `liboppscave.so` |
| oppnedxml | `OMNeTpp::nedxml` | `liboppnedxml.so` |
| oppsim | `OMNeTpp::sim` | `liboppsim.so` |
| oppenvir | `OMNeTpp::envir` | `liboppenvir.so` |
| oppcmdenv | `OMNeTpp::cmdenv` | `liboppcmdenv.so` |
| oppqtenv | `OMNeTpp::qtenv` | `liboppqtenv.so` |

### Tools and Executables
| Tool | CMake Target | Purpose |
|------|--------------|---------|
| opp_run | `OMNeTpp::opp_run` | Main simulation runner |
| opp_nedtool | `OMNeTpp::opp_nedtool` | NED file compiler |
| opp_msgtool | `OMNeTpp::opp_msgtool` | MSG file compiler |
| opp_scavetool | `OMNeTpp::opp_scavetool` | Result file processor |
| opp_makemake | `OMNeTpp::opp_makemake` | Makefile generator |
| opp_test | `OMNeTpp::opp_test` | Test runner |
| opp_featuretool | `OMNeTpp::opp_featuretool` | Feature toggle manager |
| opp_charttool | `OMNeTpp::opp_charttool` | Chart generation |

### Excluded from Scope
- Python bindings (`src/scave/python/`)
- JNI native libs (`ui/org.omnetpp.ide.nativelibs/`)
- IDE plugins (`ui/` Maven/Tycho build)

## Non-Goals

- Replace existing Makefile system (coexistence model)
- Support Python bindings or JNI native libs via CMake
- Windows/macOS platform support (future work)
- Modify sample project build (they continue using `opp_makemake`)

## Architecture

### Directory Structure

```
/workspace/omnetpp/
├── CMakeLists.txt              # Root configuration
├── cmake/
│   ├── OMNeTppConfig.cmake     # Platform detection, compiler setup
│   ├── OMNeTppFeatures.cmake   # Feature toggles
│   ├── OMNeTppDependencies.cmake # External dependency find_package wrappers
│   ├── OMNeTppMsgCompiler.cmake # opp_msgtool integration
│   ├── OMNeTppCodeGen.cmake    # Flex/bison/codegen helpers
│   └── ver.h.in                # Version header template
├── src/
│   ├── common/CMakeLists.txt
│   ├── layout/CMakeLists.txt
│   ├── eventlog/CMakeLists.txt
│   ├── scave/CMakeLists.txt
│   ├── nedxml/CMakeLists.txt   # + opp_nedtool, opp_msgtool
│   ├── sim/CMakeLists.txt
│   ├── envir/CMakeLists.txt    # + opp_run
│   ├── cmdenv/CMakeLists.txt
│   ├── qtenv/CMakeLists.txt
│   └── utils/CMakeLists.txt    # opp_makemake, opp_test, etc.
└── include/omnetpp/
```

### Library Dependency Graph

```
                    ┌─────────────┐
                    │  oppcommon  │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌───────────┐    ┌─────────────┐    ┌─────────────┐
│ opplayout │    │ oppscave    │    │ oppnedxml   │ ← opp_nedtool, opp_msgtool
└───────────┘    └─────────────┘    └──────┬──────┘
        │                                   │
        │         ┌─────────────┐           │
        │         │  oppsim     │◄──────────┘
        │         └──────┬──────┘
        │                │
        │         ┌──────┴──────┐
        │         │  oppenvir   │ ← opp_run
        │         └──────┬──────┘
        │                │
        │    ┌───────────┴───────────┐
        │    │                       │
        ▼    ▼                       ▼
┌─────────────┐              ┌─────────────┐
│ oppcmdenv   │              │ oppqtenv    │
└─────────────┘              └─────────────┘
```

## Feature Toggles

### Complete Feature Parity

```cmake
# Core features
option(WITH_QTENV "Build Qt-based graphical runtime" ON)
option(WITH_OSG "Enable OpenSceneGraph support in Qtenv" OFF)
option(WITH_OSGEARTH "Enable osgEarth support (requires OSG)" OFF)
option(WITH_NETBUILDER "Enable dynamic NED loading" ON)
option(WITH_PYTHON "Enable Python interpreter embedding" ON)
option(WITH_PARSIM "Enable parallel distributed simulation" OFF)
option(WITH_SYSTEMC "Enable SystemC support" OFF)
option(WITH_LIBXML "Enable LibXML2 for DOCTYPE XML files" OFF)
option(WITH_AKAROA "Enable Akaroa support" OFF)
option(WITH_BACKTRACE "Enable backtrace printing on exceptions" ON)

# Build options
option(OMNETPP_SHARED_LIBS "Build shared libraries" ON)
option(PREFER_SQLITE_RESULT_FILES "Use SQLite as default result format" OFF)
```

### Build Types (Simplified)

For initial implementation, support only Release and Debug:

| CMAKE_BUILD_TYPE | Equivalent MODE | Output Suffix |
|-------------------|-----------------|---------------|
| Release | release | (none) |
| Debug | debug | `_dbg` |

**Future work**: Add sanitize/profile/coverage as explicit CMake presets or custom targets, not remapped standard build types.

## Generated Artifacts

### Complete Inventory

| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `*.msg` | `*_m.cc`, `*_m.h` | `add_custom_command()` via `opp_msgtool` |
| `*.y` (bison) | `*.tab.cc`, `*.tab.h` | `find_package(BISON)` + `BISON_TARGET()` |
| `*.lex` (flex) | `*.lex.cc` | `find_package(FLEX)` + `FLEX_TARGET()` |
| `*.ui` (Qt) | `ui_*.h` | `AUTOUIC` |
| `*.h` (Q_OBJECT) | `moc_*.cpp` | `AUTOMOC` |
| `*.qrc` (Qt) | `qrc_*.cpp` | `AUTORCC` |
| `sim_std.msg` (sim) | `sim_std_m.cc/h` | `opp_msgtool` |
| `eventlogwriter.pl` (envir) | `eventlogwriter.cc/h` | `add_custom_command()` |
| `dtdclassgen.pl` (nedxml) | `dtdvalidationclasses.cc/h` | `add_custom_command()` |
| `icons_dark.qrc` (qtenv) | Generated from icons.qrc | `add_custom_command()` |

### Bootstrap Order

Some tools are built by the project and then used to generate sources:

1. **Phase 1**: Build `opp_nedtool`, `opp_msgtool` (from nedxml)
2. **Phase 2**: Use these tools to generate `*_m.cc` files
3. **Phase 3**: Build libraries that depend on generated sources

CMake handles this via target dependencies:
```cmake
add_custom_command(
  OUTPUT sim_std_m.cc
  COMMAND opp_msgtool --msg6 sim_std.msg
  DEPENDS opp_msgtool sim_std.msg
)
```

## Compatibility Layer

### setenv Compatibility

CMake builds generate a `setenv.cmake` that can be sourced:

```bash
# After CMake build
source build/setenv.cmake  # Sets PATH, LD_LIBRARY_PATH
```

### opp_configfilepath Compatibility

Generate `opp_configfilepath` executable that returns the config path:
- Returns `CMAKE_INSTALL_PREFIX/lib/omnetpp` for installed builds
- Returns `CMAKE_BINARY_DIR` for in-tree builds

### Makefile.inc Compatibility (Optional)

For `opp_makemake` to work with CMake-built OMNeT++:

1. **Option A**: Generate `Makefile.inc` from CMake variables
2. **Option B**: `opp_makemake` learns to read CMake cache

**Decision**: Option A for initial implementation. Generate `Makefile.inc` during install.

## Installation Layout

```cmake
install(TARGETS ${ALL_LIBRARIES} ${ALL_TOOLS}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/omnetpp
  DESTINATION include
)

install(FILES 
  cmake/OMNeTppConfig.cmake
  cmake/OMNeTppConfigVersion.cmake
  DESTINATION lib/cmake/OMNeTpp
)

install(EXPORT OMNeTppTargets
  FILE OMNeTppTargets.cmake
  NAMESPACE OMNeTpp::
  DESTINATION lib/cmake/OMNeTpp
)

# Generate compatibility files
install(SCRIPT "${CMAKE_SOURCE_DIR}/cmake/install_compat.cmake")
```

## Build Commands

### Basic Build
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

### Debug Build
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j$(nproc)
```

### Install
```bash
cmake --install build --prefix /opt/omnetpp
source /opt/omnetpp/setenv
```

### With Features Disabled
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release \
  -DWITH_QTENV=OFF -DWITH_PYTHON=OFF
```

## Testing Strategy

### Acceptance Criteria

| Test | Pass Criteria |
|------|---------------|
| **Configure** | `cmake -B build` succeeds |
| **Build** | `cmake --build build` completes with 0 errors |
| **Library Inventory** | All 9 libraries built, `nm` shows expected symbols |
| **Tool Inventory** | All 8 tools built and executable |
| **opp_run Test** | Run `opp_run -h` successfully |
| **Sample Build** | Build one sample simulation with `opp_makemake` |
| **Sample Run** | Run the sample simulation successfully |
| **Parity Check** | Same simulation produces same output with Make vs CMake builds |

### Test Configuration
```bash
# Minimum test matrix
cmake -B build-release -DCMAKE_BUILD_TYPE=Release
cmake -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake -B build-noqt -DCMAKE_BUILD_TYPE=Release -DWITH_QTENV=OFF
```

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Bootstrap ordering issues | Medium | High | CMake target dependencies, prototype early |
| MSG tool not found | Low | High | Build tool first, then use in same CMake run |
| Qt version incompatibility | Low | Medium | Require Qt >= 6.2, explicit error message |
| Linker flag differences | Medium | Medium | Compare `ldd` output, rpath handling |
| Generated source divergence | Low | Medium | Byte-compare generated files with Make build |
| opp_makemake incompatibility | Medium | High | Generate Makefile.inc during install |

## Implementation Phases

### Phase 1: Infrastructure (1-2 days)
- Root CMakeLists.txt with version, options
- cmake/ modules for features, dependencies, config
- Generated sources: ver.h, opp_configfilepath

### Phase 2: Core Libraries (2-3 days)
- src/common/CMakeLists.txt (no generated sources)
- src/layout/CMakeLists.txt
- src/eventlog/CMakeLists.txt
- src/scave/CMakeLists.txt

### Phase 3: Tools and NED/XML (2-3 days)
- src/nedxml/CMakeLists.txt with flex/bison
- opp_nedtool, opp_msgtool executables
- MSG compiler cmake function

### Phase 4: Simulation Kernel (2-3 days)
- src/sim/CMakeLists.txt with MSG compilation
- sim_std_m.cc generation

### Phase 5: Runtime Environment (1-2 days)
- src/envir/CMakeLists.txt with eventlogwriter generation
- src/cmdenv/CMakeLists.txt
- opp_run executable

### Phase 6: Qt GUI (2-3 days)
- src/qtenv/CMakeLists.txt
- Qt6 integration with AUTOMOC/UIC/RCC
- OSG subdirectory (optional)

### Phase 7: Utils and Polish (1-2 days)
- src/utils/CMakeLists.txt
- Install targets, export config
- Compatibility layer (setenv, Makefile.inc)

### Phase 8: Testing (1-2 days)
- Verify all acceptance criteria
- Sample simulation tests
- Documentation

**Total Estimated Effort**: 12-18 days

## Decisions (Addressing Open Questions)

1. **Output Directory**: Use `build/` as default. Users can specify any directory via `-B`.
2. **Makefile.inc Generation**: Generate during `cmake --install` for `opp_makemake` compatibility.
3. **setenv Script**: Generate `setenv.cmake` during build, `setenv` during install.

## References

- Existing Makefile structure: `/workspace/omnetpp/Makefile`
- Configuration template: `/workspace/omnetpp/Makefile.inc.in`
- Subsystem Makefiles: `/workspace/omnetpp/src/*/Makefile`
- AGENTS.md: `/workspace/omnetpp/AGENTS.md`
- configure.user.dist: `/workspace/omnetpp/configure.user.dist`