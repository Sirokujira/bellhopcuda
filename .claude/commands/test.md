---
description: Run a bellhop test suite and summarize results
argument-hint: <suite> <pass-list>   e.g. ray2D gen_ray2D_pass
---

Run the test suite `$ARGUMENTS` and summarize the outcome.

1. If no arguments were given, list the available suites
   (`(ray|tl|eigen|arr)(2D|3D|Nx2D)`) and the `*_pass.txt` lists in the repo
   root, then ask which to run.
2. The runner is POSIX shell — on Windows execute it via WSL:
   `wsl -d Ubuntu -- bash -lc "./run_tests.sh $ARGUMENTS"`.
   The C++ executables must exist first; if `bin/` is empty, run /build first.
3. Report pass/fail counts. For failures, run the matching `compare_*.py` utility
   on one failing case and include the numerical diff summary.
