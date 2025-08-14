---
title: "Implementing code examples for running on GPU"
start: False
teaching: 30
exercises: 20
questions:
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

## GPU Programming Concepts

GPUs, or Graphics Processing Units, are composed of thousands of lightweight processing cores that are optimized for handling multiple operations simultaneously. This parallel architecture makes them particularly effective for data-parallel problems, where the same operation is performed independently across large datasets such as matrix multiplications, vector operations, or image processing tasks.

Originally designed to accelerate the rendering of complex graphics and visual effects in computer games, GPUs are inherently well-suited for high-throughput computations involving large tensors and multidimensional arrays. Their architecture enables them to perform numerous arithmetic operations in parallel, which has made them increasingly valuable in scientific computing, deep learning, and simulations.

Even without explicit parallel programming, many modern libraries and frameworks (such as TensorFlow, PyTorch, and CuPy) can automatically leverage GPU acceleration to significantly improve performance. However, to fully exploit the computational power of GPUs, especially in high-performance computing (HPC) environments, explicit parallelization is often employed.

## Introduction to CUDA

In HPC systems, CUDA (Compute Unified Device Architecture), a parallel computing platform and programming model developed by NVIDIA is the most widely used platform for GPU programming. CUDA allows developers to write highly parallel code that runs directly on the GPU, providing fine-grained control over memory usage, thread management, and performance optimization. It allows developers to harness the power of NVIDIA GPUs for general-purpose computing, known as GPGPU (General-Purpose computing on Graphics Processing Units).

### A Brief History

- Introduced by NVIDIA in 2006, CUDA was the first platform to provide direct access to the GPU's virtual instruction set and parallel computational elements.
- Before CUDA, GPUs were primarily used for rendering graphics, and general-purpose computations required indirect use through graphics APIs like OpenGL or DirectX.
- CUDA revolutionized scientific computing, deep learning, and high-performance computing (HPC) by enabling massive parallelism and accelerating workloads previously limited to CPUs.

### How CUDA Works

CUDA allows developers to write C, C++, Fortran, and Python code that runs on the GPU.

- A CUDA program typically runs on both the CPU (host) and the GPU (device).
- Computational tasks (kernels) are written to execute in parallel across thousands of lightweight CUDA threads.
- These threads are organized hierarchically into:
  - Grids of Blocks
  - Blocks of Threads
- This can be visualised in the following form

![CUDA heirarchy visulation lower level](../fig/cuda_blocks.png)
![CUDA Kernel Execution on GPU](../fig/cuda_kernel_execution.png)

> ## Figure Source:
> - [CUDA Kernel Execution](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/)
{: .checklist}

### Key Features

- **Massive parallelism** with thousands of concurrent threads
- **Unified memory architecture** for seamless CPU-GPU data access
- **Built-in libraries** for BLAS, FFT, random number generation, and more (e.g., cuBLAS, cuFFT, cuRAND)
- **Tooling support** including profilers, debuggers, and performance analyzers (e.g., Nsight, CUDA-GDB)

### A CUDA program includes:

- **Host code**: Runs on the CPU, manages memory, and launches kernels.
- **Device code (kernel)**: Runs on the GPU.
- **Memory management**: Host/device memory allocations and transfers.

### To execute any CUDA program, there are three main steps:

- Copy the input data from host memory to device memory, also known as host-to-device transfer.
- Load the GPU program and execute, caching data on-chip for performance.
- Copy the results from device memory to host memory, also called device-to-host transfer.


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

### Example: Add vectors utlising CUDA using the numba python library 

