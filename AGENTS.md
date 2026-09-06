# FlyDSL

Agent context for the FlyDSL repository (Python DSL + MLIR compiler stack for AMD GPU kernels).

## Project overview

- Languages: Python 3.10+ (DSL), C++17 (MLIR dialects / Python bindings)
- Build system: CMake 3.20+ with MLIR/LLVM out-of-tree, plus Python `setuptools` via `setup.py`.
- Layout: `python/flydsl/` (DSL core), `include/flydsl/` + `lib/` (C++ dialects), `tests/` (kernels, mlir, unit), `examples/`, `kernels/`, `scripts/`.
- The project already has a detailed `CLAUDE.md`; this file supplements it with build/test/lint commands and conventions.

## Prerequisites

- ROCm-aware environment with HIP; a compatible GPU is needed for many tests/examples.
- MLIR/LLVM built or installed out-of-tree so `find_package(MLIR REQUIRED CONFIG)` succeeds.

## Setup commands

Use the provided build script, or build manually:

```bash
# Full build (LLVM + FlyDSL + Python bindings)
./scripts/build.sh

# Or manual CMake configure after MLIR/LLVM is available
mkdir -p build && cd build
cmake .. -DMLIR_DIR=/path/to/llvm/lib/cmake/mlir -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build . -j$(nproc)
cd ..

# Install the Python package in editable/develop mode
python -m pip install -e .
```

## Run / test / lint commands

```bash
# Run all tests
./scripts/run_tests.sh

# Run a single example (requires a working build + ROCm runtime)
python examples/01-vectorAdd.py

# Run benchmarks
./scripts/run_benchmark.sh

# Format Python code
black python/ tests/

# Lint Python code
ruff check python/ tests/

# C++ formatting (project uses .clang-format)
find include/ lib/ tools/ -name '*.cpp' -o -name '*.h' | xargs clang-format -i
```

## Key conventions

- Python formatter: `black`, line length 120.
- Python linter: `ruff` with `target-version = "py10"`, selecting `E`, `W`, `F`, `I`; ignoring `E501`, `F403`, `F405`, `E731`.
- First-party imports are known as `flydsl`.
- C++: C++17, `clang-format` governed by `.clang-format`.
- Kernel sources under `kernels/` are importable as `kernels.*`.
- `python/flydsl/expr/rocdl/` is target-specific and lazy-loaded; `python/flydsl/expr/` children are target-neutral.

## Important gotchas

- The CMake build requires an out-of-tree MLIR/LLVM installation; `build`/`build_*`/`build-fly`/`thirdparty` directories are excluded from lint/format.
- `setup.py` invokes CMake configuration; `pyproject.toml` build-system requires `pybind11`, `nanobind`, `numpy`, and `cmake`.
- Examples and kernel tests require a real ROCm/HIP device; unit/MLIR tests may run in CPU-only environments.
- Do not add target-specific code under `python/flydsl/expr/` directly; place ROCm-specific abstractions in `python/flydsl/expr/rocdl/`.

## Useful shortcuts

```bash
# Quick smoke test after building
python examples/01-vectorAdd.py

# Inspect a generated MLIR kernel
python -m flydsl.compiler.jit_function --help
```
