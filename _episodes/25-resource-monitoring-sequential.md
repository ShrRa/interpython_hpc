---
title: "Resource optimization and monitoring for Serial Jobs"
start: False
teaching: 30
exercises: 10
questions:
- ""
objectives:
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
~~~
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
~~~
{: .language-python}

 ~~~
 CPU Sequential time: 153.965 seconds
 CPU plot saved in 'plots/deflection_angle_cpu.png'
 ~~~
 {: .output}

### Job Monitoring and Profiling 

We would also want to monitor the resources for the job before we run the job, so we can decide if we allocated the right amount of resources for the job type. For this we will need to create a shell file which logs the CPU and Memory resource usage every five seconds. We can create that file using the code below

```shell
#File: monitor_resources.sh
#!/bin/bash
# Monitor CPU% and Memory usage of Python processes for the user (you)
# Saves results in a log file

OUTFILE="resource_usage_${SLURM_JOB_ID}.log"

# Create a header row for the log file
echo "Timestamp | CPU% | Memory(MB)" > "$OUTFILE"

# Repeat until stopped
while true
do
    # ps: shows running processes
    # -u $USER : only show processes owned by you
    # -o %cpu,rss,comm : output CPU%, memory (RSS in KB), and command name
    ps -u $USER -o %cpu,rss,comm \
    | awk '
        $3=="python" {                   # Only lines where command is "python"
            # strftime formats current date/time
            # $1 is CPU%, $2 is memory in KB — divide by 1024 for MB
            print strftime("%Y-%m-%d %H:%M:%S"), "|", $1, "|", $2/1024
        }
    ' >> "$OUTFILE"

    # sleep: pause for 5 seconds before checking again
    sleep 5
done
```
We can now include a command to run this file in the slurm job script that we will use to run the sequential example on BURA. 

### Sequential Job Script for the Example

```bash
#!/bin/bash
#SBATCH --job-name=SCPU
#SBATCH --output=SCPU_%j.out
#SBATCH --error=SCPU_%j.err
#SBATCH --partition=computes_thin
#SBATCH --nodes=1
#SBATCH --ntasks=1          
#SBATCH --time=00:10:00
#SBATCH --mem=16G

# ----------------------------
# Print the list of the loaded modules
# ----------------------------
module list

# ----------------------------
# Activate Python environment
# ----------------------------
source interpython/bin/activate
python --version

# ----------------------------
# Start the resource monitor
# ----------------------------
# The monitor_resources.sh script must be in the same directory
 bash monitor_resources.sh &
# ----------------------------
# Run the main sequential job
# ----------------------------
 python Gravitational_Lensing_SCPU.py

# ----------------------------
# Stop the monitor after the job finishes
# ----------------------------
kill %1

echo "Job completed at $(date)"
echo "Resource usage saved to resource_usage_${SLURM_JOB_ID}.log"
```
After we run the script, we can then 
> ## Exercise: Profile Your Code
> Compile and run the sequential code. Use `htop` to monitor resource usage. Identify whether it's CPU-bound or memory-bound
{: .challenge}

{% include links.md %}


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