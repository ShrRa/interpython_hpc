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

In the previous section, we saw how to make our code faster for sequential jobs. However, there are certain cases where no matter how much you optimize, a single process remains a bottleneck. These are tasks that are "embarrassingly parallel" - independent jobs that don't need to communicate with each other.

A perfect example from astronomy is finding the rotation period of stars from their light curves (measurements of brightness over time). Analyzing one star is quick, but analyzing data from thousands or millions of stars sequentially can take days or weeks. This is where parallel computing becomes essential.

### Real World Analogy of using sequential vs parallel with an example
Imagine you are a librarian who needs to reshelve 500 books.

Sequential Approach: You take the first book, walk to its correct shelf, place it, and walk back. You repeat this process 499 more times. Your total time is the sum of the time it takes for all 500 individual trips.

Parallel Approach: You hire a team of assistants. You give a stack of books to each person. They all go and shelve their books at the same time. The total time is now roughly the time it takes the slowest assistant to finish their stack, which is much faster than doing it all yourself.

In the case on Astronomy, analyzing light curves (time and brightness data of objects in the night sky) is like reshelving those books. We can either do it one by one (sequentially) or assign multiple light curves to different CPU cores to be analyzed simultaneously (in parallel).

![Serial vs. Parallel Performance Comparison](../fig/serial_parallel_comparision.png)

When we test the code with different numbers of light curves and calculate its most likely period using a standard astronomical algorithm called the Lomb-Scargle periodogram., we see the following:

- For smaller numbers (10, 50, 100, 200, 500, 1000, 5000),  
  the execution time is almost the same for both sequential and parallel runs which are denoted by the blue and the orange lines respectively.  
- This happens because starting and managing parallel jobs has a small extra cost (called *overhead*).  

- As the number of light curves grows larger (beyond 5000),  
  the benefit of parallelization becomes clear.  

- In the plot:
  - **Sequential time** keeps increasing steadily as the workload grows.  
  - **Parallel time** stays much flatter, showing that multiple processes share the work efficiently.  

> ###Takeaway:  
> For small tasks, parallelization does not save time (and may even cost a little extra).  
> But as the workload grows, parallelization provides a much more efficient workflow.
{: .callout}

## Parallel Programming on CPUs

Parallel programming on CPUs is primarily achieved through two widely-used models:

### OpenMP (Open Multi-Processing)

OpenMP is used for shared-memory parallelism. It enables multi-threading where each thread has access to the same memory space. It is ideal for multicore processors on a single node.

OpenMP was first introduced in October 1997 as a collaborative effort between hardware vendors, software developers, and academia. The goal was to standardize a simple, portable API for shared-memory parallel programming in C, C++, and Fortran. Over time, OpenMP has evolved to support nested parallelism, Single Instruction Multiple Data (vectorization), and offloading to GPUs, while remaining easy to integrate into existing code through compiler directives.

OpenMP is now maintained by the OpenMP Architecture Review Board, which includes organizations like Arm, AMD, IBM, Intel, Cray, HP, Fujitsu, Nvidia, NEC, Red Hat, Texas Instruments, and Oracle Corporation. OpenMP allows you to parallelize loops in C/C++ or Fortran using compiler directives.

> ## Terminology
> #### Nested Parallelism
> - Nested parallelism occurs when a parallel task itself spawns additional parallel tasks. For example, imagine a program where each thread is responsible for a different data block, and within each block, more threads are launched to handle sub-tasks. This is useful when dealing with hierarchical or recursive algorithms but must be managed carefully to avoid performance penalties due to thread overhead.
> 
> #### Single Instruction, Multiple Data (SIMD) – Vectorization
> - SIMD is a form of data-level parallelism where the same instruction operates on multiple data elements simultaneously. For instance, instead of adding two numbers at a time, SIMD allows processors to add pairs of numbers in parallel using wide registers (like 128-bit or 256-bit). Vectorized operations using NumPy or compiler intrinsics take advantage of this under the hood to speed up loops.
> 
> #### Offloading to GPUs
> - Offloading refers to transferring compute-intensive tasks from the CPU to the GPU, which is optimized for parallel processing. This is particularly effective for operations that can be executed simultaneously on thousands of threads, like matrix multiplications in deep learning or simulations in scientific computing. Tools like CUDA, OpenCL, or libraries like CuPy and PyTorch help achieve this in Python.
{: .callout}

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

