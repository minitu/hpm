# HPM: Heterogeneous Performance Modeling

HPM is a performance modeling framework targeted at applications that utilize
GPUs to accelerate computation and MPI for distributed memory communication.
Building on existing projects, it aims to achieve both high prediction accuracy
and ease of use for creating performance models for any distributed GPU
application; the user should not have to inspect the code or be knowledgeable
about the internals of the target application.

It builds on [CODES](https://github.com/minitu/codes) to simulate MPI
communication traces and
[gpuroofperf-toolkit](https://github.com/minitu/gpuroofperf-toolkit) to predict
GPU kernel performance. CODES requires
[DUMPI](https://github.com/minitu/sst-dumpi) to generate and read MPI traces,
[Cortex](https://github.com/minitu/dumpi-cortex) to translate collective MPI
calls to point-to-point calls in DUMPI traces, and
[ROSS](https://github.com/ROSS-org/ROSS) as the parallel discrete event
simulation engine.

## Install

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

IBM XL C/C++ 16.1.1 was used to compile DUMPI and Cortex, GCC 4.9.3 for ROSS
and CODES (CODES fails at link time with IBM XL compilers), and CMake 3.14.5.

**Before proceeding, set the environment variable** `HPM_PATH` **to point to
the directory where the HPM repository is located.**

### 1. CODES

DUMPI, Cortex, and ROSS should be installed before building CODES.

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

#### 1-2. Cortex

Cortex is a library that translates collective MPI communication calls in DUMPI
traces to their equivalent point-to-point calls, in particular as implemented
in MPICH.
Cortex can be built with the following commands (without optional Python support):

```
$ cd $HPM_PATH/cortex
$ mkdir build install
$ cd build
$ CC=mpicc CXX=mpicxx cmake .. -G "Unix Makefiles" -DMPICH_FORWARD:BOOL=TRUE -DCMAKE_INSTALL_PREFIX=$HPM_PATH/cortex/install -DDUMPI_ROOT=$HPM_PATH/sst-dumpi/install
$ make && make install
```

#### 1-3. ROSS

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

With DUMPI, Cortex, and ROSS installed, you are now ready to build CODES.

```
$ cd $HPM_PATH/codes
$ ./prepare.sh
$ mkbid build install
$ cd build
$ ../configure --prefix=$HPM_PATH/codes/install CC=mpicc CXX=mpicxx PKG_CONFIG_PATH=$HPM_PATH/ROSS/install/lib/pkgconfig --with-dumpi=$HPM_PATH/sst-dumpi/install --with-cortex=$HPM_PATH/cortex/install
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
```

## Build Performance Model
