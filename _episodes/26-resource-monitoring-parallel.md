---
title: "Resource optimization and monitoring for Parallel Jobs"
start: False
teaching: 30
exercises: 10
questions:
- "How do we optimize resource requests for parallel jobs on an HPC system?"
- "What are common pitfalls when requesting CPUs, memory, or nodes?"
- "How can we monitor parallel job performance to adjust allocations?"
objectives:
- "Understand how to request the correct number of nodes, tasks, and memory for MPI jobs."
- "Learn best practices for parallel job submission to avoid wasted resources."
- "Use a monitoring script to track CPU and memory usage of parallel jobs in real-time."
keypoints:
- "Match `--nodes`, `--ntasks`, and `--cpus-per-task` to the parallelism strategy (MPI vs OpenMP)."
- "Avoid over-requesting resources—requesting more cores than used wastes allocations."
- "Monitor CPU and memory usage during job execution to guide resource tuning."
---

<!-------------------------------------------- Section-2 ------------------------------------------------------------------>
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

**Changes from the sequential script:**
- `#SBATCH --partition=defaultq` : Sets to the default partition
- `#SBATCH -N 2`: Requests 2 compute nodes
- `#SBATCH -n 24`: Specifies 24 CPU cores per node
- `mpirun -np 48`: Launches 48 MPI processes total (2 × 24)

### Example: Gravitational Deflection Angle Parallel CPU

~~~
# File Name - deflection_angle_mpi.py
# This script computes the gravitational deflection angle of light around a massive object
# using MPI for parallelization across multiple processes. Each MPI rank computes a chunk
# of the mass grid, results are gathered at the root process, and a color plot is generated
# to visualize the deflection angles on a logarithmic scale.

# Import MPI module from mpi4py, NumPy for numerical array operations, time for measuring execution time 
# Import os for appending paths, and matplotlib for plotting the results
from mpi4py import MPI
import numpy as np
import time
import os 
import matplotlib.pyplot as plt
import matplotlib.colors as colors

# MPI setup
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Constants
G = 6.67430e-11
c = 299792458
M_sun = 1.98847e30

# Parameter grid (same on all ranks)
mass_grid = np.linspace(1, 1000, 10000)  # Solar masses
impact_grid = np.linspace(1e9, 1e12, 10000)  # meters

# Distribute mass grid among ranks
chunk_size = len(mass_grid) // size
start_idx = rank * chunk_size
end_idx = (rank + 1) * chunk_size if rank != size - 1 else len(mass_grid)

local_mass = mass_grid[start_idx:end_idx]
local_result = np.zeros((len(local_mass), len(impact_grid)))

# Timing
local_start = time.time()

# Compute local chunk
for i, M in enumerate(local_mass):
    for j, b in enumerate(impact_grid):
        local_result[i, j] = (4 * G * M * M_sun) / (c**2 * b)

local_end = time.time()
print(f"Rank {rank} local time: {local_end - local_start:.3f} seconds")

# Gather results
result = None
if rank == 0:
    result = np.zeros((len(mass_grid), len(impact_grid)))

comm.Gather(local_result, result, root=0)

if rank == 0:
    total_time = local_end - local_start
    print(f"MPI total time (wall time): {total_time:.3f} seconds")
    result = np.save("result_mpi.npy", result)
    mass_grid = np.save("mass_grid_mpi.npy", mass_grid)
    impact_grid = np.save("impact_grid_mpi.npy", impact_grid)

# Load data
result = np.load("result_mpi.npy")
mass_grid = np.load("mass_grid_mpi.npy")
impact_grid = np.load("impact_grid_mpi.npy")

# Create meshgrid
M, B = np.meshgrid(mass_grid / 1.989e30, impact_grid / 1e9, indexing='ij')

# Create output directory
os.makedirs("plots", exist_ok=True)

plt.figure(figsize=(8,6))
pcm = plt.pcolormesh(B, M, result,
                      norm=colors.LogNorm(vmin=result[result > 0].min(), vmax=result.max()),
                      shading='auto', cmap='plasma')

plt.colorbar(pcm, label='Deflection Angle (radians, log scale)')
plt.xlabel('Impact Parameter (Gm)')
plt.ylabel('Mass (Solar Masses)')
plt.title('Gravitational Deflection Angle - MPI')

plt.tight_layout()
plt.savefig("plots/deflection_angle_mpi.png", dpi=300)
plt.close()

print("MPI plot saved in 'plots/deflection_angle_mpi.png'")
~~~
{: .language-python}

 ~~~
    Rank 2 local time: 19.160 seconds
    Rank 1 local time: 18.160 seconds
    Rank 3 local time: 17.322 seconds
    Rank 0 local time: 19.576 seconds
    MPI total time (wall time): 19.576 seconds
    MPI plot saved in 'plots/deflection_angle_mpi.png'
 ~~~
 {: .output}

