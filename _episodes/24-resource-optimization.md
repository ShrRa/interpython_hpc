---
title: "Resource optimization"
teaching: 5
exercises: 0
questions:
- "Question 1"
objectives:
- Understand different types of computational workloads and their resource requirements
- Write optimized Slurm job scripts for sequential, parallel, and GPU workloads
- Monitor and analyze resource utilization
- Apply best practices for efficient resource allocation
keypoints:
- "Keypoint 1"
---

- Resource optimization (+practical session: running CPU and GPU code examples)

# Resource Optimization

## Learning Objectives

By the end of this lesson, you will be able to:


## Understanding Resource Requirements

Different computational tasks have varying resource requirements. Understanding these patterns is crucial for efficient HPC usage.

### Types of Workloads

**CPU-bound workloads**: Tasks that primarily use computational power
- Mathematical calculations, simulations, data processing
- Benefit from more CPU cores and higher clock speeds

**Memory-bound workloads**: Tasks limited by memory access speed
- Large dataset processing, in-memory databases
- Require sufficient RAM and fast memory access

**I/O-bound workloads**: Tasks limited by disk or network operations
- File processing, database queries, data transfer
- Benefit from fast storage and network connections

**GPU-accelerated workloads**: Tasks that can utilize parallel processing
- Machine learning, scientific simulations, image processing
- Require appropriate GPU resources and memory

### Resource Profiling

Before optimizing, you need to understand your program's resource usage:

```bash
# Monitor CPU and memory usage
htop

# Time a program and get resource statistics  
/usr/bin/time -v ./your_program

# Monitor GPU usage (if available)
nvidia-smi

# Watch GPU usage continuously
watch -n 1 nvidia-smi
```

---
## Types of Jobs and Resources

| Job Type   | SLURM Partition | Key SLURM Options              | Example Use Case            |
|------------|------------------|-------------------------------|-----------------------------|
| Serial     | `serial`         | `--partition`, no MPI         | Single-thread tensor calc   |
| Parallel   | `defaultq`       | `-N`, `-n`, `mpirun`          | MPI simulation              |
| GPU        | `gpu`            | `--gpus`, `--cpus-per-task`   | Deep learning training      |

---
## Sequential Job Optimization

Sequential jobs run on a single CPU core and are suitable for tasks that cannot be parallelized.

### Sequential Job Script Explained

```bash
#!/bin/bash
#SBATCH -J jobname                    # Job name for identification
#SBATCH -o outfile.%J                 # Standard output file (%J = job ID)
#SBATCH -e errorfile.%J               # Standard error file (%J = job ID)
#SBATCH --partition=serial            # Use serial queue for single-core jobs
./[programme executable name]          # Execute your program
```

**Script breakdown:**
- `#!/bin/bash`: Specifies bash shell for script execution
- `#SBATCH -J jobname`: Sets a descriptive job name for easy identification in queue
- `#SBATCH -o outfile.%J`: Redirects standard output to a file with job ID
- `#SBATCH -e errorfile.%J`: Redirects error messages to separate file
- `#SBATCH --partition=serial`: Specifies the queue/partition for sequential jobs

### Example: Matrix Multiplication (Sequential)

```c
// matrix_mult_sequential.c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int n = 1000;  // Matrix size
    double **A, **B, **C;
    
    // Allocate memory for matrices
    A = (double**)malloc(n * sizeof(double*));
    B = (double**)malloc(n * sizeof(double*));
    C = (double**)malloc(n * sizeof(double*));
    
    for(int i = 0; i < n; i++) {
        A[i] = (double*)malloc(n * sizeof(double));
        B[i] = (double*)malloc(n * sizeof(double));
        C[i] = (double*)malloc(n * sizeof(double));
    }
    
    // Initialize matrices
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {
            A[i][j] = (double)rand() / RAND_MAX;
            B[i][j] = (double)rand() / RAND_MAX;
            C[i][j] = 0.0;
        }
    }
    
    clock_t start = clock();
    
    // Matrix multiplication
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {
            for(int k = 0; k < n; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }
    
    clock_t end = clock();
    double cpu_time = ((double)(end - start)) / CLOCKS_PER_SEC;
    
    printf("Sequential matrix multiplication completed in %f seconds\n", cpu_time);
    
    // Clean up memory
    for(int i = 0; i < n; i++) {
        free(A[i]); free(B[i]); free(C[i]);
    }
    free(A); free(B); free(C);
    
    return 0;
}
```

