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

---
## Types of Jobs and Resources

When you run work on an HPC cluster, your job’s **type** determines how it will be scheduled and what resources it will use. Broadly, jobs fall into three categories:  

- **Serial jobs**  
  These use a single CPU core (or sometimes a single thread) to run all calculations. They don’t require communication between multiple processes. They’re ideal for workloads like simple data analysis, single-threaded simulations, or testing code.  

- **Parallel jobs**  
  These use multiple CPU cores — sometimes across multiple nodes — to run tasks simultaneously. Parallel jobs often use MPI (Message Passing Interface) or OpenMP explained in the previous section to coordinate work. They’re suited for large-scale simulations or computations that can be split into many parts running at once.  

- **GPU jobs**  
  These use Graphics Processing Units to accelerate certain types of workloads, especially those involving heavy numerical computation like deep learning, image processing, or large matrix operations. GPU jobs often also use CPU cores for parts of the workflow.  

Once you know your job type, you can select the correct **SLURM partition** (queue) and request the right resources:  

| Job Type   | SLURM Partition | Key SLURM Options              | Example Use Case            |
|------------|------------------|-------------------------------|-----------------------------|
| Serial     | `serial`         | `--partition`, no MPI         | Single-thread tensor calc   |
| Parallel   | `defaultq`       | `-N`, `-n`, `mpirun`          | MPI simulation              |
| GPU        | `gpu`            | `--gpus`, `--cpus-per-task`   | Deep learning training      |


## Choosing the Right Node
- **Regular Node**: For MPI-based distributed jobs or simple CPU tasks.
- **SMP Node** (*Symmetric Multiprocessing*): For jobs needing large shared memory (big matrices, in-memory data) or    multi-threaded code (OpenMP, R, Python multiprocessing).  
  - In an SMP system, multiple CPUs (cores) share the same physical memory and can access it at the same speed. This architecture is ideal when tasks need frequent access to a common memory space without the communication overhead of distributed systems.
- **GPU Node**: For massively parallel computations on GPUs (e.g., CUDA, TensorFlow, PyTorch).


**Decision chart for Choosing Nodes:**
![Decision chart for choosing node types](../fig/Job_Decision_Node_Tree.png)



{% include links.md %}
