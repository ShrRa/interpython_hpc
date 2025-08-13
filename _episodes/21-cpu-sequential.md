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

Most users begin with simple serial code, which runs sequentially on one processor. However, for problems involving large data sets, high resolution simulations, or time-critical tasks, serial execution quickly becomes inefficient.

Parallel programming allows us to split work across multiple CPUs or even GPUs. High-Performance Computing (HPC) relies on this concept to solve problems faster.

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
