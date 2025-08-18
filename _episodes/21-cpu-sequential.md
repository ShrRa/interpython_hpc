---
title: "Intro code examples"
start: False
teaching: 15
exercises: 0
questions:
- "What are our options to speed up our code?"
objectives:
- "Understand the main strategies of optimizing code."
keypoints:
- "Serial code is limited to a single thread of execution, while parallel code uses multiple cores or nodes."
---

## Motivation for HPC Coding

Modern computers are fast, however, the volumes of our data and the complexity of our algorithms can easily eat all computational resources and demand more. While most users begin with simple serial code, which runs sequentially on one processor (or rather on a single core), at some point it stops being enough. 
Maybe we want to model the entire Milky Way using the next big data release from our favorite astronomical survey, or execute high-resolution hydrodynamical simulation, or perform time-critical analysis for follow-up observations, and what took minutes or hours now would take months or years. 

So what can we do? There are two main approaches:

- Approach 1: Make the code faster.

  -- Use a better algorithm (which is always the preferred way, which, however, may take lots of time and expertise to implement).

  -- Restructure your code so the CPU works more efficiently, by using vectorization (doing many operations at once inside a single CPU core in a SIMD manner, utilizing internal CPU architecture optimized for this mode of operation) or parallelization (splitting work across multiple cores or machines).

- Approach 2: Get more computational power.

  -- Upgrade to a faster processor (with a higher clock speed).

  -- Use hardware with more processors to implement highly parallelized code. It can be a supercomputer with multiple CPU cores, or a computer with GPUs with thousands of smaller cores.

Before we move on to large-scale supercomputing, let’s first look at a much smaller but very common situation - how a simple piece of code can be written in different ways, and how that affects performance. (Approach 1)

Even if we’re only summing the numbers in a big array, the way we write the code can make a big difference.  
A naïve approach (using a `for` loop) processes one element at a time, while more efficient approaches can take advantage of the CPU’s ability to perform many operations at once.

This idea — *doing more work in the same amount of time by restructuring code* — is the foundation of high-performance computing.  

We’ll start with this simple example to see how writing smarter code (vectorization) can already give us a big speed-up, even before we try parallelization or supercomputers.


## Serial vs. Vectorized Code

Let’s look at a simple example: summing the elements of a large array. As mentioned above an obvious way to implement this is by using a `for` loop. With this implementation, each iteration runs only after the previous one has finished.

~~~
# File Name - serial_code.py
# This script demonstrates summing a large NumPy array using a Python loop.
# It highlights the performance cost of looping in Python compared to vectorized operations.

# Import NumPy for numerical array operations and time for measuring execution time
import numpy as np   
import time          

# Create a NumPy array of 10 million random values between 0 and 1
array = np.random.rand(10**7)

# Record the start time
start = time.time()

# Initialize the total sum
total = 0.0

# Loop over each element in the array and add it to the total
for value in array:
     total += value

# Record the end time
end = time.time()

# Print the final sum and the time taken
print(f"Sum: {total}, Time taken: {end - start:.4f} seconds")
~~~
{: .language-python}

 ~~~
 Sum: 4999849.298696889, Time taken: 1.2308 seconds
 ~~~
 {: .output}

Depending on your processor, this code may take up to a couple of seconds to execute. 
Summation is one of those operations that can be vectorized. In Python, if we loop over an array element by element, the interpreter has to execute each iteration in Python’s high-level code, which adds overhead and slows things down. In contrast, C is a low-level, compiled language, meaning its instructions are executed directly by the CPU without the overhead of Python’s interpreter. Vectorization in this context means that instead of performing the sum one element at a time in Python, we hand the entire array to a function like `numpy.sum`, which is implemented in optimized C code. This function can take advantage of CPU-level optimizations, such as processing multiple elements at once (using SIMD instructions) and reducing loop overhead. In essence, vectorization allows Python to delegate heavy computations to fast, compiled code, giving a huge speed-up while letting us write clean, high-level Python code.

~~~
# File Name - vector_numpy.py
# This script demonstrates summing a large NumPy array using NumPy's built-in
# vectorized function np.sum, which is much faster than a manual Python loop.

# Import NumPy for numerical array operations and time for measuring execution time
import numpy as np   
import time          

# Create a NumPy array of 10 million random values between 0 and 1
array = np.random.rand(10**7)

# Record the start time
start = time.time()

# Compute the sum using NumPy's optimized vectorized function
total = np.sum(array)

# Record the end time
end = time.time()

# Print the final sum and the time taken
print(f"Sum: {total}, Time taken: {end - start:.4f} seconds")

~~~
{: .language-python}

 ~~~
Sum: 4999849.29869658, Time taken: 0.0048 seconds
 ~~~
 {: .output}

Run this and compare the times. You should see a big difference — vectorization lets you do the same work in far fewer CPU instructions, without paying Python’s loop-by-loop penalty. For such a small task, the loop overhead is actually a big deal.

> ## Reference: 
> [Carpentries Python loops lesson](https://swcarpentry.github.io/python-novice-inflammation/05-loop.html)
{: .checklist}

---
{% include links.md %}
