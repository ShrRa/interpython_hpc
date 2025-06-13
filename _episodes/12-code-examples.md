---
title: "Intro code examples"
start: False
teaching: 30
exercises: 20
questions:
- "What is the difference between serial and parallel code?"
- "How do CPU and GPU programs differ?"
- "What tools and programming models are used for HPC development?"
objectives:
- "Understand the structure of CPU and GPU code examples."
- "Identify differences between serial, multi-threaded, and GPU-accelerated code."
- "Recognize common programming models like OpenMP, MPI, and CUDA."
- "Appreciate performance trade-offs and profiling basics."
keypoints:
- "Serial code is limited to a single thread of execution, while parallel code uses multiple cores or nodes."
- "OpenMP and MPI are popular for parallel CPU programming; CUDA is used for GPU programming."
- "High-level libraries like Numba and CuPy make GPU acceleration accessible from Python."
---

## Motivation for HPC Coding

Most users begin with simple serial code, which runs sequentially on one processor. However, for problems involving large data sets, high resolution simulations, or time-critical tasks, serial execution quickly becomes inefficient.

Parallel programming allows us to split work across multiple CPUs or even GPUs. High-Performance Computing (HPC) relies on this concept to solve problems faster.

> **Figure Suggestion**: Plot showing execution time of serial vs parallel implementation for increasing problem sizes (e.g., matrix size or loop iterations).

---

## Serial Code Example (CPU)

Let’s begin with a simple example of serial computation: summing the elements of a large array.

```python
import numpy as np
import time

array = np.random.rand(10**7)
start = time.time()
total = np.sum(array)
end = time.time()
print(f"Sum: {total}, Time taken: {end - start:.4f} seconds")
```

- This code runs on a single core.
- Useful to measure baseline performance.

> **Exercise**: Modify the above to use a manual loop with `for` instead of `np.sum`, and compare the performance.

> **Reference**: [Carpentries Python loops lesson](https://swcarpentry.github.io/python-novice-inflammation/05-loop/)

---

## Parallel CPU Programming

### OpenMP: Shared Memory Model

OpenMP allows you to parallelize loops in C/C++ or Fortran using compiler directives.

```c
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    a[i] = b[i] + c[i];
}
```

Requires adding `#include <omp.h>` and compiling with `-fopenmp`.

### MPI: Distributed Memory Model

MPI is suited for parallelism across nodes (e.g., cluster computing).

```python
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

data = rank ** 2
all_data = comm.gather(data, root=0)
if rank == 0:
    print(all_data)
```

- MPI enables message passing across multiple processes.
- Common in supercomputers and clusters.

> **Exercise**: Modify serial array summation using OpenMP (C) or `multiprocessing` (Python).

> **References**:
- [OpenMP Examples](https://www.openmp.org/resources/examples/)
- [mpi4py Documentation](https://mpi4py.readthedocs.io/en/stable/)

---

## GPU Programming Concepts

GPUs have thousands of small cores and are highly effective for data-parallel problems.

### CPU vs GPU Architecture

- CPUs: Few powerful cores, better for sequential tasks.
- GPUs: Many lightweight cores, ideal for parallel workloads.

> **Figure Suggestion**: Diagram comparing CPU vs GPU architecture, e.g., from [CUDA C Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html)

---

## Simple CUDA GPU Code Example

Here’s a basic CUDA example for vector addition:

```cuda
__global__ void add(int *a, int *b, int *c, int N) {
    int index = threadIdx.x + blockIdx.x * blockDim.x;
    if (index < N)
        c[index] = a[index] + b[index];
}
```

> **Exercise**: Show which parts of the code execute on GPU vs CPU (host vs device). Introduce concepts like memory copy and kernel launch.

> **Reference**: [NVIDIA CUDA Samples](https://github.com/NVIDIA/cuda-samples)

---

## Comparing CPU and GPU Approaches

| Feature      | CPU (OpenMP/MPI)          | GPU (CUDA)                  |
|--------------|---------------------------|-----------------------------|
| Cores        | Few (2–64)                | Thousands (1024–10000+)     |
| Memory       | Shared / distributed      | Device-local (needs transfer)|
| Programming  | Easier to debug           | Requires more setup         |
| Performance  | Good for logic-heavy tasks| Excellent for large, data-parallel problems |

> **Figure**: Bar chart showing performance on matrix multiplication or vector addition.

---

## High-Level Libraries for Portability

High-level libraries allow easier GPU programming in Python:

- **Numba**: JIT compiler for Python; supports GPU via `@cuda.jit`
- **CuPy**: NumPy-like API for NVIDIA GPUs
- **Dask**: Parallel computing with familiar APIs

Example using Numba:

```python
from numba import cuda
import numpy as np

@cuda.jit
def add_vectors(a, b, c):
    i = cuda.grid(1)
    if i < a.size:
        c[i] = a[i] + b[i]
```

> **Exercise**: Write a Numba or CuPy version of vector addition and compare speed with NumPy.

> **References**:
- [Numba CUDA Docs](https://numba.readthedocs.io/en/stable/cuda/)
- [CuPy Documentation](https://docs.cupy.dev/)

---

## Code Profiling (Optional)

To understand and improve performance, profiling tools are essential.

- **CPU**: `gprof`, `perf`, `cProfile`
- **GPU**: `nvprof`, Nsight Systems, Nsight Compute

> **Exercise**: Time your serial and parallel code. Where is the bottleneck?

> **Optional Reference**: [NVIDIA Nsight Tools](https://developer.nvidia.com/nsight-systems)

---

## Summary

- Serial code is simple but doesn’t scale well.
- Use OpenMP and MPI for parallelism on CPUs.
- Use CUDA (or high-level wrappers like Numba/CuPy) for GPU programming.
- Always profile your code to understand performance.
- Choose your tool based on problem size, complexity, and hardware.

---

## Next Episode

➡️ **Command Line for HPC and Other Remote Facilities**




{% include links.md %}