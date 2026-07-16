# to-rust 🐍 -> 🦀

`to-rust` is a hybrid Python-to-Rust JIT/AOT transpiler that automatically rewrites, compiles, and loads your Python code into highly optimized native Rust binaries.

With a simple `@to_rust` decorator, you can accelerate heavy CPU-bound Python functions by $10\times$ to $100\times$ (so far in theory). 

---

## The Core Philosophy: Performance with Zero Friction

Existing tools like Numba or Cython either fail on complex Python logic or force you to learn a new syntax. `to-rust` takes a different path:

1. **Strict Type-Hints**: If you type-hint it, we can transpile it.
2. **Deep-Copy by Default**: We completely bypass the strict Rust Borrow Checker by cloning data on entry. A minor memory allocation trade-off for 100% compilation stability. This is currently under investigation for better work-flow and minimal memory usage.
3. **Hybrid LLM Fallback (The Magic)**: If our deterministic AST parser hits a structure it doesn't understand (like complex `try/except` blocks), it calls a lightweight local LLM (**Qwen 3.5/3.6** via Ollama) to fill in the gaps and fix the Rust code on the fly.
4. **Dev vs. Runtime Split**: Zero heavy dependencies in production. Compile once in development, run anywhere in production.

---

## Installation

To keep production containers lightweight, `to-rust` is divided into two install targets:

### Production / Client (Runtime)
Installs only the loader. Zero heavy dependencies. No Rust, Cargo, or LLM required.

`pip install to-rust`

### Development / Builder (Full)
Installs the AST parser, PyO3 generator, compiler bridges, and LLM orchestration tools.
`pip install to-rust[dev]`

## Quick Start
Write pure Python with standard type hints. The first run compiles the code on the fly; subsequent runs execute at native Rust speed.
```
from to_rust import to_rust

@to_rust(export_path="./rust_cache")
def heavy_calculation(items: list[int], multiplier: int) -> int:
    result: int = 0
    for item in items:
        if item % 2 == 0:
            result += item * multiplier
    return result
```

# 1st execution: AST translation -> Cargo Build -> Dynamic Load (~5-10s)
# 2nd execution: Direct native binary execution (~microseconds!)
print(heavy_calculation([1, 2, 3, 4, 5], 10))

## Architecture: Dev vs. Prod Workflow

[ Your Python Code ]
         │
         ▼
 ┌───────────────┐
 │   @to_rust    │
 └───────┬───────┘
         │
         ├─► [ DEV Mode ] ────► AST Parsing ──► [ Fail? ] ──► Qwen LLM Fallback
         │                                         │
         │                                         ▼
         │                                    Cargo Build (PyO3)
         │                                         │
         │                                         ▼
         │                                   .so / .pyd Binary
         │                                         │
         └─► [ PROD Mode ] ◄───────────────────────┘
                 │
                 ▼
          importlib.import()
                 │
                 ▼
        [ Native Execution ]

## Roadmap & TODO List
This project is actively developed. Below is our path towards a stable 1.0.0 release.
### 🟩 Phase 1: The MVP (Current Goal)
 * [ ] *Core Decorator*: Establish the basic @to_rust wrapper with SHA-256 caching.
 * [ ] *Dynamic Loader*: Implement runtime .so/.pyd importing via importlib.
 * [ ] *Basic AST Visitor*: Transpile simple math, if/else, and for loops.
 * [ ] *Primitive Type Mapping*: Map basic types (int -> i64, float -> f64, str -> String).
 * [ ] *Deep-Copy Generation*: Automatically inject .clone() on all incoming variables to satisfy the borrow checker.
### 🟨 Phase 2: CLI & Dev Tooling
 * [ ] *Separate Poetry Targets*: Finalize the [dev] extras partition in pyproject.toml.
 * [ ] *Pre-compilation CLI*: Add a CLI command to-rust compile to pre-build all decorated functions in a project directory for CI/CD pipelines.
 * [ ] *Platform Cross-Compilation*: Support building Linux binaries from macOS/Windows host machines via Docker.
### 🟧 Phase 3: Hybrid LLM Fallback
 * [ ] *Ollama Connector*: Implement a lightweight local client looking for localhost:11434.
 * [ ] *FIM (Fill-In-the-Middle) Prompt Engine*: Design precise system prompts optimized for *Qwen 3.5 / 3.6* to inject missing Rust code block repairs.
 * [ ] *Self-Healing Compiler Loop*: Capture cargo build error logs, feed them back to Qwen, and retry compilation up to 3 times.
### 🟥 Phase 4: Advanced Python Features
 * [ ] *Structured Exceptions*: Translate basic Python try/except chains to Rust Result<T, E>.
 * [ ] *Async & Concurrency*: Map simple asyncio routines using pyo3-asyncio and Tokio.
 * [ ] *Custom Classes*: Allow decorating entire classes, mapping Python attributes to Rust structs.
## Contributing
Contributions are welcome! If you want to help us bridge the gap between Python's developer speed and Rust's raw execution power:
 1. Fork the Project.
 2. Create your Feature Branch (git checkout -b feature/AmazingFeature).
 3. Commit your Changes (git commit -m 'Add some AmazingFeature').
 4. Push to the Branch (git push origin feature/AmazingFeature).
 5. Open a Pull Request.
## 📄 License
Distributed under the *Apache License 2.0*. See LICENSE for more information.
