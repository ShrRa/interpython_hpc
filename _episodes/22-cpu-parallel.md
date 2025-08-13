---
title: "Parallelising our code for CPU"
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

## Parallel CPU Programming

### Introduction to OpenMP and MPI

#### Parallel programming on CPUs is primarily achieved through two widely-used models:

### OpenMP (Open Multi-Processing)

OpenMP is used for shared-memory parallelism. It enables multi-threading where each thread has access to the same memory space. It is ideal for multicore processors on a single node.

OpenMP was first introduced in October 1997 as a collaborative effort between hardware vendors, software developers, and academia. The goal was to standardize a simple, portable API for shared-memory parallel programming in C, C++, and Fortran. Over time, OpenMP has evolved to support nested parallelism, Single Instruction Multiple Data (vectorization), and offloading to GPUs, while remaining easy to integrate into existing code through compiler directives.

OpenMP is now maintained by the OpenMP Architecture Review Board, which includes organizations like Arm, AMD, IBM, Intel, Cray, HP, Fujitsu, Nvidia, NEC, Red Hat, Texas Instruments, and Oracle Corporation. OpenMP allows you to parallelize loops in C/C++ or Fortran using compiler directives.

### Example: Running a loop in parallel using OpenMP    
```c
#include <omp.h>
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    a[i] = b[i] + c[i];
}
```

Since C programming is not a prerequisite for this workshop, let's break down the parallel loop code in detail.

**Requirements**:  
- Add `#include <omp.h>` to your code
- Compile with `-fopenmp` flag

> ### Explanation of the code
>
> - `#include <omp.h>`: Includes the OpenMP API header needed for all OpenMP functions and directives.
> - `#pragma omp parallel for`: A **compiler directive** that tells the compiler to **parallelize the `for` loop** that follows.
> - The `for` loop itself performs **element-wise addition** of two arrays (`b` and `c`), storing the result in array `a`.
>
> ### How OpenMP Executes This
>
> 1. OpenMP detects available CPU cores (e.g., 4 or 8).
> 2. It splits the loop into chunks — one for each thread.
> 3. Each core runs its chunk **simultaneously** (in parallel).
> 4. The threads **synchronize automatically** once all work is done.
>
> ### Output
>
> The output is stored in array `a`, which will contain the sum of corresponding elements from arrays `b` and `c`. The execution is faster than running the loop sequentially.
>
> ### Real-World Analogy
>
> Suppose you need to send 100 emails:
>
> - **Without OpenMP**: One person sends all 100 emails one by one.
> - **With OpenMP**: 4 people each send 25 emails **at the same time** — finishing in a quarter of the time.
>
{: .discussion}


---
> ## Exercise: Parallelization Challenge
>
> Consider this loop:
> 
> ~~~c
> for (int i = 1; i < N; i++) {
>   a[i] = a[i-1] + b[i];
> }
> ~~~
> Can this be parallelized with OpenMP? Why or why not?
{: .challenge}

> ## Solution
>
> No, this cannot be safely parallelized because each iteration depends on the result of the previous iteration (`a[i-1]`). 
> 
> OpenMP requires loop iterations to be independent for parallel execution. Here, since each `a[i]` relies on `a[i-1]`, the loop has a **sequential dependency**, also known as a **loop-carried dependency**. 
> 
> This prevents naive parallelization with OpenMP's `#pragma omp parallel for`.
>
> However, this type of problem can be parallelized using more advanced techniques like a **parallel prefix sum (scan)** algorithm, which restructures the computation to allow parallel execution in logarithmic steps instead of linear.
{: .solution}


### MPI (Message Passing Interface)

MPI is used for distributed-memory parallelism. Processes run on separate memory spaces (often on different nodes) and communicate via message passing. It is suitable for large-scale HPC clusters.

MPI emerged earlier, in the early 1990s, as the need for a standardized message-passing interface became clear in the growing field of distributed-memory computing. Before MPI, various parallel systems used their own vendor-specific libraries, making code difficult to port across machines.

In June 1994, the first official MPI standard (MPI-1) was published by the MPI Forum, a collective of academic institutions, government labs, and industry partners. Since then, MPI has become the de facto standard for scalable parallel computing across multiple nodes, and it continues to evolve with versions like MPI-2, MPI-3, MPI-4, and finally MPI-5 released on June 5 2025 which add support for features like parallel I/O and dynamic process management. 

### Example: Implementation of MPI using the mpi4py library in python

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

> ## Explanation of the code
>
> This example demonstrates a basic use of `mpi4py` to perform a **gather operation** using the `MPI.COMM_WORLD` communicator.
>
> Each process:
>
> - Determines its **rank** (an integer from 0 to N-1, where N is the number of processes).
> - Computes `rank ** 2` (the square of its rank).
> - Uses `comm.gather()` to send the result to the **root process** (rank 0).
>
> Only the **root process** gathers the data and prints the complete list.
>
> ### Example Output (4 processes):
>
> - Rank 0 computes `0² = 0`
> - Rank 1 computes `1² = 1`
> - Rank 2 computes `2² = 4`
> - Rank 3 computes `3² = 9`
>
> The root process (rank 0) gathers all results and prints:
>
> ```text
> [0, 1, 4, 9]
> ```
>
> Other ranks do not print anything.
>
> This example illustrates **point-to-root communication** — useful when one process needs to collect and process results from all workers.
{: .discussion}

> ## Note:
> You won't be able to run this code in your current environment. This example requires a Slurm job submission script to launch MPI processes across nodes. Detailed instructions on how to configure Slurm scripts and request resources are provided in [Section 2: HPC Bura - Resource Optimization ](https://meet-vyas-dev.github.io/interpython_hpc/24-resource-optimization/index.html).
{: .prereq}

Typically one would run this file after having a slurm script with the required resources followed by this command

```bash
mpirun -n 4 python your_script.py
```

> ## Exercise: 
> Modify serial array summation using OpenMP (C) or `multiprocessing` (Python).
{: .challenge}

> ## References:
> - [OpenMP Tutorials](https://www.openmp.org/resources/tutorials-articles/)
> - [mpi4py library Documentation](https://mpi4py.readthedocs.io/en/stable/)
{: .checklist}
---

{% include links.md %}
