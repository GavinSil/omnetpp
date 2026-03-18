# test/ — OMNeT++ Test Suite

## Running Tests

```bash
# From repo root:
make tests              # Run all tests

# From test/:
make                    # Run all tests
make test_quick         # Quick subset

# Individual suite:
make test_core          # Kernel tests (750 files)
make test_envir         # Environment tests
make test_common        # Common utility tests
make test_scave_results_scalar   # Scave scalar tests
make test_fingerprint   # Fingerprint-based regression

# Individual test:
cd test/core && ./runtest cModule_creation_1.test
cd test/core && ./runtest   # All tests in directory
```

## Test Suites

| Directory | Tests | What It Covers |
|-----------|-------|----------------|
| `core/` | 750 .test files | Simulation kernel (modules, messages, gates, scheduling, statistics) |
| `common/` | Common utility functions |
| `envir/` | Runtime environment, config |
| `anim/` | Animation and canvas rendering |
| `models/` | Sample model behavior |
| `makemake/` | opp_makemake tool |
| `featuretool/` | opp_featuretool |
| `fingerprint/` | Simulation fingerprint regression |
| `scave/` | Result file processing |
| `sqliteresultfiles/` | SQLite result backends |
| `build/` | Build system tests |
| `distro/` | Distribution packaging |
| `toolchain/` | Compiler toolchain |
| `ide/` | IDE functionality |

## .test File Format

Tests use a custom format processed by `opp_test` (in `src/utils/`):

```
%description:
Brief description of what this test verifies.

%activity:
// C++ code that runs as the module body
EV << "testing something" << endl;
cMessage *msg = new cMessage("test");
scheduleAt(1.0, msg);

%contains: stdout
testing something

%not-contains: stderr
Error

%exitcode: 0
```

### Common Sections

| Section | Purpose |
|---------|---------|
| `%description:` | What the test verifies |
| `%activity:` | C++ code for a simple module's activity |
| `%module:` | Full module class definition |
| `%network:` | NED network definition |
| `%file: <name>` | Additional file to create (NED, ini, etc.) |
| `%contains: <output>` | Expected substring in output |
| `%not-contains: <output>` | Must NOT appear in output |
| `%contains-regex: <output>` | Regex match in output |
| `%exitcode: N` | Expected exit code |
| `%inifile:` | omnetpp.ini content |
| `%subst:` | Text substitution on output before comparison |

### Mechanism
1. `opp_test` extracts code from `.test` file
2. Generates C++ source + NED + ini files
3. Compiles and links against OMNeT++ libs
4. Runs simulation
5. Compares output against `%contains`/`%not-contains` patterns

## Adding New Tests

1. Create `test/<suite>/your_test_name.test`
2. Follow the `.test` format above
3. Run: `cd test/<suite> && ./runtest your_test_name.test`
4. For kernel tests, use `test/core/` — name should describe what's being tested