### Optimized Sequential Job Script

```bash
#!/bin/bash
#SBATCH -J matrix_sequential
#SBATCH -o matrix_seq_%J.out
#SBATCH -e matrix_seq_%J.err
#SBATCH --partition=serial
#SBATCH --time=00:30:00              # Set time limit
#SBATCH --mem=4G                     # Request 4GB memory

echo "Starting sequential matrix multiplication..."
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURM_NODELIST"

./matrix_mult_sequential

echo "Job completed at $(date)"
```

> ## Exercise: Profile Your Code
> Compile and run the sequential matrix multiplication. Use `time` and `htop` to monitor resource usage. Identify whether it's CPU-bound or memory-bound
{: .challenge}

## Parallel Job Optimization

Parallel jobs can utilize multiple CPU cores across one or more nodes to accelerate computation.

### Parallel Job Script Explained

```bash
#!/bin/bash
#SBATCH -J jobname                    # Job name
#SBATCH -o outfile.%J                 # Output file
#SBATCH -e errorfile.%J               # Error file
#SBATCH --partition=defaultq          # Parallel job queue
#SBATCH -N 2                          # Number of compute nodes
#SBATCH -n 24                         # Total number of CPU cores per node
mpirun -np 48 ./mpi_program           # Run with 48 MPI processes (2 nodes × 24 cores)
```

**Key parameters:**
- `#SBATCH -N 2`: Requests 2 compute nodes
- `#SBATCH -n 24`: Specifies 24 CPU cores per node
- `mpirun -np 48`: Launches 48 MPI processes total (2 × 24)

### Example: Matrix Multiplication (OpenMP Parallel)

```c
// matrix_mult_openmp.c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

int main() {
    int n = 1000;
    double **A, **B, **C;
    
    // Allocate memory (same as sequential version)
    A = (double**)malloc(n * sizeof(double*));
    B = (double**)malloc(n * sizeof(double*));
    C = (double**)malloc(n * sizeof(double*));
    
    for(int i = 0; i < n; i++) {
        A[i] = (double*)malloc(n * sizeof(double));
        B[i] = (double*)malloc(n * sizeof(double));
        C[i] = (double*)malloc(n * sizeof(double));
    }
    
    // Initialize matrices
    #pragma omp parallel for
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {
            A[i][j] = (double)rand() / RAND_MAX;
            B[i][j] = (double)rand() / RAND_MAX;
            C[i][j] = 0.0;
        }
    }
    
    double start_time = omp_get_wtime();
    
    // Parallel matrix multiplication
    #pragma omp parallel for
    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {
            for(int k = 0; k < n; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }
    
    double end_time = omp_get_wtime();
    
    printf("OpenMP matrix multiplication completed in %f seconds\n", end_time - start_time);
    printf("Number of threads used: %d\n", omp_get_max_threads());
    
    // Clean up memory
    for(int i = 0; i < n; i++) {
        free(A[i]); free(B[i]); free(C[i]);
    }
    free(A); free(B); free(C);
    
    return 0;
}
```

### Optimized Parallel Job Script