We would now again want to monitor the resources, this time for the parallel job, so we can decide if we allocated the right amount of resources for the job. For this we will need to create a shell file which logs the CPU and Memory resource usage of the MPI job every five seconds. We can create that file using the code below

```shell
#File: monitor_resources_parallel.sh
#!/bin/bash
# Monitor CPU% and Memory usage of parallel Python (MPI) processes
# Saves results in a log file for later analysis

OUTFILE="resource_usage_${SLURM_JOB_ID}.log"

# Create header
echo "Timestamp | PID | CPU% | Memory(MB) | Command" > "$OUTFILE"

# Repeat until stopped
while true
do
    # List all processes owned by user
    # Filter only python commands (MPI ranks running Python)
    ps -u $USER -o pid,%cpu,rss,comm \
    | awk '
        $4=="python" {
            # rss is memory in KB, convert to MB
            print strftime("%Y-%m-%d %H:%M:%S"), "|", $1, "|", $2, "|", $3/1024, "|", $4
        }
    ' >> "$OUTFILE"

    # Wait 5 seconds before next sample
    sleep 5
done
```

### Parallel Job Script for the Example

```bash
#!/bin/bash
#SBATCH --job-name=PCPU # Name of the Job
#SBATCH --output=PCPU_%j.out # Name of the output file for the Job
#SBATCH --error=PCPU_%j.err # Name of the error file for the Job
#SBATCH --partition=computes_thin # Request the appropriate partition for the job
#SBATCH --nodes=2 # Request the appropriate number of computing nodes required for the job
#SBATCH --ntasks=4 # This specifies how many mpi processes will run across the nodes
#SBATCH --time=00:10:00 # This specifies the maximum amount of time that the job will run for
#SBATCH --mem=16G # This specifies the amount of memory which will be allocated for the job

# Load required modules (This is a sanity check in case jobs are not running as required)
module list

# Activate your virtual environment (We have already activated this in terminal so this again a sanity check)
source interpython/bin/activate

# Start the resource monitor in the background.
# The "&" symbol is used so the monitor runs simultaneously with the main job instead of blocking it
# The monitor_resources_parallel.sh script must be in the same directory as the python file and the slurm script.
bash monitor_resources_parallel.sh &

# Save the process ID (PID) of the resource monitor.
# In the serial example, we used "kill %1" to stop the first background job.
# Here we use "$!" to capture the exact PID of the last background process (the monitor), which is safer and more robust. 
# Unlike "%1", this works even if there are multiple background processes, since it directly targets the correct one.
MONITOR_PID=$!

echo "Starting MPI job..."

# Run the Python mpi script, here the -np flag specifies the number of processes (copies) the mpi program will run 
mpirun -np 4 python example_parallel.py

# Stop the resource monitor once the job is done.
# This ensures the monitor doesn’t keep running after the main program finishes.
kill $MONITOR_PID

# Print the date and time when the job completed.
echo "Job completed at $(date)"

# Print the name of the log file which was preapared using the resource monitor script
echo "Resource usage saved to resource_usage_${SLURM_JOB_ID}.log"
```

After we run the script, we can then `cat` into the `resource_usage_${SLURM_JOB_ID}.log` file to view the logged CPU and memory usage over time.  

### Viewing the Results
To quickly view the contents of the log file:
```bash
cat resource_usage_${SLURM_JOB_ID}.log
```
```text
Timestamp            | PID   | CPU% | Memory(MB) | Command
2025-08-17 15:01:12  | 12345 | 98.5 | 250.1      | python
2025-08-17 15:01:17  | 12346 | 97.8 | 248.9      | python
```

## Monitoring the parallel script using `psutil`

A more convenient approach for job monitoring is to use the Python package `psutil`. It allows us to collect resource usage information from within our script itself.

