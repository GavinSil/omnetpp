# ui/ — OMNeT++ IDE (Eclipse-based)

## Overview

The OMNeT++ IDE is built on Eclipse with 30+ custom plugins. All Java code lives here.

## Structure

```
ui/
├── org.omnetpp.main/          Main entry point, branding
├── org.omnetpp.common/        Shared UI utilities and widgets
├── org.omnetpp.common.core/   Core (non-UI) shared utilities
├── org.omnetpp.ned.core/      NED parsing, validation, model (headless)
├── org.omnetpp.ned.model/     NED model interfaces
├── org.omnetpp.ned.editor/    Graphical + text NED editor
├── org.omnetpp.neddoc/        NED documentation generator
├── org.omnetpp.launch/        Simulation launch configurations
├── org.omnetpp.cdt/           CDT (C/C++) integration
├── org.omnetpp.scave/         Analysis tool UI
├── org.omnetpp.scave.model/   Scave data model
├── org.omnetpp.scave.builder/  Result file indexer
├── org.omnetpp.scave.pychart/ Python chart support
├── org.omnetpp.inifile.editor/ INI file editor
├── org.omnetpp.msg.editor/    MSG file editor
├── org.omnetpp.eventlogtable/ Event log viewer
├── org.omnetpp.sequencechart/ Sequence chart visualization
├── org.omnetpp.ide.nativelibs/ Native library loader
├── org.omnetpp.python/        Python integration
├── pom.xml                    Maven build (Tycho)
└── releng/                    Release engineering
```

## Build

```bash
make ui          # Build native libs for IDE (from repo root)
# Full IDE build uses Maven/Tycho:
cd ui && mvn clean verify
```

## Plugin Architecture

Each plugin follows Eclipse conventions:
- `plugin.xml` — Extension points
- `META-INF/MANIFEST.MF` — Bundle metadata, dependencies
- `src/` — Java source
- `icons/` — UI icons

### JNI Bridge
`org.omnetpp.ide.nativelibs` loads C++ libraries into the Java IDE:
- `org.omnetpp.ned.model` — JNI for NED model
- Platform-specific variants: `.linux.x86_64`, `.macosx`, `.win32.x86_64`

## Key Plugin Groups

| Group | Plugins | Purpose |
|-------|---------|---------|
| Core | main, common, common.core, nativelibs | Foundation, shared utilities |
| NED | ned.core, ned.model, ned.editor, neddoc | NED language support |
| Analysis | scave, scave.model, scave.builder, scave.pychart, scave.templates | Result analysis |
| Editors | inifile.editor, msg.editor, eventlogtable, sequencechart, figures | File type editors |
| Launch | launch, cdt, dsp | Simulation running/debugging |

> See `.agent/rules/architecture.md` for the complete plugin table with detailed descriptions.
