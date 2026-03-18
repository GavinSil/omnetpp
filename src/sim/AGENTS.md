# src/sim/ — Simulation Kernel

The core discrete event simulation engine. Everything else in OMNeT++ depends on this.

## Architecture

### Component Model
- `cModule` (base) → `cSimpleModule` (behavioral) / compound modules (structural)
- `cGate` — Module ports for connections
- `cChannel` / `cDatarateChannel` — Connection modeling
- `cMessage` / `cPacket` — Events and data units
- `cSimulation` — Simulation lifecycle and event loop

### Event Scheduling
- Future Event Set (FES): `cFutureEventSet` → `cEventHeap`
- `simtime_t` (fixed-point time) — always initialize to zero
- `scheduleAt()`, `scheduleAfter()`, `cancelEvent()`

### Statistics
- `cResultRecorder` / `cResultFilter` — Signal-based statistics
- `cStatistic`, `cStdDev`, `cHistogram` — Statistical collectors
- `@statistic` NED property → auto-generated recorders

## Key Patterns

### Module Registration
```cpp
// In .cc file — registers module class with simulation kernel
Define_Module(MyModule);
Register_Class(MyClass);        // For non-module cOwnedObject subclasses
```

### Module Lifecycle
```cpp
class MyModule : public cSimpleModule
{
  protected:
    void initialize() override;           // Always override
    void handleMessage(cMessage *msg) override;  // Always override
    void finish() override;               // Optional cleanup
};
```

### Parameter Access
```cpp
// CORRECT — use implicit conversion operators
int count = par("count");
double rate = par("rate");
std::string name = par("name").stdstringValue();  // String → std::string

// WRONG — don't use explicit value methods
int count = par("count").intValue();  // Avoid
```

### check_and_cast
```cpp
// CORRECT — throws on failure, never returns nullptr
auto *pkt = check_and_cast<cPacket *>(msg);
pkt->doSomething();  // Safe, no nullptr check needed

// WRONG — redundant nullptr check
auto *pkt = check_and_cast<cPacket *>(msg);
if (pkt != nullptr)  // Unnecessary — check_and_cast never returns nullptr
    pkt->doSomething();
```

## Subdirectories

- `netbuilder/` — Dynamic NED loading (builds network from NED at runtime)
- `parsim/` — Parallel/distributed simulation (39 files): MPI, named pipes, file-based communication

## Dependencies

- Depends on: `common/`, `nedxml/`
- Depended on by: `envir/`, and transitively by `cmdenv/`, `qtenv/`
- Public headers: `include/omnetpp/` (123 files)

## Public API Export

All public classes use `SIM_API` macro:
```cpp
class SIM_API cModule : public cComponent { ... };
```

Headers use `#ifndef __OMNETPP_CMODULE_H` include guards and Doxygen `@brief`/`@ingroup` documentation.
