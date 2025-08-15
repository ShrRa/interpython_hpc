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

{% include links.md %}