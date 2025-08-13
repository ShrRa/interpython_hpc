---
title: "Resource optimization"
start: False
teaching: 30
exercises: 10
questions:
- ""
keypoints:
- ""
---

## Example

For understanding how we can utilise different resources available on the HPC for the same computational task, we take the example of a python code which calculates the Gravitational Deflection Angle defined in the following way: 

### Deflection Angle Formula

For light passing near a massive object, the deflection angle (α) in the weak-field approximation is given by:

```
α = 4GM / (c²b)
```

Where:

- G = Gravitational constant (6.67430 × 10⁻¹¹ m³ kg⁻¹ s⁻²)
- M = Mass of the lensing object (in kilograms)
- c = Speed of light (299792458 m/s)
- b = Impact parameter (the closest approach distance of the light ray to the mass, in meters)

## Computational Task Description

Compute the deflection angle over a grid of:

- Mass values: From 1 to 1000 solar masses (10³⁰ to 10³³ kg)
- Impact parameters: From 10⁹ to 10¹² meters

Generate a 2D array where each entry corresponds to the deflection angle for a specific pair of mass and impact parameter. Now we will look at how we will implement this for the different resources available on the HPC. 

<!-------------------------------------------- Section-1 ------------------------------------------------------------------>
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

### Example: Gravitational Deflection Angle Sequential CPU
```python
import numpy as np
import time
import matplotlib.pyplot as plt
import os
import matplotlib.colors as colors

# Constants
G = 6.67430e-11
c = 299792458
M_sun = 1.98847e30

# Parameter grid
mass_grid = np.linspace(1, 1000, 10000)  # Solar masses
impact_grid = np.linspace(1e9, 1e12, 10000)  # meters

result = np.zeros((len(mass_grid), len(impact_grid)))

# Timing
start = time.time()

# Sequential computation
for i, M in enumerate(mass_grid):
    for j, b in enumerate(impact_grid):
        result[i, j] = (4 * G * M * M_sun) / (c**2 * b)

end = time.time()

print(f"CPU Sequential time: {end - start:.3f} seconds")

result = np.save("result_cpu.npy", result)
mass_grid = np.save("mass_grid_cpu.npy", mass_grid)
impact_grid = np.save("impact_grid_cpu.npy", impact_grid)

# Load data
result = np.load("result_cpu.npy")
mass_grid = np.load("mass_grid_cpu.npy")
impact_grid = np.load("impact_grid_cpu.npy")

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
plt.title('Gravitational Deflection Angle - CPU')

plt.tight_layout()
plt.savefig("plots/deflection_angle_cpu.png", dpi=300)
plt.close()

print("CPU plot saved in 'plots/deflection_angle_cpu.png'")
```

<!-- ```c
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
``` -->

### Sequential Job Script for the Example

<!-- ```bash
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
``` -->

```bash
#!/bin/bash
#SBATCH --job-name=HPC_WS_SCPU # Provide a name for the job 
#SBATCH --output=HPC_WS_SCPU_%j.out # Request the output file along with the job number
#SBATCH --error=HPC_WS_SCPU_%j.err # Request the error file along with the job number
#SBATCH --partition=serial
#SBATCH --nodes=1 # Request one CPU node
#SBATCH --ntasks=1 # Request 1 core from the CPU node
#SBATCH --time=-01:00:00 # Set time limit for the job
#SBATCH --mem=16G #Request 16GB memory 

# Load required modules
module purge # Remove the list of pre loaded modules
module load Python/3.9.1
module list

# Create a python virtual environment 
python3 -m venv name_of_your_venv

# Activate your Python environment
source name_of_your_venv/bin/activate

echo "Starting Gravitational Lensing Deflection calculation of Sequential CPU..."
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURM_NODELIST"

# Run the Python script (with logging)
python Gravitational_Deflection_Angle_SCPU.py

echo "Job completed at $(date)"
```

> ## Exercise: Profile Your Code
> Compile and run the sequential code. Use `htop` to monitor resource usage. Identify whether it's CPU-bound or memory-bound
{: .challenge}

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

```python
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
```
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

### Parallel Job Script for the Example

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