```bash
#!/bin/bash
#SBATCH -J matrix_openmp
#SBATCH -o matrix_omp_%J.out
#SBATCH -e matrix_omp_%J.err
#SBATCH --partition=defaultq
#SBATCH -N 1                         # Single node for shared memory
#SBATCH --ntasks-per-node=1          # One task per node
#SBATCH --cpus-per-task=24           # 24 CPU cores for OpenMP
#SBATCH --time=00:15:00
#SBATCH --mem=8G

# Set number of OpenMP threads
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

echo "Starting OpenMP matrix multiplication..."
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURM_NODELIST"
echo "Number of OpenMP threads: $OMP_NUM_THREADS"

./matrix_mult_openmp

echo "Job completed at $(date)"
```

> ## Exercise: Optimize Parallel Performance
> Compile the OpenMP version with different thread counts. Submit jobs with varying `--cpus-per-task` values. Plot performance vs. thread count
{: .challenge}

## GPU Job Optimization

GPU jobs leverage graphics processing units for massively parallel computations.

### GPU Job Script Explained

```bash
#!/bin/bash
#SBATCH --nodes=1                     # Single node (GPUs are node-local)
#SBATCH --ntasks-per-node=1           # One task per node
#SBATCH --cpus-per-task=4             # CPU cores to support GPU
#SBATCH -o output-%J.out              # Output file with job ID
#SBATCH -e error-%J.err               # Error file with job ID
#SBATCH --partition=gpu               # GPU-enabled partition
#SBATCH --mem 25G                     # Memory allocation
#SBATCH --gpus-per-node=1             # Number of GPUs requested
./[programme executable name]          # GPU program execution
```

**GPU-specific parameters:**
- `--partition=gpu`: Specifies GPU-enabled compute nodes
- `--gpus-per-node=1`: Requests one GPU per node
- `--mem 25G`: Allocates sufficient memory for GPU operations
- `--cpus-per-task=4`: Provides CPU cores to feed data to GPU

### Example: Matrix Multiplication (CUDA)

```cuda
// matrix_mult_cuda.cu
#include <stdio.h>
#include <cuda_runtime.h>

__global__ void matrixMult(float* A, float* B, float* C, int n) {
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;
    
    if (row < n && col < n) {
        float sum = 0.0f;
        for (int k = 0; k < n; k++) {
            sum += A[row * n + k] * B[k * n + col];
        }
        C[row * n + col] = sum;
    }
}

int main() {
    int n = 1000;
    size_t size = n * n * sizeof(float);
    
    // Host memory allocation
    float *h_A = (float*)malloc(size);
    float *h_B = (float*)malloc(size);
    float *h_C = (float*)malloc(size);
    
    // Initialize matrices
    for (int i = 0; i < n * n; i++) {
        h_A[i] = (float)rand() / RAND_MAX;
        h_B[i] = (float)rand() / RAND_MAX;
    }
    
    // Device memory allocation
    float *d_A, *d_B, *d_C;
    cudaMalloc(&d_A, size);
    cudaMalloc(&d_B, size);
    cudaMalloc(&d_C, size);
    
    // Copy data to device
    cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);
    
    // Launch kernel
    dim3 blockSize(16, 16);
    dim3 gridSize((n + blockSize.x - 1) / blockSize.x, 
                  (n + blockSize.y - 1) / blockSize.y);
    
    cudaEvent_t start, stop;
    cudaEventCreate(&start);
    cudaEventCreate(&stop);
    
    cudaEventRecord(start);
    matrixMult<<<gridSize, blockSize>>>(d_A, d_B, d_C, n);
    cudaEventRecord(stop);
    
    cudaEventSynchronize(stop);
    
    float milliseconds = 0;
    cudaEventElapsedTime(&milliseconds, start, stop);
    
    // Copy result back to host
    cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);
    
    printf("CUDA matrix multiplication completed in %f seconds\n", milliseconds/1000.0);
    
    // Cleanup
    free(h_A); free(h_B); free(h_C);
    cudaFree(d_A); cudaFree(d_B); cudaFree(d_C);
    cudaEventDestroy(start); cudaEventDestroy(stop);
    
    return 0;
}
```

### Example: Tensor Operations (Python + CUDA)

