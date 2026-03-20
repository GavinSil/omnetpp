# OMNeT++ CMake Build System Design

**Date**: 2026-03-20
**Author**: AI Agent
**Status**: Draft (Revised v2)

## Overview

Add CMake build support to OMNeT++ 6.4.0 as a parallel build system alongside the existing Autoconf/Makefile system. Target: modern developer experience (IDE integration, ninja, CLion/VS Code CMake Tools support).

## Goals

- **Primary**: Enable modern build toolchain (ninja, IDE integration)
- **Scope**: Complete developer runtime (libraries + tools + executables)
- **Platform**: Linux only (initial implementation)
- **Coexistence**: CMake and Makefile run in parallel, user selects

## Deliverables (Complete Developer Runtime)

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
| opp_charttool | `OMNeTpp::opp_charttool` | Chart generation (requires WITH_SCAVE_PYTHON_BINDINGS) |
| opp_configfilepath | `OMNeTpp::opp_configfilepath` | Returns path to Makefile.inc |

### Excluded from Scope
- Python bindings (`src/scave/python/`)
- JNI native libs (`ui/org.omnetpp.ide.nativelibs/`)
- IDE plugins (`ui/` Maven/Tycho build)

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
option(WITH_SCAVE_PYTHON_BINDINGS "Enable scave Python bindings" ON)

# Build options
option(OMNETPP_SHARED_LIBS "Build shared libraries" ON)
option(PREFER_SQLITE_RESULT_FILES "Use SQLite as default result format" OFF)
```

**Note**: `opp_charttool` requires `WITH_SCAVE_PYTHON_BINDINGS=ON`. If disabled, `opp_charttool` will not be built.

### Build Types (Simplified)

| CMAKE_BUILD_TYPE | Equivalent MODE | Output Suffix |
|-------------------|-----------------|---------------|
| Release | release | (none) |
| Debug | debug | `_dbg` |

## Generated Artifacts (Complete Inventory)

### src/common/
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `expression.y` | `expression.tab.cc`, `expression.tab.h` | `BISON_TARGET()` |
| `expression.lex` | `expression.lex.cc` | `FLEX_TARGET()` |
| `matchexpression.y` | `matchexpression.tab.cc`, `matchexpression.tab.h` | `BISON_TARGET()` |
| `matchexpression.lex` | `matchexpression.lex.cc` | `FLEX_TARGET()` |

### src/nedxml/
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `ned2.y` | `ned2.tab.cc`, `ned2.tab.h` | `BISON_TARGET()` |
| `ned2.lex` | `ned2.lex.cc` | `FLEX_TARGET()` |
| `msg2.y` | `msg2.tab.cc`, `msg2.tab.h` | `BISON_TARGET()` |
| `msg2.lex` | `msg2.lex.cc` | `FLEX_TARGET()` |
| `dtdclassgen.pl` | `dtdvalidationclasses.cc/h` | `add_custom_command()` |

### src/sim/
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `sim_std.msg` | `sim_std_m.cc`, `sim_std_m.h` | `opp_msgtool` |

### src/envir/
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `eventlogwriter.pl` | `eventlogwriter.cc`, `eventlogwriter.h` | `add_custom_command()` |

### src/eventlog/
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `eventlogentries.msg` | `eventlogentries_m.cc/h` | `opp_msgtool` |

### src/qtenv/
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `*.ui` | `ui_*.h` | `AUTOUIC` |
| `*.h` (Q_OBJECT) | `moc_*.cpp` | `AUTOMOC` |
| `*.qrc` | `qrc_*.cpp` | `AUTORCC` |
| `icons.qrc` + dark SVG generation | `icons_dark.qrc` | `add_custom_command()` |

### src/qtenv/osg/ (optional, WITH_OSG)
| Source | Generated Files | CMake Mechanism |
|--------|-----------------|-----------------|
| `osg.msg` | `osg_m.cc/h` | `opp_msgtool` |

## Bootstrap Order

```
Phase 1: Build code generators
  └─ opp_nedtool, opp_msgtool (from src/nedxml/)

Phase 2: Generate sources using tools from Phase 1
  └─ MSG files → *_m.cc via opp_msgtool

Phase 3: Build libraries with generated sources
  └─ sim → envir → cmdenv/qtenv
