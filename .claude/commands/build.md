---
description: Configure and build the C++-only targets (CUDA off)
---

Build the project C++-only and report the result.

1. Pick the toolchain for the current platform:
   - Windows: use `cmake` from PATH if available; otherwise use the CMake bundled
     with Visual Studio / Build Tools (locate it with
     `vswhere -latest -find **\cmake.exe`) with `-G "Visual Studio 17 2022" -A x64`,
     or build inside WSL.
   - Linux/macOS: use `cmake` from PATH.
2. Configure into a fresh out-of-tree directory (NOT the checked-in `build/`,
   which may hold another platform's cache):
   `cmake -S . -B <builddir> -DBHC_ENABLE_CUDA=OFF -DBHC_BUILD_EXAMPLES=OFF`
3. Build Release with parallelism (`cmake --build <builddir> --config Release -j`).
4. If it fails, show the first error with file/line and stop; otherwise confirm the
   four executables and the libraries exist in `bin/`.
