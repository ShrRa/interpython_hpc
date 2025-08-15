---
title: "Resource optimization and monitoring for GPU Jobs"
start: False
teaching: 30
exercises: 10
questions:
- ""
keypoints:
- ""
---

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

~~~
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
~~~
{: .language-python}

 ~~~
    CUDA time: 0.341 seconds
    Result and grids saved as .npy files.
    CUDA plot saved in 'plots/deflection_angle_cuda.png'
 ~~~
 {: .output}
 
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

<!-- Just to check -->
{% include links.md %}