```python
# File Name - deflection_angle_mpi_monitor.py
# This script computes the gravitational deflection angle of light around a massive object
# using MPI for parallelization across multiple processes. Each rank computes a portion
# of the mass grid and the results are gathered at the root process. 
# Additionally, each rank launches a background monitoring thread that periodically logs
# CPU and memory usage during execution. The root process generates a heatmap plot of the
# deflection angle results using a logarithmic color scale.

# Import MPI module from mpi4py, NumPy for numerical array operations, time for measuring execution time
# Import os for appending paths, matplotlib for plotting the results, and psutil for resource monitoring
from mpi4py import MPI
import numpy as np
import time
import os
import psutil
import threading
import matplotlib.pyplot as plt
import matplotlib.colors as colors

# MPI setup
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Constants
G = 6.67430e-11
c = 299792458
M_sun = 1.98847e30

# Monitoring setup
pid = os.getpid()
process = psutil.Process(pid)

def monitor(interval=5):
    """Continuously log CPU and memory usage every `interval` seconds."""
    while True:
        cpu = process.cpu_percent(interval=None)   # CPU usage %
        mem = process.memory_info().rss / (1024*1024)  # Memory in MB
        print(f"[Rank {rank}] CPU%: {cpu:.1f} | Mem: {mem:.2f} MB", flush=True)
        time.sleep(interval)

# Start monitoring thread (all ranks or just rank 0)
t = threading.Thread(target=monitor, daemon=True)
t.start()

# Parameter grid (same on all ranks)
mass_grid = np.linspace(1, 1000, 10000)   # Solar masses
impact_grid = np.linspace(1e9, 1e12, 10000)  # meters

# Divide work among ranks
chunk_size = len(mass_grid) // size
start_idx = rank * chunk_size
end_idx = (rank + 1) * chunk_size if rank != size - 1 else len(mass_grid)

local_mass = mass_grid[start_idx:end_idx]
local_result = np.zeros((len(local_mass), len(impact_grid)))

# Timing + Computation
local_start = time.time()

for i, M in enumerate(local_mass):
    for j, b in enumerate(impact_grid):
        local_result[i, j] = (4 * G * M * M_sun) / (c**2 * b)

local_end = time.time()
print(f"[Rank {rank}] Local compute time: {local_end - local_start:.3f} seconds")

# Gather results
result = None
if rank == 0:
    result = np.zeros((len(mass_grid), len(impact_grid)))

comm.Gather(local_result, result, root=0)

# Post-processing + Plotting
if rank == 0:
    total_time = local_end - local_start
    print(f"[Rank 0] MPI total time (wall time): {total_time:.3f} seconds")

    # Save results
    np.save("result_mpi.npy", result)
    np.save("mass_grid_mpi.npy", mass_grid)
    np.save("impact_grid_mpi.npy", impact_grid)

    # Reload for plotting
    result = np.load("result_mpi.npy")
    mass_grid = np.load("mass_grid_mpi.npy")
    impact_grid = np.load("impact_grid_mpi.npy")

    # Create meshgrid
    M, B = np.meshgrid(mass_grid, impact_grid / 1e9, indexing='ij')

    # Create output directory
    os.makedirs("plots", exist_ok=True)

    # Plot results
    plt.figure(figsize=(8,6))
    pcm = plt.pcolormesh(B, M, result,
                         norm=colors.LogNorm(vmin=result[result > 0].min(), vmax=result.max()),
                         shading='auto', cmap='plasma')

    plt.colorbar(pcm, label='Deflection Angle (radians, log scale)')
    plt.xlabel('Impact Parameter (Gm)')
    plt.ylabel('Mass (Solar Masses)')
    plt.title('Gravitational Deflection Angle - MPI with Monitoring')

    plt.tight_layout()
    plt.savefig("plots/deflection_angle_mpi.png", dpi=300)
    plt.close()

    print("MPI plot saved in 'plots/deflection_angle_mpi.png'")
```
## What Changed in the code

Compared to the earlier version of the MPI script, the scientific workflow (splitting work across ranks, computing deflection angles, gathering results, and plotting) remains the same.  

The **main change** in the final version is the addition of a **resource monitoring component**:

1. **psutil integration**  
   - Each MPI rank now imports and uses the `psutil` library.  
   - `psutil.Process()` gives access to the rank’s own process information (CPU usage, memory usage, etc.).  

2. **Background monitoring thread**  
   - A lightweight thread is started on each rank.  
   - This thread runs independently of the computation, waking up every few seconds to record:  
     - CPU percentage used by that rank.  
     - Memory usage in MB.  
   - The results are printed with the rank number (e.g., `[Rank 2] CPU%: 99.8 | Mem: 350.25 MB`).  

3. **Unified output**  
   - Instead of saving to a separate log file (like the shell script), the monitoring results are written directly into the job’s standard output, alongside the scientific results.  
   - This way, you can see both the computation progress *and* the resource usage together in real time.  

In short, the **final script is the original scientific MPI program plus a built-in live performance monitor**, achieved by combining `psutil` (to gather resource stats) with `threading` (to run the monitor in the background without interrupting the main calculations).