```

CMake handles this via target dependencies:
```cmake
add_custom_command(
  OUTPUT sim_std_m.cc
  COMMAND opp_msgtool --msg6 sim_std.msg
  DEPENDS opp_msgtool sim_std.msg
)
```

## Compatibility Layer (Critical for opp_makemake)

### Rule: CMake builds must generate shell-compatible artifacts

A CMake build MUST produce:
1. **Shell `setenv` script** in build tree (not just on install)
2. **`Makefile.inc`** in build tree (for `opp_makemake`)
3. **`opp_configfilepath`** executable that returns the Makefile.inc path

### setenv Script

Generate a shell script, not a CMake file:

```bash
# build/setenv (generated by CMake)
export OMNETPP_ROOT="/workspace/omnetpp"
export OMNETPP_IMAGE_PATH="$OMNETPP_ROOT/images"
export PATH="$OMNETPP_ROOT/build/bin:$PATH"
export LD_LIBRARY_PATH="$OMNETPP_ROOT/build/lib:$LD_LIBRARY_PATH"
```

Generated by CMake:
```cmake
configure_file(
  "${CMAKE_SOURCE_DIR}/cmake/setenv.in"
  "${CMAKE_BINARY_DIR}/setenv"
  @ONLY
)
```

### Makefile.inc Generation

Generate in build tree immediately after CMake configuration:

```cmake
# Generate Makefile.inc for opp_makemake compatibility
configure_file(
  "${CMAKE_SOURCE_DIR}/cmake/Makefile.inc.in"
  "${CMAKE_BINARY_DIR}/Makefile.inc"
  @ONLY
)
```

### opp_configfilepath

Must return the actual Makefile.inc file path:

```cpp
// src/utils/opp_configfilepath.cc
int main() {
    std::cout << CMAKE_BINARY_DIR << "/Makefile.inc" << std::endl;
    return 0;
}
```

Generated at configure time to embed the path:
```cmake
configure_file(
  "${CMAKE_SOURCE_DIR}/src/utils/opp_configfilepath.cc.in"
  "${CMAKE_BINARY_DIR}/opp_configfilepath.cc"
  @ONLY
)
add_executable(opp_configfilepath "${CMAKE_BINARY_DIR}/opp_configfilepath.cc")
```

### Verification

The acceptance test "Sample Build" can now work:
```bash
cmake -B build
cmake --build build
source build/setenv
cd samples/tictoc
opp_makemake --deep  # Uses opp_configfilepath → build/Makefile.inc
make
```

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
  DESTINATION lib/cmake/OMNeTpp
)

# Install compatibility files
install(FILES "${CMAKE_BINARY_DIR}/setenv"
  DESTINATION .
  PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
)
install(FILES "${CMAKE_BINARY_DIR}/Makefile.inc"
  DESTINATION .
)
```

## Build Commands

### Basic Build
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
source build/setenv
```

### Debug Build
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j$(nproc)
source build/setenv
```

### With Features Disabled
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release \
  -DWITH_QTENV=OFF \
  -DWITH_SCAVE_PYTHON_BINDINGS=OFF
```

## Testing Strategy

### Acceptance Criteria

| Test | Pass Criteria |
|------|---------------|
| **Configure** | `cmake -B build` succeeds |
| **Build** | `cmake --build build` completes with 0 errors |
| **setenv** | `source build/setenv` succeeds, PATH updated |
| **Library Inventory** | All 9 libraries built |
| **Tool Inventory** | All tools built and executable |
| **opp_run Test** | `opp_run -h` succeeds |
| **opp_configfilepath** | Returns `build/Makefile.inc` |
| **opp_makemake Test** | Build a sample simulation with opp_makemake |
| **Sample Run** | Run the sample simulation successfully |

### Test Matrix
```bash
# Minimum configurations to test
cmake -B build-release -DCMAKE_BUILD_TYPE=Release
cmake -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake -B build-noqt -DCMAKE_BUILD_TYPE=Release -DWITH_QTENV=OFF
cmake -B build-nopython -DCMAKE_BUILD_TYPE=Release -DWITH_SCAVE_PYTHON_BINDINGS=OFF
```

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Bootstrap ordering issues | Medium | High | CMake target dependencies |
| MSG tool not found | Low | High | Build tool first with DEPENDS |
| Qt version incompatibility | Low | Medium | Require Qt >= 6.2 |
| Makefile.inc drift | Medium | Medium | Generate from CMake variables |
| opp_makemake incompatibility | Medium | High | Test with sample projects |

## Implementation Phases

### Phase 1: Infrastructure (1-2 days)
- Root CMakeLists.txt
- cmake/ modules
- Compatibility layer: setenv, Makefile.inc, opp_configfilepath

### Phase 2: Core Libraries (2-3 days)
- src/common/ with flex/bison
- src/layout/
- src/eventlog/ with MSG
- src/scave/

### Phase 3: Tools and NED/XML (2-3 days)
- src/nedxml/ with flex/bison
- opp_nedtool, opp_msgtool

### Phase 4: Simulation Kernel (2-3 days)
- src/sim/ with MSG
- Generated sim_std_m.cc

### Phase 5: Runtime Environment (1-2 days)
- src/envir/ with eventlogwriter
- src/cmdenv/
- opp_run

### Phase 6: Qt GUI (2-3 days)
- src/qtenv/ with AUTOMOC/UIC/RCC
- OSG support (optional)

### Phase 7: Utils (1 day)
- src/utils/
- opp_makemake, opp_test, etc.

### Phase 8: Testing (1-2 days)
- All acceptance criteria
- Sample simulation tests

**Total Estimated Effort**: 12-18 days

## Decisions Summary

| Question | Decision |
|----------|----------|
| Output directory | `build/` default, user can change via `-B` |
| Makefile.inc | Generate in build tree immediately |
| setenv | Generate shell script in build tree |
| opp_configfilepath | Return full path to Makefile.inc |
| opp_charttool | Built only when WITH_SCAVE_PYTHON_BINDINGS=ON |

## References

- Existing Makefile: `/workspace/omnetpp/Makefile`
- Makefile.inc.in: `/workspace/omnetpp/Makefile.inc.in`
- configure.user.dist: `/workspace/omnetpp/configure.user.dist`
- AGENTS.md: `/workspace/omnetpp/AGENTS.md`