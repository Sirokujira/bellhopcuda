# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**bellhopcxx / bellhopcuda** is a C++17/CUDA port of the BELLHOP/BELLHOP3D underwater acoustics ray-tracing simulator. It computes sound propagation in ocean environments using ray acoustics, supporting 2D, 3D, and Nx2D (range-dependent) simulations. The same codebase compiles to multithreaded CPU (bellhopcxx) or NVIDIA GPU (bellhopcuda) executables, and also as a shared/static library.

## Build

Requires CMake ≥ 3.15. The `glm` submodule must be populated (`git clone --recursive` or `git submodule update --init`).

```bash
mkdir build && cd build
cmake ..                         # CUDA enabled by default; disable with -DBHC_ENABLE_CUDA=OFF
cmake --build .
```

Or set the environment variable `BHC_NO_CUDA=1` to skip the CUDA build without changing CMake flags.

Windows note: if `cmake` is not on PATH, use the CMake bundled with Visual Studio / Build Tools — run from a "Developer PowerShell for VS 2022" environment (which puts it on PATH), locate it with `vswhere -latest -find **\cmake.exe`, or build inside WSL. The existing `build/` directory may contain another platform's CMake cache — always configure into a fresh out-of-tree directory.

Output binaries go to `bin/`:
- Executables: `bellhopcxx`, `bellhopcxx2d`, `bellhopcxx3d`, `bellhopcxxnx2d` (and `bellhopcuda*` variants)
- Libraries: `libbellhopcxxlib.so` / `.dylib` / `.dll` and `libbellhopcxxstatic.a` / `.lib`

Key CMake options:
| Option | Default | Description |
|--------|---------|-------------|
| `BHC_ENABLE_CUDA` | ON | Build CUDA version |
| `BHC_USE_FLOATS` | OFF | Use 32-bit floats (default is double) |
| `BHC_LIMIT_FEATURES` | OFF | Restrict to features in original BELLHOP/BELLHOP3D |
| `BHC_DEBUG` | OFF | Enable debug features (uses RelWithDebInfo) |
| `BHC_DIM_ENABLE_2D/3D/NX2D` | ON | Enable dimensionalities |
| `BHC_RUN_ENABLE_TL/EIGENRAYS/ARRIVALS` | ON | Enable run types |
| `CUDA_ALL_ARCHES` | OFF | Build for all GPUs, not just newest detected |

## Running Tests

```bash
# Run a specific test suite (format: (ray/tl/eigen/arr)(2D/3D/Nx2D))
./run_tests.sh ray2D gen_ray2D_pass

# Run all generated tests
./run_all_gen.sh

# Regenerate test cases
python3 gen_tests.py
```

Comparison utilities in the root: `compare_ray_2.py`, `compare_shdfil.py`, `compare_arrivals.py`, `compare_floats.py`.

Test input files are in `test/in/`. Pass/fail result lists are in text files like `ray_3d_pass.txt`, `tl_long.txt` in the root.

## Code Formatting

clang-format is enforced via a git pre-commit hook (installed automatically by CMake). Column limit is 90. To format manually:

```bash
clang-format -i src/somefile.cpp   # format in place
clang-format --dry-run src/somefile.cpp  # check only
```

## Architecture

### Template Metaprogramming Pattern

The entire codebase is templated on two boolean parameters:
- `O3D` — ocean is 3D (true for 3D and Nx2D runs, false for 2D)
- `R3D` — ray is 3D (true only for full 3D runs)

This enables compile-time specialization with a single codebase. Most functions are declared `HOST_DEVICE` so they compile for both CPU and GPU.

### Public API (`include/bhc/`)

- `bhc.hpp` — primary public API: `bhc::setup()`, `bhc::run()`, `bhc::writeout()`, `bhc::finalize()`
- `structs.hpp` — `bhcParams<O3D, R3D>` and `bhcOutputs<O3D, R3D>` hold all state (no global variables)
- `platform.hpp` — `HOST_DEVICE`, `BHC_DLL_IMPORT/EXPORT` macros

### Source Structure (`src/`)

**Simulation modes** (`src/mode/`): `ray`, `tl`, `eigen`, `arr` — each implements the top-level loop for its run type. `field.hpp` is the parent class for TL/eigenrays/arrivals.

**Parameter modules** (`src/module/`): 21 modules, each reads one section of the `.env` input file — `ssp.hpp`, `boundary.hpp`, `atten.hpp`, `rayangles.hpp`, `rcvrranges.hpp`, `beaminfo.hpp`, etc. All inherit from `ParamsModule`.

**Core physics** (large files in `src/`):
- `step.hpp` — ray stepping algorithm (advances ray one step)
- `trace.hpp` — orchestrates the full ray trace
- `influence.hpp` — beam/Gaussian influence accumulation onto receiver grid
- `ssp.hpp` — sound speed profile interpolation (N2-linear, C-linear, cubic, PCHIP, quad, hexahedral, analytic)
- `boundary.hpp` — ocean surface/bottom boundary interactions
- `reflect.hpp` — reflection coefficient calculations

**Utilities** (`src/util/`): error handling, timing, binary/Fortran-unformatted I/O, CUDA atomics.

### Data Flow

1. `bhc::setup()` — reads `.env` file; each parameter module processes its section; validates and preprocesses all parameters
2. `bhc::run()` — dispatches to the selected mode; for each ray: `step.hpp` advances the trajectory, `influence.hpp` accumulates field contributions
3. `bhc::writeout()` — writes `.ray`, `.shd` (shade/TL), or `.arr` files in BELLHOP-compatible format
4. `bhc::finalize()` — frees memory

### Template Generation

`config/GenTemplates.cmake` generates `.cpp`/`.cu` source files from `.cpp.in`/`.cu.in` templates, producing one translation unit per (run type × influence type × SSP type) combination to control compilation.

### Library Usage

To embed bellhopcxxlib: copy `include/bhc/` into your project, set C++17, and link `libbellhopcxxlib.so` (or `.a`). On Windows DLL, define `BHC_DLL_IMPORT` before including the header.

## Coding Rules

Domain rules live in `.claude/rules/` and are loaded below. Slash commands for
common workflows: `/build` (C++-only build), `/test <suite> <pass-list>`.

@.claude/rules/cpp.md
@.claude/rules/cmake.md
@.claude/rules/testing.md
