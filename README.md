HPM is a performance modeling framework targeted at applications that utilize
GPUs to accelerate computation and MPI for distributed memory communication.
With improvements to and integration of existing projects, it aims to achieve
both high prediction accuracy and ease of use for creating performance models
for any distributed GPU application; the user should not have to inspect the
code or be knowledgeable about the internals of the target application.

It builds on [CODES](https://github.com/minitu/codes) to simulate MPI
communication traces and
[gpuroofperf-toolkit](https://github.com/minitu/gpuroofperf-toolkit) to predict
GPU kernel performance. CODES requires
[DUMPI](https://github.com/minitu/sst-dumpi) to generate and read MPI traces,
and [ROSS](https://github.com/ROSS-org/ROSS) as the parallel discrete event
simulation engine.

## Installation

When cloning this repository, you should make sure to obtain all the submodules
as well. In other words, use

```
$ git clone --recurse-submodules https://github.com/minitu/hpm
```

for cloning. If you forgot to use the `--recurse-submodules` flag, you can run

```
$ git submodule init
$ git submodule update
```

to initialize and update the submodules separately.

IBM XL C/C++ 16.1.1 was used to compile DUMPI, GCC 4.9.3 for ROSS and CODES
(CODES fails at link time with IBM XL compilers), and CMake 3.14.5.

**Before proceeding, set the environment variable** `HPM_PATH` **to point to
the directory where the HPM repository is located.**

### 1. CODES

DUMPI and ROSS should be installed before building CODES.

#### 1-1. DUMPI

The MPI-Replay module in CODES reads DUMPI traces to simulate MPI communication.
To build DUMPI:

```
$ cd $HPM_PATH/sst-dumpi
$ ./bootstrap.sh
$ mkdir build install
$ cd build
$ ../configure --enable-libdumpi --enable-libundumpi CC=mpicc CXX=mpicxx --prefix=$HPM_PATH/sst-dumpi/install
$ make && make install
```

#### 1-2. ROSS

ROSS serves as the simulation engine to process events created by CODES in
parallel. It can be installed as follows:

```
$ cd $HPM_PATH/ROSS
$ mkdir build install
$ cd build
$ CC=mpicc CXX=mpicxx cmake .. -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=$HPM_PATH/ROSS/install
$ make && make install
```

**Note:** With GCC on IBM Power systems, CMakeLists.txt may need to be modified to use GCC flags with ppc64le.

---

With DUMPI and ROSS installed, you are now ready to build CODES.

```
$ cd $HPM_PATH/codes
$ ./prepare.sh
$ mkdir build install
$ cd build
$ ../configure --prefix=$HPM_PATH/codes/install CC=mpicc CXX=mpicxx PKG_CONFIG_PATH=$HPM_PATH/ROSS/install/lib/pkgconfig --with-dumpi=$HPM_PATH/sst-dumpi/install
$ make && make install
```

### gpuroofperf-toolkit

To obtain GPU hardware parameters, you need to build the microbenchmark:

```
$ cd $HPM_PATH/gpuroofperf-toolkit
$ cd bench
$ mkdir build
$ cd build
$ cmake ../
$ make
```

## Building a Performance Model

**1. Obtain GPU hardware parameters**

GPU hardware parameters should be obtained on all machines/GPUs where you want
to predict the performance of your GPU kernels. This means the process of
executing the microbenchmark as below should be repeated for the different GPU
hardware.

```
$ cd $HPM_PATH/gpuroofperf-toolkit/bench/build
$ ./gpuroofperf-bench [GPU #] -o gpu_parameter.csv
```

GPU # is only required when you have multiple GPUs available, otherwise use 1.

**2. Link target application with DUMPI**

To enable MPI trace generation with DUMPI, the target application must be linked
with DUMPI. To do this, simply add `-L$HPM_PATH/sst-dumpi/install/lib -ldumpi`
as linker options.

If you want to profile and generate traces for only a certain part of the
application, include the DUMPI header file in your application as
`#include <dumpi/libdumpi/libdumpi.h>` and use `libdumpi_disable_profiling()`
and `libdumpi_enable_profiling()` to disable and enable profiling, respectively.

**3. Profile application and generate DUMPI traces**

Now that the application is linked with DUMPI, we profile it with
`gpuroofperf-toolkit` to obtain the kernel parameters and generate DUMPI traces
at the same time. Note that path to DUMPI should be added to `LD_LIBRARY_PATH`
when profiling.

`gpuroofperf-toolkit` uses NVIDIA's profiler tool (`nvprof`) under the hood
to obtain GPU performance metrics. If you want to limit the GPU profiling to a
certain part of the application, add `#include <cuda_profiler_api.h>` as well as
`cudaProfilerStart()` and `cudaProfilerStop()` calls to start and stop
profiling, respectively.

```
$ cd
$ export LD_LIBRARY_PATH=$HPM_PATH/sst-dumpi/install/lib:$LD_LIBRARY_PATH
$ python3 gpuroofperf-cli.py -o kernel_parameter.csv -x "[launcher command, e.g. jsrun]" [application]
```

Once the profiling completes, DUMPI trace files and a kernel parameter file
will be available in the folder. You can use `dumpi2ascii` for a human-readable
version of the DUMPI traces.
