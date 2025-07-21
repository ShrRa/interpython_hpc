---
title: "HPC Intro"
teaching: 5
exercises: 0
questions:
- "Question 1"
objectives:
- "Objective 1"
keypoints:
- "Keypoint 1"
---

- Intro into HPC calculations and how they differ from usual ones

## Intro

## Flynn's Taxonomy: a framework for parallel computing
When we talk about parallel computing, it's helpful to have a framework to classify different types of computer architectures. The most common one is Flynn's Taxonomy, which was proposed in 1966 (\cite{https://ieeexplore.ieee.org/document/1447203}). It gives a simple vocabulary for describing how computers handle tasks, and will help us in understanding how certain programming models are better for certain problems.

Flynn’s taxonomy uses four words:
*   **S**ingle
*   **I**nstruction
*   **M**ultiple
*   **D**ata

These are combined to describe four main architectures (\cite{https://onlinelibrary.wiley.com/doi/book/10.1002/9780470932025}). For a thorough overview on these, you can refer to the HIPOWERED book. Let us go over them briefly,

*   **SISD (Single Instruction, Single Data):** This is a traditional serial computer, and is also called a von Neumann computer. It executes one instruction at a time on a single piece of data. Your laptop, when running a simple, non-parallel program, is acting as a SISD machine.
*   **SIMD (Single Instruction, Multiple Data):** This is a parallel architecture where multiple processors all execute the *same instruction* at the same time, but each one works on a *different piece of data*. This is the key to massive data parallelism.
*   **MISD (Multiple Instruction, Single Data):** Each processor uses a different instruction on the same piece of data. This architecture is very rare in practice.
*   **MIMD (Multiple Instruction, Multiple Data):** This is the most common type of parallel computer today. It has multiple processors, and each one can execute different instructions on different data, all at the same time. This is the architecture of a multi-core processor and of entire computing clusters.

In addition to these, a separate way in which parallel computers can be organized are:
1. Multiprocessors: Computers with shared memory.
2. Multicomputers: Computers with distributed memory.

### SIMD in Practice: GPUs

An important example of SIMD architecture in modern computing is the GPU (Graphics Processing Unit).

GPUs were originally designed for computer graphics, which is an inherently parallel task (for e.g., calculating the color of millions of pixels at once). Researchers soon realized this massive parallelism could be used for general-purpose scientific computing, including physics simulations and training AI models, leading to the term GPGPU (General-Purpose GPU). These allow for significant speedups in "data-parallel" models. The trade-off is that GPUs have a different memory hierarchy (with less cache per core compared to CPUs), meaning performance can be limited by algorithms that require frequent or irregular communication between threads.


A CPU consists of a few very powerful cores optimized for complex, sequential tasks. A GPU, in contrast, is made of thousands of simpler cores that are masters of efficiency for data-parallel problems. Because of this, nearly all modern supercomputers are hybrid systems that use both CPUs and GPUs, leveraging the strengths of each.

### Supercomputers vs. Computing Clusters

In the early days of HPC, a "supercomputer" was often a single, monolithic machine with custom vector processors. Today, that has completely changed, the vast majority of systems are clusters. Let us define some terms associated with this,
*   **Cluster:** A cluster is a collection of many individual, standard (SISD) computers (often called nodes) connected by a very fast, high-performance network. A modern supercomputer is a massive cluster. These are classified as multicomputers as they were originally built by connecting multiple SISD computers.
*   **Node:** A node is a single computer within the cluster. It has its own processors (CPUs), memory (RAM), and sometimes its own accelerators (GPUs). A typical compute node in a cluster today has two CPUs with multiple cores each.
*   **Workload Manager (or Scheduler):**  The entire cluster is managed by a special piece of software called a workload manager or scheduler, such as SLURM or PBS. Its job is to manage all the resources, handle a queue of jobs from many users, and decide when and where jobs will run. When submitting a job, it is the scheduler which reserves a set of nodes for the job for a certain amount of time.

### Network Topology for Clusters

Since a cluster is just a collection of nodes, the way these nodes are connected (called the **network topology**) is critical to performance. If any program needs to send data between nodes frequently, a slow or inefficient network will create a major bottleneck.

Common topologies for HPC include:

*   **Mesh:** Nodes are arranged in a two or three-dimensional grid, with each node connected to its nearest neighbors. This structure is illustrated in Figure 1 below, which shows examples of a 2D mesh, a 3D mesh, and a 2D torus (where the edges of the mash wrap around to connect the boundaries, forming a torus). 

    ![Schematic figure of mesh topology](../fig/11_mesh.png)
    <br>
    <sub>Figure 1: 2D and 3D meshes: a) 2D mesh, b) 3D mesh, c) 2D torus.</sub>

*   **Fat Tree:** The fat tree topology, shown in Figure 2, is widely used in large clusters. It is a hierarchical tree structure, but with "fatter" (higher bandwidth) links closer to the root to prevent network congestion when many nodes communicate simultaneously.

    ![Schematic figure of fat tree topology](../fig/11_fattree.png)
    <br>
    <sub>Figure 2: Fat tree topology.</sub>


Other topologies which are less common for an HPC include Bus, Ring, Star, Hypercube, Fully connected, Crossbar and Multistage interconnection. More information can be found in the HiPowered book.


#### BUBBLE
<span style="color: purple;"><em>Sid: Can someone pls fix this and make into a bubble?</em></span>

<div style="background-color:#ffe6e6; border-left: 6px solid #ff0000; padding: 1em; margin: 1em 0;">
<strong style="color:#b30000;">Never Run Computations on the Login Node!</strong><br><br>
When you connect to an HPC cluster, you land on a login node. This node is a shared resource for all users to compile code, manage files, and submit jobs to the workload manager. It is not designed for heavy computation!<br><br>
Running an intensive program on the login node will slow it down for everyone and is a classic mistake for new users. Your job must be submitted through the workload manager (e.g., using <code>sbatch</code> in SLURM) to run on the compute nodes.
</div>

### File system
Scratch, temp, etc

<span style="color: purple;"><em>Sid: Not sure about these</em></span>

## Which computer for which task?
- If you have an algorithm that requires the output from step A to start step B… (sequential code)
- If you have an algorithm that performs the same operation on a large volume of homogeneous data… (parallelizable code)
- If you have an algorithm that operates on vectors or matrices… (vectorization)

<span style="color: purple;"><em>Sid: Not sure about these either...</em></span>


{% include links.md %}