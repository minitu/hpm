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
