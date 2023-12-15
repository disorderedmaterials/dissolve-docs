# Print Formatting

- Status: **Proposed**
- Deciders: Adam Washington, Dan Nixon, Tristan Youngs
- Date: 16-11-2020

## Context and Problem Statement

The need for a comprehensive parallelism solution throughout the code
to enable large simulations to be completed in a reasonable time while
running on diverse architectures.


## Decision Drivers

- Current MPI solution does not easily support multiple platforms
- Need to support multiple hardware architectures (e.g. multi-core, GPU)


## Considered Options

1. [MPI](https://www.mpi-forum.org)
2. [Parallel STL](https://en.cppreference.com/w/cpp/header/execution)
3. [OpenMP](https://openmp.org)
4. [CUDA](https://developer.nvidia.com/cuda-zone)
5. [OpenCL](https://www.khronos.org/opencl/)
6. [CyCL](https://www.khronos.org/sycl/)

## Decision Outcome

We ultimately have a 4 stage path forward

### 1) Modernisation

We continue moving the code base toward standard C++ containers.  This
provides more information to the compiler and should provide
performance benefits under all circumstances.  Even if the roadmap for
parallelism changes, it is likely that this will still provide benefits
and highly unlikely that it will be a hindrance.

### 2) STL Parallelism

Using standard containers will enable us to use standard algorithms,
many of which now support parallelism as of C++17.  This will involve
converting many of our loops into algorithms and lambdas.  Beyond
immediately providing a speed-up on multi-core machines, this process
moves the code base into a position that will be easier to parallelize
through one of the GPU libraries

### 3) CyCL

Once the parallel STL components are in place, we can then start
moving some of these pieces into CyCL.  This will involve more
marshalling of arrays than straight C++ STL, but will enable GPU
access that should hopefully provide speed ups of orders of magnitude.

### 4) MPI removal

If the CyCL transition is successful and is able to perform faster on
a single GPU enabled node than the MPI version is able to perform
across multiple nodes in a cluster, then it would be possible to
remove the MPI code from dissolve.  This would simplify multiple
sections of the code and the build systems, as well as eliminating
much of our use of the pre-processor.



### Positive Consequences

- No point in this process should involve a performance regression for the users.
- This should enable code speed-ups on everything from a laptop to a cluster.
- Should new technologies emerge, using algorithms will enable shifting the code faster than the current for-loops
- Intel's support of CyCL ensures that the library should continue to be available and supported for the foreseeable future.


### Negative Consequences

- It will not be possible to benchmark the CyCL code until late in the process.  Should the CyCL code outperform expectation and the STL underperform, it's possible that excessive time may be spent on stage 2 instead of skipping straight to stage 3.
- CyCL is a relatively new library, so the avaiable knowledge and best
  practices aren't as well defined.


## Pros and Cons of the Options

### MPI

- Good, already written
- Good, mature technology
- Good, supported by computing clusters at many institutions
- Bad, poor workstation support
- Bad, memory usage scales with processor count
- Bad, communication overhead between nodes
- Bad, no support for advanced hardware

### Parallel STL

- Good, supported on all platforms with a C++17 compiler
- Good, will likely be supported as long as C++ exists
- Good, support for multiple cores and vector instructions
- Good, support for standard C++ containers and algorithms
- Bad, requires algorithms to be specifically labelled (no just for loops)
- Bad, no support for advanced hardware (GPU code)

### OpenMP

- Good, supported by most common C++ compilers
- Good, support for multiple cores and vector instructions
- Good, requires simple annotations of existing for-loops
- Bad, weak support for advanced hardware (GPU code)
- Bad, no support for C++ algorithms

### CUDA

- Good, support for NVIDIA GPUs
- Good, significant tooling support available
- Good, plentiful libraries
- Good, single C++ code base
- Bad, no support for non-NVIDIA GPUs

### OpenCL

- Good, support for GPUs from multiple vendors
- Good, support for multi-core systems
- Bad, notoriously heavy boiler plate
- Bad, splits code base into two languages
- Bad, poor vendor support

### CyCL

- Good, support for GPUs from multiple vendors
- Good, support for multi-core systems
- Good, single C++ code base
- Good, tooling support available from Intel
- Good, multiple implementations
- Bad, newer technology with few best practices established


