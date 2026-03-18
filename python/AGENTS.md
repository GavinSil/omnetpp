# python/ — OMNeT++ Python Bindings and Tools

## Package Structure

```
python/omnetpp/
├── scave/          Result analysis and plotting (main Python API)
├── llmtool/        LLM integration tools
│   ├── cpp-coding-guidelines.txt   C++ coding style reference (376 lines)
│   ├── cpp-improve.txt             Code improvement prompt template
│   └── patch-format.txt            SEARCH/REPLACE patch format spec
├── lldb/           LLDB debugger integration
├── ned.py          NED file handling
├── nedast.py       NED AST representation
├── nedlinter.py    NED linting rules
├── repl.py         Interactive REPL
└── test.py         Test utilities
```

## Key Components

### scave/ — Result Analysis
The primary Python API for processing simulation results (`.vec`, `.sca` files). Used by the IDE's analysis tool and `opp_charttool`.

### llmtool/ — LLM Coding Guidance
Contains the authoritative C++ coding guidelines for this project:
- `cpp-coding-guidelines.txt` — **The definitive style guide** (376 lines). All C++ changes should conform to this.
- `cpp-improve.txt` — Template for LLM code improvement prompts
- `patch-format.txt` — SEARCH/REPLACE block format for suggesting code changes

### NED Tools
- `ned.py` / `nedast.py` — NED file parsing and AST
- `nedlinter.py` — NED style checking

## Related Tools
- `opp_charttool` (`src/utils/`) — Chart generation, uses scave/
- `opp_neddoc` (`src/utils/`) — NED documentation generator
