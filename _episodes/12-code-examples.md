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

Before diving into parallel computing or GPU acceleration, it's important to understand how performance can already be improved significantly on a CPU using efficient libraries. One of the most widely used tools for this in Python is NumPy.

NumPy provides a fast and memory-efficient way to handle large numerical datasets using multi-dimensional arrays and vectorized operations. While regular Python lists are flexible, they are not optimized for heavy numerical tasks. Looping through data element by element can quickly become a bottleneck as the problem size grows.

NumPy solves this problem by providing a powerful N-dimensional array object and tools for performing operations on these arrays efficiently. Under the hood, NumPy uses optimized C code, so operations are much faster than using standard Python loops.

NumPy also supports vectorized operations, which means you can apply functions to entire arrays without writing explicit loops. This not only improves performance but also leads to cleaner and more readable code.

Using NumPy on the CPU is often the first step toward writing efficient scientific code. It's a strong foundation before we move on to parallel computing or GPU acceleration.

Now, we'll see an example of how a simple numerical operation is implemented using NumPy on a single CPU core.

We begin with a simple example of serial computation: summing the elements of a large array.

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

> ## Exercise: Modify the above to use a manual loop with `for` instead of `np.sum`, and compare the performance.
{: .challenge}

> **Reference**: [Carpentries Python loops lesson](https://swcarpentry.github.io/python-novice-inflammation/05-loop.html)

---

## Parallel CPU Programming

### Introduction to OpenMP and MPI

Parallel programming on CPUs is primarily achieved through two widely-used models:

- **OpenMP (Open Multi-Processing)**: Used for shared-memory parallelism. It enables multi-threading where each thread has access to the same memory space. Ideal for multicore processors on a single node.
- **MPI (Message Passing Interface)**: Used for distributed-memory parallelism. Processes run on separate memory spaces (often on different nodes) and communicate via message passing. Suitable for large-scale HPC clusters.


### OpenMP: Shared Memory Model

OpenMP was first introduced in 1997 as a collaborative effort between hardware vendors, software developers, and academia. The goal was to standardize a simple, portable API for shared-memory parallel programming in C, C++, and Fortran. Over time, OpenMP has evolved to support nested parallelism, SIMD (vectorization), and offloading to GPUs, while remaining easy to integrate into existing code through compiler directives (e.g., #pragma omp parallel).

OpenMP is now maintained by the OpenMP Architecture Review Board, which includes organizations like Intel, AMD, and NVIDIA.     OpenMP allows you to parallelize loops in C/C++ or Fortran using compiler directives.

```c
#include <omp.h>
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    a[i] = b[i] + c[i];
}
```

Requires adding `#include <omp.h>` and compiling with `-fopenmp`.

### MPI: Distributed Memory Model

MPI emerged earlier, in the early 1990s, as the need for a standardized message-passing interface became clear in the growing field of distributed-memory computing. Before MPI, various parallel systems used their own vendor-specific libraries, making code difficult to port across machines.

In 1994, the first official MPI standard (MPI-1) was published by the MPI Forum, a collective of academic institutions, government labs, and industry partners. Since then, MPI has become the de facto standard for scalable parallel computing across multiple nodes, and it continues to evolve with versions like MPI-2 and MPI-3, which add support for features like parallel I/O and dynamic process management. MPI is suited for parallelism across nodes (e.g., cluster computing).

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

> ## Exercise: Modify serial array summation using OpenMP (C) or `multiprocessing` (Python).
{: .challenge}

> **References**:
- [OpenMP Examples](https://www.openmp.org/resources/examples/)
- [mpi4py Documentation](https://mpi4py.readthedocs.io/en/stable/)

---

## GPU Programming Concepts

GPUs, or Graphics Processing Units, are composed of thousands of lightweight processing cores that are optimized for handling multiple operations simultaneously. This parallel architecture makes them particularly effective for data-parallel problems, where the same operation is performed independently across large datasets—such as matrix multiplications, vector operations, or image processing tasks.

Originally designed to accelerate the rendering of complex graphics and visual effects in computer games, GPUs are inherently well-suited for high-throughput computations involving large tensors and multidimensional arrays. Their architecture enables them to perform numerous arithmetic operations in parallel, which has made them increasingly valuable in scientific computing, deep learning, and simulations.

Even without explicit parallel programming, many modern libraries and frameworks (such as TensorFlow, PyTorch, and CuPy) can automatically leverage GPU acceleration to significantly improve performance. However, to fully exploit the computational power of GPUs, especially in high-performance computing (HPC) environments, explicit parallelization is often employed.

## Introduction to CUDA

In HPC systems, **CUDA** (Compute Unified Device Architecture), a parallel computing platform and programming model developed by **NVIDIA** is the most widely used platform for GPU programming. CUDA allows developers to write highly parallel code that runs directly on the GPU, providing fine-grained control over memory usage, thread management, and performance optimization. It allows developers to harness the power of **NVIDIA GPUs** for general-purpose computing, known as GPGPU (General-Purpose computing on Graphics Processing Units).

### A Brief History

- **Introduced by NVIDIA in 2006**, CUDA was the first platform to provide direct access to the GPU's virtual instruction set and parallel computational elements.
- Before CUDA, GPUs were primarily used for rendering graphics, and general-purpose computations required indirect use through graphics APIs like OpenGL or DirectX.
- CUDA revolutionized scientific computing, deep learning, and high-performance computing (HPC) by enabling massive parallelism and accelerating workloads previously limited to CPUs.

### How CUDA Works

CUDA allows developers to write **C, C++, Fortran, and Python** code that runs on the GPU.

- A CUDA program typically runs **on both the CPU (host)** and **the GPU (device)**.
- Computational tasks (kernels) are written to execute in parallel across thousands of lightweight **CUDA threads**.
- These threads are organized hierarchically into:
  - **Grids** of **Blocks**
  - **Blocks** of **Threads**

This hierarchical design allows fine-grained control over memory and computation.

### Key Features

- **Massive parallelism** with thousands of concurrent threads
- **Unified memory architecture** for seamless CPU-GPU data access
- **Built-in libraries** for BLAS, FFT, random number generation, and more (e.g., cuBLAS, cuFFT, cuRAND)
- **Tooling support** including profilers, debuggers, and performance analyzers (e.g., Nsight, CUDA-GDB)

### A CUDA program includes:

- **Host code**: Runs on the CPU, manages memory, and launches kernels.
- **Device code (kernel)**: Runs on the GPU.
- **Memory management**: Host/device memory allocations and transfers.

### Checking CUDA availability before running code

```python
import cuda

if cuda.is_available():
    print("CUDA is available!")
    print(f"Detected GPU: {cuda.get_current_device().name}")
else:
    print("CUDA is NOT available.")
```

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

## Simple CUDA GPU Code Example

Here’s a basic CUDA example for vector addition:

```cuda
__global__ void add(int *a, int *b, int *c, int N) {
    int index = threadIdx.x + blockIdx.x * blockDim.x;
    if (index < N)
        c[index] = a[index] + b[index];
}
```

> ## Exercise: Show which parts of the code execute on GPU vs CPU (host vs device). Introduce concepts like memory copy and kernel launch.
{: .challenge}

> **Reference**: [NVIDIA CUDA Samples](https://github.com/NVIDIA/cuda-samples)

---

### CPU vs GPU Architecture

- CPUs: Few powerful cores, better for sequential tasks.
- GPUs: Many lightweight cores, ideal for parallel workloads.

> ## Figure Suggestion: Diagram comparing CPU vs GPU architecture, e.g., from [CUDA C Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html)
{: .callout}

## Comparing CPU and GPU Approaches

| Feature      | CPU (OpenMP/MPI)          | GPU (CUDA)                  |
|--------------|---------------------------|-----------------------------|
| Cores        | Few (2–64)                | Thousands (1024–10000+)     |
| Memory       | Shared / distributed      | Device-local (needs transfer)|
| Programming  | Easier to debug           | Requires more setup         |
| Performance  | Good for logic-heavy tasks| Excellent for large, data-parallel problems |

> **Figure**: Bar chart showing performance on matrix multiplication or vector addition.

---

## Code Profiling (Optional)

To understand and improve performance, profiling tools are essential.

- **CPU**: `gprof`, `perf`, `cProfile`
- **GPU**: `nvprof`, Nsight Systems, Nsight Compute

> ## Exercise: Time your serial and parallel code. Where is the bottleneck?
{: .challenge}

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