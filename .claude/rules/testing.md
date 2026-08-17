---
paths:
  - "test/**"
  - "*.sh"
  - "compare_*.py"
  - "gen_tests.py"
---

# Testing rules

- Test scripts are POSIX shell. On Windows run them through WSL:
  `wsl -d Ubuntu -- bash -lc "./run_tests.sh ray2D gen_ray2D_pass"`.
- Suite naming is `(ray|tl|eigen|arr)(2D|3D|Nx2D)`; pass/fail lists live in
  `*_pass.txt` / `*_fail.txt` at the repo root.
- Output files (`.ray`, `.shd`, `.arr`, `.prt`) are BELLHOP-compatible binary /
  Fortran-unformatted formats. Never hand-edit them; diff with the `compare_*.py`
  utilities.
- Numerical results depend on float mode: `BHC_USE_FLOATS=ON` (and CUDA
  fast-math) changes outputs. Always compare like configuration against like.
- Never commit generated outputs (`sample.*`, `test/dbg/`, `*.prt`); they are
  gitignored.