```bash
#!/bin/bash
#SBATCH --job-name=HPC_WS_PCPU # Provide a name for the job 
#SBATCH --output=HPC_WS_PCPU_%j.out # Request the output file along with the job number
#SBATCH --error=HPC_WS_PCPU_%j.err # Request the error file along with the job number
#SBATCH --partition=defaultq 
#SBATCH --nodes=2 # Request two CPU nodes
#SBATCH --ntasks=4 # Request 2 cores from each CPU node
#SBATCH --time=-01:00:00 # Set time limit for the job
#SBATCH --mem=16G #Request 16GB memory 

# Load required modules
module purge # Remove the list of pre loaded modules
module load Python/3.9.1
module load openmpi4/default
module list # List the modules

# Create a python virtual environment 
python3 -m venv name_of_your_venv

# Activate your Python virtual environment
source name_of_your_venv/bin/activate

echo "Starting Gravitational Lensing Deflection calculation of Sequential CPU..."
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $SLURM_NODELIST"

# Run the Python script with MPI (with logging)
mpirun -np 4 python Gravitational_Lensing_PCPU.py

echo "Job completed at $(date)"
```

> ## Exercise: Optimize Parallel Performance
> Compile the OpenMP version with different thread counts. Submit jobs with varying `--cpus-per-task` values. Plot performance vs. thread count
{: .challenge}

<!-------------------------------------------- Section-3 ------------------------------------------------------------------>
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
#SBATCH --mem 32G                     # Memory allocation
#SBATCH --gpus-per-node=1             # Number of GPUs requested
./[programme executable name]          # GPU program execution
```

**GPU-specific parameters:**
- `--partition=gpu`: Specifies GPU-enabled compute nodes
- `--gpus-per-node=1`: Requests one GPU per node
- `--mem 32G`: Allocates sufficient memory for GPU operations
- `--cpus-per-task=4`: Provides CPU cores to feed data to GPU

### Example: CUDA Implementation

```python
import numpy as np
from numba import cuda
import time
import matplotlib.pyplot as plt
import os
import matplotlib.colors as colors


# Constants
G = 6.67430e-11
c = 299792458

# Parameter grid
mass_grid = np.linspace(1e30, 1e33, 10000)
impact_grid = np.linspace(1e9, 1e12, 10000)

mass_grid_device = cuda.to_device(mass_grid)
impact_grid_device = cuda.to_device(impact_grid)
result_device = cuda.device_array((len(mass_grid), len(impact_grid)))

# CUDA kernel
@cuda.jit
def compute_deflection(mass_array, impact_array, result):
    i, j = cuda.grid(2)
    if i < mass_array.size and j < impact_array.size:
        M = mass_array[i]
        b = impact_array[j]
        result[i, j] = (4 * G * M) / (c**2 * b)

# Setup thread/block dimensions
threadsperblock = (16, 16)
blockspergrid_x = (mass_grid.size + threadsperblock[0] - 1) // threadsperblock[0]
blockspergrid_y = (impact_grid.size + threadsperblock[1] - 1) // threadsperblock[1]
blockspergrid = (blockspergrid_x, blockspergrid_y)

# Run the kernel
start = time.time()
compute_deflection[blockspergrid, threadsperblock](mass_grid_device, impact_grid_device, result_device)
cuda.synchronize()
end = time.time()

result = result_device.copy_to_host()

print(f"CUDA time: {end - start:.3f} seconds")

# Save the result and grids
np.save("result_cuda.npy", result)
np.save("mass_grid_cuda.npy", mass_grid)
np.save("impact_grid_cuda.npy", impact_grid)

print("Result and grids saved as .npy files.")

# Load data
result = np.load("result_cuda.npy")
mass_grid = np.load("mass_grid_cuda.npy")
impact_grid = np.load("impact_grid_cuda.npy")

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
plt.title('Gravitational Deflection Angle - CUDA')

plt.tight_layout()
plt.savefig("plots/deflection_angle_cuda.png", dpi=300)
plt.close()

print("CUDA plot saved in 'plots/deflection_angle_cuda.png'")
```
### GPU Job Script for the Example

```bash
#!/bin/bash
#SBATCH --job-name=HPC_WS_GPU  # Provide a name for the job 
#SBATCH --output=HPC_WS_GPU_%j.out
#SBATCH --error=HPC_WS_GPU_%j.err
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4 # Number of CPUs for data preparation 
#SBATCH --mem=32G # Memmory allocation
#SBATCH --gpus-per-node=1
#SBATCH --time=06:00:00

# --------- Load Environment ---------
module load Python/3.9.1
module load cuda/11.2
module list

# Activate your Python virtual environment
source name_of_your_venv/bin/activate

# --------- Run the Python Script ---------
python Gravitational_Lensing_GPU.py
```

> ## Exercise: GPU vs CPU Comparison
> Run the tensor operations script on both CPU and GPU. Compare execution times and memory usage. Calculate the speedup factor
{: .challenge}

{% include links.md %}