```python
from numba_cuda import cuda
import numpy as np
import time

@cuda.jit
def add_vectors(a, b, c):
    i = cuda.grid(1)
    if i < a.size:
        c[i] = a[i] + b[i]

# Setup input arrays
N = 1_000_000
a = np.arange(N, dtype=np.float32)
b = np.arange(N, dtype=np.float32)
c = np.zeros_like(a)

# Copy arrays to device
d_a = cuda.to_device(a)
d_b = cuda.to_device(b)
d_c = cuda.device_array_like(a)

# Configure the kernel
threads_per_block = 256
blocks_per_grid = (N + threads_per_block - 1) // threads_per_block

# Launch the kernel
start = time.time()
add_vectors[blocks_per_grid, threads_per_block](d_a, d_b, d_c)
cuda.synchronize()  # Wait for GPU to finish
gpu_time = time.time() - start

# Copy result back to host
d_c.copy_to_host(out=c)

# Verify results
print("First 5 results:", c[:5])
print("Time taken on GPU:", gpu_time, "seconds")
```

## Slurm Script to execute the code 

The following script can be used to submit a GPU-accelerated Python job (`numba_cuda_test.py`) using Slurm:

```bash
#!/bin/bash
#SBATCH --job-name=Numba_Cuda
#SBATCH --output=Numba_Cuda_%j.out
#SBATCH --error=Numba_Cuda_%j.err
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --gpus-per-node=1
#SBATCH --time=00:10:00

# --------- Load Environment ---------
module load Python/3.9.1
module load cuda/11.2
module list

# --------- Check whether the GPU is available ---------
from numba import cuda
print("CUDA Available:", cuda.is_available())
# Activate virtual environment
source 'name_of_venv'/bin/activate # Here name_of_venv refers to the name of your virtual environment without the quotes

# --------- Run the Python Script ---------
 python numba_cuda_test.py
```
Make sure your virtual environment includes the `numba-cuda` python library to access the GPU. 

> ## Exercise: 
> Write a Numba or CuPy version of vector addition and compare speed with NumPy.
{: .challenge}

> ## References:
> - [Numba-CUDA Docs](https://nvidia.github.io/numba-cuda/)
> - [CuPy Documentation](https://docs.cupy.dev/)
{: .checklist}
---

<!-- ## Simple CUDA GPU Code Example

Here’s a basic CUDA example for vector addition:

```cuda
__global__ void add(int *a, int *b, int *c, int N) {
    int index = threadIdx.x + blockIdx.x * blockDim.x;
    if (index < N)
        c[index] = a[index] + b[index];
}
``` -->

---

### CPU vs GPU Architecture

- CPUs: Few powerful cores, better for sequential tasks.
- GPUs: Many lightweight cores, ideal for parallel workloads.

> ## Figure Suggestion: 
> Diagram comparing CPU vs GPU architecture, e.g., from [CUDA C Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html)
{: .callout}

## Comparing CPU and GPU Approaches

| Feature      | CPU (OpenMP/MPI)          | GPU (CUDA)                  |
|--------------|---------------------------|-----------------------------|
| Cores        | Few (2–64)                | Thousands (1024–10000+)     |
| Memory       | Shared / distributed      | Device-local (needs transfer)|
| Programming  | Easier to debug           | Requires more setup         |
| Performance  | Good for logic-heavy tasks| Excellent for large, data-parallel problems |

> ## Exercise: 
> Show which parts of the code execute on GPU vs CPU (host vs device). Read about concepts like memory copy and kernel launch.
{: .challenge}

> **Reference**: [NVIDIA CUDA Samples](https://github.com/NVIDIA/cuda-samples)
{: .checklist}

> ## Figure:
> Bar chart showing performance on matrix multiplication or vector addition.
{: .callout}

---

## Code Profiling (Optional)

To understand and improve performance, profiling tools are essential.

- **CPU**: `gprof`, `perf`, `cProfile`
- **GPU**: `nvprof`, Nsight Systems, Nsight Compute

> ## Exercise: 
> Time your serial and parallel code. Where is the bottleneck?
{: .challenge}

> **Optional Reference**: [NVIDIA Nsight Tools](https://developer.nvidia.com/nsight-systems)
{: .checklist}

---

## Summary

- Serial code is simple but doesn’t scale well.
- Use OpenMP and MPI for parallelism on CPUs.
- Use CUDA (or high-level wrappers like Numba/CuPy) for GPU programming.
- Always profile your code to understand performance.
- Choose your tool based on problem size, complexity, and hardware.

---

{% include links.md %}