```python
# tensor_operations.py
import numpy as np
import time
try:
    import cupy as cp
    GPU_AVAILABLE = True
except ImportError:
    GPU_AVAILABLE = False
    print("CuPy not available, running CPU-only version")

def tensor_multiply_cpu(A, B):
    """CPU tensor multiplication using NumPy"""
    start_time = time.time()
    C = np.tensordot(A, B, axes=2)
    end_time = time.time()
    return C, end_time - start_time

def tensor_multiply_gpu(A, B):
    """GPU tensor multiplication using CuPy"""
    if not GPU_AVAILABLE:
        return None, 0
    
    # Transfer to GPU
    A_gpu = cp.asarray(A)
    B_gpu = cp.asarray(B)
    
    start_time = time.time()
    C_gpu = cp.tensordot(A_gpu, B_gpu, axes=2)
    cp.cuda.Stream.null.synchronize()  # Wait for GPU completion
    end_time = time.time()
    
    # Transfer back to CPU
    C = cp.asnumpy(C_gpu)
    return C, end_time - start_time

def main():
    # Create large tensors
    shape = (500, 500, 100)
    print(f"Creating tensors of shape {shape}")
    
    A = np.random.random(shape).astype(np.float32)
    B = np.random.random(shape).astype(np.float32)
    
    # CPU computation
    print("Running CPU tensor multiplication...")
    C_cpu, cpu_time = tensor_multiply_cpu(A, B)
    print(f"CPU time: {cpu_time:.4f} seconds")
    
    # GPU computation
    if GPU_AVAILABLE:
        print("Running GPU tensor multiplication...")
        C_gpu, gpu_time = tensor_multiply_gpu(A, B)
        print(f"GPU time: {gpu_time:.4f} seconds")
        
        if gpu_time > 0:
            speedup = cpu_time / gpu_time
            print(f"GPU speedup: {speedup:.2f}x")
            
            # Verify results match
            if np.allclose(C_cpu, C_gpu, rtol=1e-5):
                print("Results match between CPU and GPU!")
            else:
                print("Warning: Results differ between CPU and GPU")
    else:
        print("GPU computation skipped (CuPy not available)")

if __name__ == "__main__":
    main()
```
> ## Exercise: GPU vs CPU Comparison
> Run the tensor operations script on both CPU and GPU. Compare execution times and memory usage. Calculate the speedup factor
{: .challenge}

### Optimized GPU Job Script

```bash
#!/bin/bash
#SBATCH -J tensor_gpu
#SBATCH -o tensor_gpu_%J.out
#SBATCH -e tensor_gpu_%J.err
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8            # More CPUs for data preparation
#SBATCH --gpus-per-node=1
#SBATCH --mem=32G                    # More memory for large tensors
#SBATCH --time=00:20:00

# Load required modules
module load cuda/11.8
module load python/3.9

echo "Starting GPU tensor operations..."
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURM_NODELIST"
echo "GPU info:"
nvidia-smi --query-gpu=name,memory.total,memory.free --format=csv

# Run CUDA program
echo "Running CUDA matrix multiplication..."
./matrix_mult_cuda

echo ""
echo "Running Python tensor operations..."
python tensor_operations.py

echo "Job completed at $(date)"
```

## Resource Monitoring and Performance Analysis

### Monitoring Job Performance

```bash
#!/bin/bash
# monitor_job.sh - Script to monitor running jobs

# Get job efficiency statistics
seff $SLURM_JOB_ID

# Monitor real-time resource usage
while [ -f /proc/$$/comm ]; do
    echo "=== $(date) ==="
    echo "CPU Usage:"
    top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1"%"}'
    
    echo "Memory Usage:"
    free -h | grep '^Mem:' | awk '{print $3 "/" $2 " (" $3/$2*100 "%)"}'
    
    if command -v nvidia-smi &> /dev/null; then
        echo "GPU Usage:"
        nvidia-smi --query-gpu=utilization.gpu,memory.used,memory.total --format=csv,noheader,nounits
    fi
    
    sleep 30
done
```

