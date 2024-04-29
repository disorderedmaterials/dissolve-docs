# GPU Library Decision

- Status: **Decided**
- Deciders: Dan Nixon, Adam Washington, Tristan Youngs
- Date: 26-04-2024

## Context and Problem Statement

Many calculations in Dissolve could be greatly accelerated through use
of GPU cards.  Unfortunately, these cards are not immediately
accessibly from straight C++ and require some form of compability
library to access them.

## Decision Drivers

- SYCL implementations use standard C++ algorithms
  + No need for developers to learn another language to access GPUs
  + No need to separately debug CPU vs GPU implementations
- Most users run Dissolve on IDAaaS (which also supplies GPU cards),
  so supporting Mac and Windows is a lower priority.
- The CI build system for Linux (i.e. Nix) already has AdaptiveCPP
  available


## Considered Options

1. [OneAPI](https://www.intel.com/content/www/us/en/developer/tools/oneapi/overview.html)
2. [AdaptiveCPP](https://github.com/AdaptiveCpp/AdaptiveCpp)
3. [OpenCL](https://www.khronos.org/opencl/)
4. [CUDA](https://developer.nvidia.com/cuda-downloads)

## Decision Outcome

Chosen option: AdaptiveCPP, because it allows for a single code base
and is already available in the nixpkgs.  Additionally, since
AdaptiveCPP and OneAPI are both SYCL implementations, it should be
possible to replace AdaptiveCPP with OneAPI at a later date to add
support for other operating systems.

### Positive Consequences

- Development for GPU code can begin immediately 
- GPUs can be supported on IDAaaS, which remains the primary user platform
- GPU and CPU versions use a single code path, reducing divergence
- Calculations expected to greatly increase in speed

### Negative Consequences

- GPU support will only be available on Linux with experimental
  Windows/NVIDIA support also reported.  As most users access Dissolve
  via IDAaaS, this is not a blocker.
- AdaptiveCPP lacks a commercial sponsor (but does have academic
  support from Heidelberg University).  This brings a risk for long
  term support.  However, as it is merely an implementation of the
  SYCL standard, it would be possible to replace it with an alternatve
  implementation (e.g. OneAPI).

## Pros and Cons of the Options

### OneAPI

[OneAPI](https://www.intel.com/content/www/us/en/developer/tools/oneapi/overview.html)
is an implementation of SYCL provided by Intel.

- Good, because it is professionally maintained and well supported
- Good, unified CPU and GPU code paths
- Good, because it implements a standard with competing implementaions
- Good, because it supports a wide range of hardware, including FPGAs
- Bad, no Mac support
- Bad, requires a proprietary intel compiler
- Bad, currently unavailable in nixpkgs

### AdaptiveCPP

[AdaptiveCPP](https://github.com/AdaptiveCpp/AdaptiveCpp) is an open
source implementation of SYCL from Heidelberg University.

- Good, because it implements a standard with competing implementaions
- Good, unified CPU and GPU code paths
- Good, because it supports a wide range of hardware
- Good, uses a modification of the free [Clang](https://clang.llvm.org/) compiler
- Good, already available in nixpkgs
- Bad, only supports Linux (with Windows/CUDA support possibly available)

### OpenCL

[OpenCL](https://www.khronos.org/opencl/) is a standard interface for
connecting to heterogeneous processors.

- Good, because it implements a standard with competing implementaions
- Good, because it supports a wide range of hardware
- Bad, Mac support is officially deprecated
- Bad, requires kernels to be written in C instead of C++
- Bad, requires separate code paths for accelerated and unaccelerated code

### CUDA

[CUDA](https://developer.nvidia.com/cuda-zone) is a popular platform for accelerating code on NVIDIA cards.

- Good, abundant documentation and resources
- Good, because it is professionally maintained and well supported
- Good, kernels are written in C++
- Bad, only supports NVIDIA cards
- Bad, requires separate code paths for accelerated and unaccelerated code
