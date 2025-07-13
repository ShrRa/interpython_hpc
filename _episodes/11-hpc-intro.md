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

Simple, inexpensive computing tasks are typically performed **sequentially**, *i.e.*, where instructions are completed one after another in the order that they appear in the code, which is the default paradigm in most programming languages. For larger tasks that require many tasks to be executed, it is often more efficient to take advantage of the intrinisically parallel nature of most processors, which are designed to execute multiple processes simultaneously. Many common programming languages, including Python, support software that is executed in **parallel**, where multiple CPU cores are employed to perform tasks independently.

In modern computing, parallel programming has become more and more essential as computational tasks become more demanding. From protein folding in experimental drug development to galaxy formation and evolution, complex simulations rely on parallel computing to solve some of the most difficult problems in science. Parallel programming, hardware architecture, and systems admininstration come together in the multidisciplinary field of **high-performance computing** (HPC). In constrast to running code locally on your home machine, high-performance computing involves connecting to a cluster of computers elsewhere in the world that are networked together in order to run many operations in parallel.  

## Intro

### Computer Architectures

Historically, computer architectures can be divided into two categories -- von Neumann and Harvard. In the former, a computer system contains the following components:

- Arithmetic/logic unit (ALU)
- Control unit (CU)
- Memory unit (MU)
- Input/output (I/O) devices

The ALU takes in data from local memory from the MU and performs calculations, and the CU interprets instructions and directs the flow of data to and from the I/O devices, as shown in the diagram below. The MU contains all of the memory and instructions, which creates a performance bottleneck related to data transfer.

    ![von Neumann diagram](../fig/vonneumann.png)
    <br>
    <sub>Diagram of von Neumann architecture, from (\cite{https://onlinelibrary.wiley.com/doi/book/10.1002/9780470932025})</sub>

The Harvard architecture is a variant of the von Neumann design, where instruction and data storage are physically separated, which allows simulataneous access to instructions and memory. This partially overcomes the von Neumann bottleneck, and most modern central processing units (CPU) adopt this architecture.

    ![Harvard diagram](../fig/harvard.png)
    <br>
    <sub>Diagram of Harvard architecture, from (\cite{https://onlinelibrary.wiley.com/doi/book/10.1002/9780470932025})</sub>

### Performance

Computational peformance is largely determed by three components:

- **CPU:** CPU performance is quantified by frequency, or "clock speed." This is determines how quickly a CPU executes the instructions passed to it in terms of CPU cycles per second. For example a CPU with a clock speed of 3.5 GHz peforms 3.5 billion cycles each second. Some CPUs have multiple **cores** that support parallelization by executing multiple instructions simultaneously (\cite{https://www.intel.com/content/www/us/en/gaming/resources/cpu-clock-speed.html})

- **RAM:** Random access memory (RAM) is a computer's short-term memory and stores the data a computer needs to run applications and open files. Faster RAM allows data to flow to and from your CPU more rapidly, and more RAM capacity helps the CPU complete complex operations simultaneously (\cite{https://www.intel.com/content/www/us/en/tech-tips-and-tricks/computer-ram.html}). 

- **Hard drive:** In contrast to RAM, a computer's hard drive is for long term data storage. Hard drives are characterized by their capacity and performance. Higher-capacity drives can hold more data and higher-performance drives read and write data faster. Hard disk drives (HDD) tend to offer more capacity at a lower cost, while solid state drives (SSDs) offer better performance and reliability. 

Processing astronomical data, building models and running simulations requires significant computational power.
The laptop or PC you're using right now probably has between 8 and 32 Gb of **RAM**, a processor with 4-10 **cores**, and a **hard drive** that can store between 256 Gb and 1 Tb of data. But what happens if you need to process a dataset that is larger than 1 Tb, or if your model that has to be loaded into the RAM is larger than 32 Gb, or if the simulation you are running will take a month to calculate on your CPU? You need a bigger computer, or you need many computers working in parallel.
  
## SIMD classification system. GPUs
**Should be explain this taxonomy, or it's redundant?**

- Flynn’s taxonomy, uses four words:
    * **S** single.
    * **I** instruction.
    * **M** multiple.
    * **D** data.
- These four letters are used to define four main and distinct types of computer architectures [Gebali, 2011]:
    * SISD single instruction single data. Single processor machines.
    * SIMD single instruction multiple data. Multiple processors. All processors
execute the same instruction on different data.
    * MISD multiple instruction single data. Systolic arrays. Uncommon.
    * MIMD multiple instruction multiple data. Multicore processors and multithreaded multiprocessors. 
Each processor is running its instructions on its local data.
In addition to the previous classifications, parallel computers can be further divided
into two principal groups based on memory organization [Gebali, 2011]:
- multiprocessors computers with shared memory.
- multicomputers computers with distributed memory.

### Supercomputers vs Computing Clusters. Terminology: nodes, load balancer

### Network topology for clusters
Don't run computations on login node)

### File system

HPC clusters use a few different locations and formats for storage. 

- **Home directories:** HPC clusters allocate personal storage to individual users, though typically with limited capacity. This is a good place to store scripts and configuration files.

- **Scratch:** Scratch space is temporary storage that offers signifcantly larger capacity for active jobs and processing that is not backed up and usually deleted after job completion. Using scratch space is appropriate for: 

    - Jobs that require large storage capacity while running
    - Data sets that do not fit in personal storage but are not permanently needed
    - Jobs that need higher-performance storage than provided by personal storage

- **Shared:** Shared storage is accessible to multiple users. These spaces tend to be allocated to members of a research groups as a common working directory and are continuously backed up. 

\cite{https://www.hpc.iastate.edu/guides/nova/storage#:~:text=Home%20directories%20(/home),used%20for%20high%20volume%20access.})
\cite{https://services.dartmouth.edu/TDClient/1806/Portal/KB/ArticleDet?ID=140938}

## Which computer for which task?

- If you have an algorithm that requires the output from step A to start step B... (sequential code)
- If you have an algorithm that performs the same operation on a large volume of homogeneous data... (parallelizable code)
- If you have an algorithm that operates on vectors or matrices... (vectorization)

{% include links.md %}
