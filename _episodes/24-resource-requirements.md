---
title: "Resource requirements"
start: False
teaching: 30
exercises: 10
questions:
- "What is the difference between requesting for CPU and GPU resources using Slurm?"
- "How can I optimize my slurm script to avail the best resources for my specific task?"
objectives:
- "Understand different types of computational workloads and their resource requirements"
- "Write optimized Slurm job scripts for sequential, parallel, and GPU workloads"
- "Monitor and analyze resource utilization"
- "Apply best practices for efficient resource allocation"
keypoints:
- "Different computational models (sequential, parallel, GPU) significantly impact runtime and efficiency."
- "Sequential CPU execution is simple but inefficient for large parameter spaces."
- "Parallel CPU (e.g., MPI or OpenMP) reduces runtime by distributing tasks but is limited by CPU core counts and communication overhead."
- "GPU computing can drastically accelerate tasks with massively parallel workloads like grid-based simulations."
- "Choosing the right computational model depends on the problem structure, resource availability, and cost-efficiency."
- "Effective Slurm job scripts should match the workload to the hardware: CPUs for serial/parallel, GPUs for highly parallelizable tasks."
- "Monitoring tools (like `nvidia-smi`, `seff`, `top`) help validate whether the resource request matches the actual usage."
- "Optimizing resource usage minimizes wait times in shared environments and improves overall throughput."
---

## Understanding Resource Requirements

#### Different computational tasks have varying resource requirements. Understanding these patterns is crucial for efficient HPC usage.

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

<!-- ### Resource Profiling

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
``` -->

---
## Types of Jobs and Resources

| Job Type   | SLURM Partition | Key SLURM Options              | Example Use Case            |
|------------|------------------|-------------------------------|-----------------------------|
| Serial     | `serial`         | `--partition`, no MPI         | Single-thread tensor calc   |
| Parallel   | `defaultq`       | `-N`, `-n`, `mpirun`          | MPI simulation              |
| GPU        | `gpu`            | `--gpus`, `--cpus-per-task`   | Deep learning training      |


## Choosing the Right Node
- **GPU Node**: For massively parallel computations on GPUs (e.g., CUDA, TensorFlow, PyTorch).
- **SMP Node**: For jobs needing large shared memory (big matrices, in-memory data) or multi-threaded code (OpenMP, R, Python multiprocessing).
- **Regular Node**: For MPI-based distributed jobs or simple CPU tasks.
**Decision chart for Choosing Nodes:**
![Decision chart for choosing node types](../fig/Job_Decision_Node_Tree.png)


{% include links.md %}