### Performance Comparison Script

```python
# performance_compare.py
import subprocess
import json
import matplotlib.pyplot as plt

def run_benchmark(script_name, job_name):
    """Submit job and collect performance metrics"""
    # Submit job
    result = subprocess.run(['sbatch', script_name], capture_output=True, text=True)
    job_id = result.stdout.strip().split()[-1]
    
    # Wait for completion (simplified - in practice, use more sophisticated polling)
    subprocess.run(['squeue', '-j', job_id, '-h', '--format=%T'], capture_output=True)
    
    # Get efficiency statistics
    seff_result = subprocess.run(['seff', job_id], capture_output=True, text=True)
    
    return {
        'job_name': job_name,
        'job_id': job_id,
        'efficiency_output': seff_result.stdout
    }

def parse_timing_from_output(output_file):
    """Extract timing information from job output"""
    with open(output_file, 'r') as f:
        content = f.read()
        # Extract timing information (implementation depends on output format)
        # This is a placeholder - implement based on your actual output format
        return 0.0

def create_performance_chart():
    """Create performance comparison visualization"""
    methods = ['Sequential', 'OpenMP', 'CUDA']
    times = [120.5, 15.2, 3.8]  # Example times in seconds
    
    plt.figure(figsize=(10, 6))
    bars = plt.bar(methods, times, color=['blue', 'green', 'red'])
    plt.ylabel('Execution Time (seconds)')
    plt.title('Matrix Multiplication Performance Comparison')
    
    # Add value labels on bars
    for bar, time in zip(bars, times):
        plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 1,
                f'{time:.1f}s', ha='center', va='bottom')
    
    plt.tight_layout()
    plt.savefig('performance_comparison.png', dpi=300, bbox_inches='tight')
    plt.show()

if __name__ == "__main__":
    create_performance_chart()
```

> ## Exercise: Resource Efficiency Analysis
> Submit jobs with different resource allocations. Use `seff` to analyze job efficiency. Identify optimal resource configurations
{: .challenge}

## Best Practices and Common Pitfalls

### Resource Allocation Best Practices

1. **Match resources to workload requirements**
   - Don't request more resources than you can use
   - Consider memory requirements carefully
   - Use appropriate partitions/queues

2. **Test with small jobs first**
   - Validate your scripts with shorter runs
   - Check resource utilization before scaling up

3. **Monitor and optimize**
   - Use profiling tools to identify bottlenecks
   - Adjust resource requests based on actual usage

### Common Mistakes to Avoid

1. **Over-requesting resources**
   ```bash
   # Bad: Requesting 32 cores for sequential code
   #SBATCH --cpus-per-task=32
   ./sequential_program
   
   # Good: Match core count to parallelization
   #SBATCH --cpus-per-task=1
   ./sequential_program
   ```

2. **Memory allocation errors**
   ```bash
   # Bad: Not specifying memory for memory-intensive jobs
   #SBATCH --partition=defaultq
   
   # Good: Specify adequate memory
   #SBATCH --partition=defaultq
   #SBATCH --mem=16G
   ```

3. **GPU job inefficiencies**
   ```bash
   # Bad: Too many CPU cores for GPU job
   #SBATCH --cpus-per-task=32
   #SBATCH --gpus-per-node=1
   
   # Good: Balanced CPU-GPU ratio
   #SBATCH --cpus-per-task=4
   #SBATCH --gpus-per-node=1
   ```




## Summary

Resource optimization in HPC involves understanding your workload characteristics and matching them with appropriate resource allocations. Key takeaways:

- Profile your code to understand resource requirements
- Use sequential jobs for single-threaded applications
- Leverage parallel computing for scalable workloads
- Utilize GPUs for massively parallel computations
- Monitor performance and adjust allocations accordingly
- Avoid common pitfalls like over-requesting resources

Efficient resource utilization not only improves your job performance but also ensures fair access to shared HPC resources for all users.


{% include links.md %}