# OMNeT++ CMake Build System Design

**Date**: 2026-03-20
**Author**: AI Agent
**Status**: Draft

## Overview

Add CMake build support to OMNeT++ 6.4.0 as a parallel build system alongside the existing Autoconf/Makefile system. Target: modern developer experience (IDE integration, ninja, CLion/VS Code CMake Tools support).

## Goals

- **Primary**: Enable modern build toolchain (ninja, IDE integration)
- **Scope**: Core libraries + QtEnv (excludes Python bindings, JNI/IDE native libs)
- **Platform**: Linux only (initial implementation)
- **Coexistence**: CMake and Makefile run in parallel, user selects

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
│   ├── OMNeTppFeatures.cmake   # Feature toggles (WITH_QTENV, etc.)
│   ├── OMNeTppDependencies.cmake # External dependency find_package wrappers
│   └── OMNeTppMsgCompiler.cmake # opp_msgtool integration
├── src/
│   ├── common/CMakeLists.txt   # oppcommon library
│   ├── layout/CMakeLists.txt   # opplayout library
│   ├── eventlog/CMakeLists.txt # oppeventlog library
│   ├── scave/CMakeLists.txt    # oppscave library
│   ├── nedxml/CMakeLists.txt   # oppnedxml library
│   ├── sim/CMakeLists.txt      # oppsim library
│   ├── envir/CMakeLists.txt    # oppenvir library + opp_run executable
│   ├── cmdenv/CMakeLists.txt   # oppcmdenv library
│   └── qtenv/CMakeLists.txt    # oppqtenv library (Qt6 integration)
└── include/omnetpp/            # Public headers (unchanged)
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
│ opplayout │    │ oppscave    │    │ oppnedxml   │
└───────────┘    └─────────────┘    └──────┬──────┘
        │                                   │
        │         ┌─────────────┐           │
        │         │  oppsim     │◄──────────┘
        │         └──────┬──────┘
        │                │
        │         ┌──────┴──────┐
        │         │  oppenvir   │
        │         └──────┬──────┘
        │                │
        │    ┌───────────┴───────────┐
        │    │                       │
        ▼    ▼                       ▼
┌─────────────┐              ┌─────────────┐
│ oppcmdenv   │              │ oppqtenv    │
└─────────────┘              └─────────────┘
        │                           │
        └───────────┬───────────────┘
                    │
                    ▼
              opp_run (executable)
```

### CMake Targets

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
| opp_run | `OMNeTpp::opp_run` | `opp_run` (executable) |

## Feature Toggles

### Cache Options

```cmake
option(WITH_QTENV "Build Qt-based graphical runtime" ON)
option(WITH_OSG "Enable OpenSceneGraph support in Qtenv" OFF)
option(WITH_OSGEARTH "Enable osgEarth support (requires OSG)" OFF)
option(WITH_NETBUILDER "Enable dynamic NED loading" ON)
option(WITH_PYTHON "Enable Python interpreter embedding" ON)
option(WITH_PARSIM "Enable parallel distributed simulation" OFF)
option(WITH_SYSTEMC "Enable SystemC support" OFF)
option(OMNETPP_SHARED_LIBS "Build shared libraries" ON)
```

### Build Types

| CMAKE_BUILD_TYPE | Equivalent MODE | Output Suffix |
|-------------------|-----------------|---------------|
| Release | release | (none) |
| Debug | debug | `_dbg` |
| RelWithDebInfo | sanitize | `_sanitize` |
| MinSizeRel | profile | `_profile` |

Note: CMake multi-config generators (Ninja Multi-Config, VS) can build multiple types in one tree.

## Key Implementation Details

### 1. MSG File Compilation

OMNeT++ uses `.msg` files that compile to `*_m.cc` and `*_m.h` via `opp_msgtool`.

```cmake
# cmake/OMNeTppMsgCompiler.cmake
function(omnetpp_add_msg_library target)
  find_program(OPP_MSGTOOL opp_msgtool
    PATHS "${OMNETPP_BIN_DIR}"
    REQUIRED
  )
  
  foreach(msg_file ${ARGN})
    get_filename_component(base ${msg_file} NAME_WE)
    
    add_custom_command(
      OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${base}_m.cc
             ${CMAKE_CURRENT_BINARY_DIR}/${base}_m.h
      COMMAND ${OPP_MSGTOOL} --msg6 ${CMAKE_CURRENT_SOURCE_DIR}/${msg_file}
      DEPENDS ${msg_file}
      COMMENT "MSG: ${msg_file}"
    )
    
    list(APPEND generated_sources ${CMAKE_CURRENT_BINARY_DIR}/${base}_m.cc)
  endforeach()
  
  add_library(${target} ${generated_sources} ...)
