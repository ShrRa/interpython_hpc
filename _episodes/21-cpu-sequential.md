---
title: "Intro code examples"
start: False
teaching: 30
exercises: 20
questions:
- "What is the difference between serial and parallel code?"
objectives:
- "Understand the structure of CPU and GPU code examples."
keypoints:
- "Serial code is limited to a single thread of execution, while parallel code uses multiple cores or nodes."
---

## Motivation for HPC Coding

Modern computers are fast, however, the volumes of our data and the complexity of our algorithms can easily eat all computational resources and demand more. While most users begin with simple serial code, which runs sequentially on one processor (or rather on a single core), at some point it stops being enough. 
Maybe we want to model the entire Milky Way using the next big data release from our favorite astronomical survey, or execute high-resolution hydrodinamical simulation, or perform time-critical analysis for follow-up observations, and what took minutes or hours now would take months or years. 

So what can we do? There are two main approaches:

- Make the code faster.

-- Use a better algorithm (which is always the preferred way, which, however, may take lots of time and expertise to implement).

-- Restructure your code so the CPU works more efficiently, by using vectorization (doing many operations at once inside a single CPU core in a SIMD manner, utilizing internal CPU architecture optimized for this mode of operation) or parallelization (splitting work across multiple cores or machines).

- Get more computational power.

-- Upgrade to a faster processor (with a higher clock speed).

-- Use hardware with more processors to implement highly parallelized code. It can be a supercomputer with multiple CPU cores, or a computer with GPUs with thousands of smaller cores.

> ## Figure Suggestion: 
> Plot showing execution time of serial vs parallel implementation for increasing problem sizes (e.g., matrix size or loop iterations).
{: .callout}

## Serial Code Example (CPU)

### Introduction to NumPy
Before diving into parallel computing or GPU acceleration, it's important to understand how performance can already be improved significantly on a CPU using efficient libraries. 
- One of the most widely used tools for this in Python is `NumPy`. NumPy provides a fast and memory-efficient way to handle large numerical datasets using multi-dimensional arrays and vectorized operations. 
- While regular Python lists are flexible, they are not optimized for heavy numerical tasks. Looping through data element by element can quickly become a bottleneck as the problem size grows. 
- NumPy solves this problem by providing a powerful N-dimensional array object and tools for performing operations on these arrays efficiently. 
- Under the hood, NumPy uses optimized C code, so operations are much faster than using standard Python loops. 
- NumPy also supports vectorized operations, which means you can apply functions to entire arrays without writing explicit loops. This not only improves performance but also leads to cleaner and more readable code. 
- Using NumPy on the CPU is often the first step toward writing efficient scientific code. 
- It's a strong foundation before we move on to parallel computing or GPU acceleration. Now, we'll see an example of how a simple numerical operation is implemented using NumPy on a single CPU core.

### Example: Summing the elements of a large array using Serial Computation

```python
import numpy as np
import time

array = np.random.rand(10**7)
start = time.time()
total = np.sum(array)
end = time.time()
print(f"Sum: {total}, Time taken: {end - start:.4f} seconds")
```

> ## Exercise: 
> Modify the above to use a manual loop with `for` instead of `np.sum`, and compare the performance.
{: .challenge}

> ## Solution
>
> Replace `np.sum(array)` with a manual loop using `for`.  
> Note: This will be **much slower** due to Python’s loop overhead.
>
> ```python
> import numpy as np
> import time
>
> array = np.random.rand(10**7)
> start = time.time()
> total = 0.0
> for value in array:
>     total += value
> end = time.time()
> print(f"Sum: {total}, Time taken: {end - start:.4f} seconds")
> ```
>
> This gives you a baseline for how optimized `np.sum` is compared to native Python loops.
{: .solution}

> ## Reference: 
> [Carpentries Python loops lesson](https://swcarpentry.github.io/python-novice-inflammation/05-loop.html)
{: .checklist}

---
{% include links.md %}