Before we look at the explanation of the C code, we will first look at the Python Equivalent of this code

### Python Equivalent of the Code Logic 
 ```python 
def add_arrays(b, c):
     """
     Takes two lists `b` and `c`, adds corresponding elements, 
     and returns the resulting list `a` where a[i] = b[i] + c[i].
     """
    # Make sure both lists are the same length
    assert len(b) == len(c), "Input arrays must be the same length"

    # Create an output list of the same size
    a = [0.0 for _ in range(len(b))]

    # Loop through and compute a[i] = b[i] + c[i]
    for i in range(len(b)):
        a[i] = b[i] + c[i]

    return a

 # Example usage
 N = 100000
 b = [i * 0.1 for i in range(N)]
 c = [i * 0.2 for i in range(N)]

 a = add_arrays(b, c)

 # Print first few values to verify
 print(a[:10])
```
> ## Explanation of the C code
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
> - The output is stored in array `a`, which will contain the sum of corresponding elements from arrays `b` and `c`. The execution is faster than running the loop sequentially.
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
> This example demonstrates a basic use of `mpi4py` to perform a **gather operation** (collection of results) using the `MPI.COMM_WORLD` communicator which shows how multiple programs (called *processes*) can work together and then share their results.  
> When you run a program with MPI, you are actually running **many copies of the same program at once**. Each copy gets a number, called its **rank**.  
- If there are 4 processes, their ranks will be 0, 1, 2, and 3.  
>
> In the code each process:
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
> ~~~
> [0, 1, 4, 9]
> ~~~
{: .output}
>
> - Other ranks do not print anything.
>
> This example illustrates **point-to-root communication** which is useful when one process needs to collect and process results from all workers.
> 
> Since the terms in bold letters are introduced for the first time, here is an indepth definition which can be referred to for understanding the code 
>
> ### Terminology
>
> - **Process**:  
>  A single copy of your program that runs at the same time as the others.  
>  *Analogy: Imagine four students all solving the same type of math problem at once.*
>
> - **Rank**:  
>  The ID number for each process (starting at 0).  
>  *Analogy: Just like students in a classroom might be numbered 0, 1, 2, 3 so the teacher knows who is who.*
>
> - **Communicator (`MPI.COMM_WORLD`)**:  
>  The group of all processes that can talk to each other.  
>  *Analogy: Think of it as a big group chat that includes everyone.*
>
> - **Gather**:  
>  A way for many processes to send their results to one chosen process.  
>  *Analogy: Everyone puts their homework into the teacher’s basket, and the teacher collects them.*
>
> - **Root process**:  
>  The process that receives and collects information (by default, rank 0).  
>  *Analogy: The teacher who collects the homework and shows the class the results.*
>
> - **Point-to-root communication**:  
>  A communication pattern where many processes send information to one process.  
>  *Analogy: All students talk to the teacher, but not to each other.*
>
{: .discussion}

## Slurm Script to execute the code 

```bash
#!/bin/bash
#SBATCH --job-name=mpi_example
#SBATCH --output=mpi_%j.out
#SBATCH --error=mpi_%j.err
#SBATCH --partition=computes_thin
#SBATCH --nodes=2
#SBATCH --ntasks=4
#SBATCH --time=00:10:00
#SBATCH --mem=16G

# Load required modules
module list 
# Activate your Python environment
source /home/edu05/HPC_WS/Slurm_Scripts/interpython/bin/activate

# OPTION 1: Run using Python script (with logging)
mpirun -np 4 python  mpi_example.py
```

Make sure your virtual environment has `mpi4py` installed and that your system has access to the OpenMPI runtime via `mpirun`. Adjust the number of nodes and tasks depending on the cluster policies.

> ## Exercise 1: Gather lists instead of numbers
>
> Modify the code so that instead of collecting `rank ** 2`,  
> each process sends a **list of numbers** from `0` to `rank`.  
>
> Example (4 processes):
> - Rank 0 sends `[0]`  
> - Rank 1 sends `[0, 1]`  
> - Rank 2 sends `[0, 1, 2]`  
> - Rank 3 sends `[0, 1, 2, 3]`  
>
> The root process should gather and print:
>
> ```text
> [[0], [0, 1], [0, 1, 2], [0, 1, 2, 3]]
> ```
{: .challenge}