endfunction()
```

Usage:
```cmake
omnetpp_add_msg_library(sim_std_m sim_std.msg)
target_link_libraries(oppsim PRIVATE sim_std_m)
```

### 2. Qt Integration (qtenv)

Qt requires MOC, UIC, and RCC code generation. CMake's `AUTOMOC`, `AUTOUIC`, `AUTORCC` handle this automatically.

```cmake
# src/qtenv/CMakeLists.txt
find_package(Qt6 REQUIRED COMPONENTS 
  Core Gui Widgets OpenGL OpenGLWidgets PrintSupport
)

add_library(oppqtenv SHARED ${SOURCES})
target_link_libraries(oppqtenv
  PUBLIC OMNeTpp::envir
  PUBLIC OMNeTpp::layout
  PUBLIC OMNeTpp::common
  PRIVATE Qt6::Core Qt6::Gui Qt6::Widgets 
          Qt6::OpenGL Qt6::OpenGLWidgets Qt6::PrintSupport
)

set_target_properties(oppqtenv PROPERTIES
  AUTOMOC ON
  AUTOUIC ON
  AUTORCC ON
)
```

### 3. Version Generation

The existing build generates `ver.h` from the `Version` file:

```cmake
# Read version from file
file(STRINGS "${CMAKE_SOURCE_DIR}/Version" OMNETPP_VERSION LIMIT_COUNT 1)

# Generate ver.h
configure_file(
  "${CMAKE_SOURCE_DIR}/cmake/ver.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/ver.h"
  @ONLY
)
```

### 4. Installation Layout

```cmake
install(TARGETS oppcommon oppsim oppenvir oppcmdenv oppqtenv
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/omnetpp
  DESTINATION include
)

install(FILES cmake/OMNeTppConfig.cmake
  DESTINATION lib/cmake/OMNeTpp
)
```

### 5. Export for External Projects

```cmake
# Allow find_package(OMNeTpp) from external projects
install(EXPORT OMNeTppTargets
  FILE OMNeTppTargets.cmake
  NAMESPACE OMNeTpp::
  DESTINATION lib/cmake/OMNeTpp
)
```

## Build Commands

### Basic Build

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

### Debug Build

```bash
cmake -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build-debug -j$(nproc)
```

### Install

```bash
cmake --install build --prefix /usr/local
```

### With QtEnv Disabled

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release -DWITH_QTENV=OFF
```

## Migration from Makefile

### Variable Mapping

| Makefile Variable | CMake Equivalent |
|-------------------|------------------|
| `MODE=release` | `-DCMAKE_BUILD_TYPE=Release` |
| `MODE=debug` | `-DCMAKE_BUILD_TYPE=Debug` |
| `SHARED_LIBS=yes` | `-DOMNETPP_SHARED_LIBS=ON` |
| `WITH_QTENV=yes` | `-DWITH_QTENV=ON` |
| `V=1` | `cmake --build build -- VERBOSE=1` |

### Output Directory

| Makefile | CMake |
|----------|-------|
| `out/gcc-release/` | `build/` (default) |
| `lib/` | `build/` (in-tree) or install prefix |

## Testing Strategy

1. **Build Parity Tests**: Compare library symbols between Make and CMake builds
2. **Runtime Tests**: Run existing test suite (`make tests`) with CMake-built libraries
3. **Sample Simulations**: Build and run sample simulations with CMake-built `opp_run`

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| MSG compiler integration issues | Medium | High | Prototype early in sim/ subsystem |
| Qt version differences | Low | Medium | Explicitly require Qt >= 6.2 |
| Linker flag differences | Medium | Medium | Compare `ldd` output and rpath |
| Missing optional dependencies | Low | Low | Graceful degradation with feature flags |

## Implementation Phases

### Phase 1: Core Libraries (common, layout, eventlog, scave)
- Estimated: 2-3 days
- Deliverables: 4 CMakeLists.txt files, basic cmake/ modules

### Phase 2: Simulation Kernel (nedxml, sim)
- Estimated: 2-3 days
- Deliverables: MSG compiler integration, sim/ CMakeLists.txt

### Phase 3: Runtime Environment (envir, cmdenv)
- Estimated: 1-2 days
- Deliverables: opp_run executable, cmdenv library

### Phase 4: Qt GUI (qtenv)
- Estimated: 2-3 days
- Deliverables: Qt6 integration, MOC/UIC/RCC automation

### Phase 5: Polish and Testing
- Estimated: 1-2 days
- Deliverables: Install targets, export config, documentation

**Total Estimated Effort**: 8-13 days

## Open Questions

1. Should CMake build output to `out/cmake-release/` for compatibility with existing workflows?
2. Should we generate a `Makefile.inc` equivalent from CMake for `opp_makemake` compatibility?
3. How to handle the `setenv` script with CMake builds?

## References

- Existing Makefile structure: `/workspace/omnetpp/Makefile`
- Configuration template: `/workspace/omnetpp/Makefile.inc.in`
- Subsystem Makefiles: `/workspace/omnetpp/src/*/Makefile`
- AGENTS.md: `/workspace/omnetpp/AGENTS.md`