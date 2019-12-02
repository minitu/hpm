# HPM: Heterogeneous Performance Modeling

HPM is a performance modeling framework targeted at applications that utilize
GPUs to accelerate computation and MPI for distributed memory communication.
It aims to achieve both high prediction accuracy and ease of use for creating
performance models for distributed GPU applications; the user is spared from
having to inspect the code or know the internals of the code of the target
application.

It builds on [CODES](https://github.com/minitu/codes) to simulate MPI
communication traces and
[gpuroofperf-toolkit](https://github.com/minitu/gpuroofperf-toolkit) to predict
GPU kernel performance. It also uses
[DUMPI](https://github.com/minitu/sst-dumpi) to generate MPI traces.

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

### gpuroofperf-toolkit

### DUMPI

```
$ cd sst-dumpi
$ ./bootstrap.sh
$ mkdir build install
$ cd build
$ ../configure --enable-libdumpi --enable-libundumpi CC=mpicc CXX=mpicxx --prefix=[PATH_TO_HPM]/sst-dumpi/install
$ make && make install
```