> ## Solution
>
> ```python
> from mpi4py import MPI
>
> comm = MPI.COMM_WORLD
> rank = comm.Get_rank()
>
> # Each process creates a list from 0 to rank
> data = list(range(rank + 1))
>
> # Gather lists at the root
> all_data = comm.gather(data, root=0)
>
> if rank == 0:
>     print(all_data)
> ```
{: .solution}

---

> ## Exercise 2: Broadcast after gather
>
> Currently, only the root process (rank 0) collects the data.  
> Modify the code so that after gathering, the root process  
> **broadcasts** the complete list back to all processes.  
>
> Hint: Use `comm.bcast()` after `comm.gather()`.  
>
> - What happens if each process prints the result after the broadcast?
{: .challenge}

> ## Solution
>
> ```python
> from mpi4py import MPI
>
> comm = MPI.COMM_WORLD
> rank = comm.Get_rank()
>
> # Each process sends rank squared
> data = rank ** 2
>
> # Gather at root
> gathered = comm.gather(data, root=0)
>
> # Broadcast the gathered list from root to all processes
> result = comm.bcast(gathered, root=0)
>
> print(f"Process {rank} received: {result}")
> ```
>
> Example output (4 processes):
>
> ```text
> Process 0 received: [0, 1, 4, 9]
> Process 1 received: [0, 1, 4, 9]
> Process 2 received: [0, 1, 4, 9]
> Process 3 received: [0, 1, 4, 9]
> ```
>
> Now **all processes** have the final list, not just the root.
{: .solution}


> ## References:
> - [OpenMP Tutorials](https://www.openmp.org/resources/tutorials-articles/)
> - [mpi4py library Documentation](https://mpi4py.readthedocs.io/en/stable/)
{: .checklist}
---

{% include links.md %}

<!-- import numpy as np
import matplotlib.pyplot as plt
from astropy.timeseries import LombScargle
import time
from joblib import Parallel, delayed

# ----- 1. Generate synthetic light curves -----
def generate_light_curve(period, duration=30, points=300, noise_level=0.2):
    time = np.linspace(0, duration, points)
    flux = np.sin(2 * np.pi * time / period) + np.random.normal(0, noise_level, points)
    return time, flux

# ----- 2. Period finding using Lomb-Scargle -----
def find_period(time, flux, min_period=0.5, max_period=10.0):
    frequency, power = LombScargle(time, flux).autopower(minimum_frequency=1/max_period,
                                                         maximum_frequency=1/min_period)
    best_period = 1 / frequency[np.argmax(power)]
    return best_period

# ----- 3. Wrapper for serial and parallel testing -----
def run_serial(light_curves):
    start = time.time()
    results = [find_period(t, f) for t, f in light_curves]
    end = time.time()
    return end - start, results

def run_parallel(light_curves, n_jobs=-1):
    start = time.time()
    results = Parallel(n_jobs=n_jobs)(delayed(find_period)(t, f) for t, f in light_curves)
    end = time.time()
    return end - start, results

# ----- 4. Benchmarking -----
def benchmark():
    num_curves_list = [10, 50, 100, 200, 300, 500]
    serial_times = []
    parallel_times = []

    for n in num_curves_list:
        # Generate multiple light curves with random periods
        periods = np.random.uniform(1.0, 5.0, size=n)
        light_curves = [generate_light_curve(p) for p in periods]

        t_serial, _ = run_serial(light_curves)
        t_parallel, _ = run_parallel(light_curves)

        serial_times.append(t_serial)
        parallel_times.append(t_parallel)
        print(f"{n} curves: Serial={t_serial:.2f}s, Parallel={t_parallel:.2f}s")

    # ----- 5. Plotting -----
    plt.figure(figsize=(10, 6))
    plt.plot(num_curves_list, serial_times, 'o-', label='Serial')
    plt.plot(num_curves_list, parallel_times, 'o-', label='Parallel')
    plt.xlabel("Number of Light Curves")
    plt.ylabel("Execution Time (seconds)")
    plt.title("Serial vs Parallel Execution Time for Period Finding")
    plt.legend()
    plt.grid(True)
    plt.tight_layout()
    plt.show()

# Run the benchmark
if __name__ == "__main__":
    benchmark() -->
