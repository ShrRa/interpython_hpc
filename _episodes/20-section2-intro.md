---
title: "Section 2: Running code on Bura"
colour: "#fafac8"
start: False
teaching: 5
exercises: 0
questions:
- "What are the topics covered in this section?"
objectives:
- "Understand the scope of today's session."
- "Activate the virtual environment and load modules needed for the code examples."
keypoints:
- "We will rely on practical exercises to learn what different modes of program execution look like in real life and which tools we can use for performance analysis."
---

In this section, we will do some hands-on exercises to learn what sequential, parallel, and GPU code looks like 
in real life, and how to adapt your code when you switch to another code execution mode. We are going to look into why `Python` relies on other languages, such as `C` and `Cuda`,
for massive parallelization, which tools can be used for monitoring the performance of your code and how to use Slurm for running your code across various nodes. 

We will use the virtual environment that we created in the previous episodes, so don't forget to run the commands for activating your environment and loading the modules we'll use in the code examples:

```bash 
$ source interpython/bin/activate
$ module load python/Python-3.10.5
$ module load mpi/intel-2021.5
$ module load gcc/gcc-13.2.0
```

{% include links.md %}