### Viewing the Results
To quickly view the contents of the log file:
```bash
cat PCPU_${SLURM_JOB_ID}.out
```
```text 
[Rank 1] CPU%: 0.0 | Mem: 219.05 MB
[Rank 0] CPU%: 0.0 | Mem: 230.35 MB
[Rank 2] CPU%: 0.0 | Mem: 230.82 MB
[Rank 3] CPU%: 0.0 | Mem: 227.89 MB
[Rank 1] CPU%: 100.2 | Mem: 272.06 MB
[Rank 0] CPU%: 100.0 | Mem: 279.32 MB
[Rank 2] CPU%: 100.2 | Mem: 283.75 MB
[Rank 3] CPU%: 100.4 | Mem: 280.70 MB
[Rank 1] CPU%: 100.2 | Mem: 328.06 MB
[Rank 0] CPU%: 100.4 | Mem: 329.32 MB
[Rank 2] CPU%: 100.2 | Mem: 331.75 MB
[Rank 3] CPU%: 100.1 | Mem: 332.70 MB
[Rank 1] CPU%: 100.1 | Mem: 384.98 MB
[Rank 0] CPU%: 99.9 | Mem: 383.32 MB
[Rank 2] CPU%: 100.1 | Mem: 383.75 MB
[Rank 3] CPU%: 99.9 | Mem: 388.70 MB
[Rank 1] Local compute time: 17.564 seconds
[Rank 3] Local compute time: 18.035 seconds
[Rank 2] Local compute time: 18.849 seconds
[Rank 0] Local compute time: 18.754 seconds
[Rank 0] MPI total time (wall time): 18.754 seconds
```
We can use the same reference used in the sequential section to understand the resource patterns and allocate the correct amount of resources for our job. 

Having understood both the results we can now draw a comparision between both the methods by using the following table

## Comparision between using a shell script and psutil
| Aspect                  | Shell Script (`monitor_resources_parallel.sh`)    | Python + psutil (Final Script)          |
|--------------------------|--------------------------------------------------|------------------------------------------|
| Where it runs            | Separate job alongside the MPI program           | Inside the MPI program itself             |
| Level of detail          | Per-process (just PID and command name)          | Per-rank, clearly tagged as `[Rank N]`   |
| Output location          | Written to a separate log file                   | Integrated directly into the job’s stdout|
| Setup effort             | Requires maintaining an extra monitoring script | Built into the code, no extra setup      |
| Continuous logging       | Yes, with fixed `sleep` interval                 | Yes, background thread with custom interval |
| Flexibility              | Works with any process, even non-Python ones     | Requires code access, Python only        |
| Best use case            | Black-box monitoring on shared HPC systems       | debugging, reproducibility     |

## Best Practices and Common Pitfalls for Resource Allocation for Parallel Scripts

### Resource Allocation Best Practices

1. **Match tasks to algorithm design**  
   - Use `--ntasks=N` for MPI programs where each process runs on its own core.  
   - Use `--cpus-per-task=M` for threaded/OpenMP programs that share memory.  
   - For hybrid codes (MPI + OpenMP), request both: `--ntasks=N --cpus-per-task=M`.  

2. **Distribute across nodes carefully**  
   - Use `--nodes` and `--ntasks-per-node` to control placement.  
   - Example: `--nodes=2 --ntasks-per-node=8` runs 16 MPI ranks across 2 nodes.  
   - Always check your cluster’s topology (socket, core counts) for best placement.  

3. **Request memory per node, not per task (unless required)**  
   - Use `--mem=64G` to request memory for the entire node.  
   - If memory scales per process, use `--mem-per-cpu` instead.  
   - Rule of thumb: multiply expected memory per rank by `ntasks`.  

4. **Estimate wall time realistically**  
   - Parallel scaling reduces runtime, but communication overhead adds cost.  
   - Test smaller runs and extrapolate to larger core counts.  
   - Always request a little more time than expected to avoid job termination.  

5. **Monitor scaling efficiency**  
   - Collect wall time and speedup vs. process count.  
   - Use scaling plots and Amdahl’s Law to understand diminishing returns.  
   - Helps avoid “oversubscription” (too many processes for too little gain).  

### Common Pitfalls for Parallel Jobs

1. **Over-requesting resources**
```bash
# Bad: Requesting 128 cores for a code that scales only to 32
#SBATCH --ntasks=128

# Good: Match tasks to measured scalability
#SBATCH --ntasks=32
```

2. **Memory allocation errors**
```bash
# Bad: Forgetting memory requests for large MPI jobs
#SBATCH --nodes=2
./parallel_program
# Good: Explicit memory per node
#SBATCH --nodes=2
#SBATCH --mem=64G
./parallel_program
```

<!-- > ## Exercise: Optimize Parallel Performance
> 
> Run the MPI version of the gravitational lensing code with 2, 4, 8, 16, and 32 processes.
> Measure wall time for each run and plot speedup vs. process count.
> Compare results to Amdahl’s Law prediction.
{: .challenge} -->

{% include links.md %}


<!-- ```bash
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
``` -->

<!-- ```c
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
} -->
<!-- ``` -->