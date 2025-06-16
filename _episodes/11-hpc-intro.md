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

- Historical intro: von Neumann and Harvard architecture. What is CPU, RAM and permanent memory. Typical bottlenecks - data transfer.
- Main characteristics defining the computational power of a computer: CPU, RAM, and hard drive size. 
- Processing astronomical data, building models and running simulations requires significant computational power.
The laptop or PC you're using right now probably has between 8 and 32 Gb of **RAM**, a processor with 4-10 **cores**, and a **hard drive** that can store between 
256 Gb and 1 Tb of data. But what happens if you need to process a dataset that is larger than 1 Tb, or if your model that has to be loaded into the RAM is larger than 32 Gb, or if the simulation you are running will take a month to calculate on your CPU? You need a bigger computer, or you need many computers working in parallel.
  
## SIMD classification system. GPUs
**Should be explain this taxonomy, or it's redundant?**

- Flynn’s taxonomy, uses four words:
-- **S** single.
-- **I** instruction.
-- **M** multiple.
-- **D** data.
- These four letters are used to define four main and distinct types of computer architectures [Gebali, 2011]:
-- SISD single instruction single data. Single processor machines.
-- SIMD single instruction multiple data. Multiple processors. All processors
execute the same instruction on different data.
-- MISD multiple instruction single data. Systolic arrays. Uncommon.
-- MIMD multiple instruction multiple data. Multicore processors and multithreaded multiprocessors. 
Each processor is running its instructions on its local data.
In addition to the previous classifications, parallel computers can be further divided
into two principal groups based on memory organization [Gebali, 2011]:
-- multiprocessors computers with shared memory.
-- multicomputers computers with distributed memory.

### Supercomputers vs Computing Clusters. Terminology: nodes, load balancer

### Network topology for clusters

## Which computer for which task?

- If you have an algorithm that requires the output from step A to start step B... (sequential code)
- If you have an algorithm that performs the same operation on a large volume of homogeneous data... (parallelizable code)
- If you have an algorithm that operates on vectors or matrices... (vectorization)

{% include links.md %}
