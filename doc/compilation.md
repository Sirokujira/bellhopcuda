# Compilation of bellhopcxx / bellhopcuda

We use a CMake-based build system. The following instructions assume you have
[CMake >=3.27 installed](https://cmake.org/install/) and available in your system PATH.
Build compiles and runs on Linux and Windows, and we recommend the most recent compilers with
C++17 support.

### Building bellhopcxx (CPU only)
To build the CPU-only version, `bellhopcxx`, follow these steps:

1. Clone the repository:
   ```bash
   git clone https://github.com/A-New-BellHope/bellhopcuda.git
   cd bellhopcuda
   git submodule update --init
   git submodule update --recursive
   ```
2. Create a build directory and navigate into it:
   ```bash
   mkdir build
   cd build
   cmake .. -DBHC_ENABLE_CUDA=OFF
   cmake --build .
   ``` 

On Windows using Visual Studio 2022 or later, you can open the folder in Visual Studio directly after
cloning, and it will automatically configure the project.

On macOS (Apple Silicon or Intel), the Command Line Tools compiler (AppleClang) is enough:

```bash
xcode-select --install   # if not already installed
cmake -B build -DBHC_ENABLE_CUDA=OFF
cmake --build build -j"$(sysctl -n hw.ncpu)"
```

CUDA is not available on macOS, so `-DBHC_ENABLE_CUDA=OFF` is the only supported
configuration there. (The CUDA subdirectory is skipped automatically on Apple platforms,
but passing the option keeps the configure output clean.)

### Building bellhopcuda (with CUDA support)
To build the CUDA-enabled version, `bellhopcuda`, ensure you have the
[NVIDIA CUDA Toolkit installed](https://developer.nvidia.com/cuda-downloads) and
available in your system PATH. Then follow the steps above , but omit the
`-DBHC_ENABLE_CUDA=OFF` option in the `cmake` command.

We have tested on many GPUs, including consumer models from the 20xx, 30xx, and 40xx series, server
GPUs A6000, A100, and GH200. We recommend using the latext version of CUDA and commonly compile
on CUDA versions 12.4, and 12.8, and 13.1.

Both compilation paths will produce a set of executables and libraries in a bin directory, with the
CUDA-enabled version (bellhopcuda*) having additional GPU support. Note that building with CUDA
is hardware specific; ensure your GPU is compatible with the CUDA version you have installed.

### Dependencies

Apart from a C++17 compiler and CMake, the CPU build needs only the system threads
library and the bundled `glm` submodule. It does **not** use OpenMP, LAPACK/LAPACKE,
HDF5, or MPI — multithreading is done with `std::thread`. Passing `-DOpenMP_*`,
`-DLAPACKE_*`, `-DUSE_HDF5`, `-DWITH_MPI` or similar has no effect; CMake reports
them under "Manually-specified variables were not used by the project".

All build options this project understands are prefixed `BHC_` (plus
`CUDA_ALL_ARCHES`); see the `option(...)` lines in the top-level `CMakeLists.txt`,
or list them with `cmake -B build -LH | grep BHC_`. In particular, CUDA is toggled
with `-DBHC_ENABLE_CUDA=OFF`, not `-DWITH_CUDA=OFF`.

### Troubleshooting

**`fatal error: 'glm/common.hpp' file not found`**

The `glm/` directory is empty because the repository was cloned without its
submodules. Run:

```bash
git submodule update --init --recursive
```

in the repository root and re-run CMake. Configuring does this for you when the
checkout is a git working tree and `git` is on the PATH; set
`-DBHC_GIT_SUBMODULE=OFF` to disable that and manage the submodule yourself.
Note that a stale build directory keeps the old compiler command lines, so
re-running `cmake --build build` after fixing the submodule is enough — no need
to delete `build/`.
