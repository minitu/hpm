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

### CODES

DUMPI, Cortex, and ROSS should be installed before building CODES.

#### DUMPI

The MPI-Replay module in CODES reads DUMPI traces to simulate MPI communication.
To build DUMPI:

```
$ cd sst-dumpi
$ ./bootstrap.sh
$ mkdir build install
$ cd build
$ ../configure --enable-libdumpi --enable-libundumpi CC=mpicc CXX=mpicxx --prefix=[PATH_TO_HPM]/sst-dumpi/install
$ make && make install
```

#### Cortex

Cortex is a library that translates collective MPI communication calls in DUMPI
traces to their equivalent point-to-point calls, in particular as implemented
in MPICH.
Cortex can be built with the following commands (without optional Python support):

```
$
```

#### ROSS

### gpuroofperf-toolkit

