---
paths:
  - "src/**/*.cpp"
  - "src/**/*.hpp"
  - "src/**/*.cu"
  - "include/**/*.hpp"
---

# C++ / CUDA rules

- Format with clang-format, column limit 90 (enforced by the pre-commit hook that
  CMake installs). If clang-format is unavailable locally, keep lines <= 90 by hand
  and say so in the PR.
- Simulation code is templated on `<bool O3D, bool R3D>`. New shared functions must
  be `HOST_DEVICE` and must compile under g++, MSVC, and NVCC.
- `HOST_DEVICE` code must be GPU-safe: no exceptions, no STL containers or
  std::string, no iostream, no heap allocation. Report errors with
  `RunError(errState, BHC_ERR_...)`; call `ResetErrState` before first use of a
  local `ErrState`. Host-only code may use `EXTERR` / `EXTWARN`.
- Use `real` (double, or float when `BHC_USE_FLOATS`), `FL(...)` literals, and
  `VEC23<R3D>` vectors. Avoid bare double literals in device code paths.
- Allocate simulation arrays with `trackallocate` / `trackdeallocate` (they account
  against the memory budget). Host-only owned objects use `std::unique_ptr`
  (see src/api.cpp) so exception paths do not leak.
- Promote source/receiver grid index arithmetic to `size_t` before multiplying
  (pattern: `GetFieldAddr` in src/common.hpp) — 32-bit overflow is a real bug class
  here.
- New files keep the GPL v3 header block used across the codebase.
