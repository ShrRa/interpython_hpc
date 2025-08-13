---
title: "Slurm"
teaching: 5
exercises: 0
questions:
- "Question 1"
objectives:
- "Objective 1"
keypoints:
- "Keypoint 1"
---

- Slurm as a workload manager (+practical session: how to use Slurm)

## Intro
Slurm is an open source, fault-tolerant, and highly scalable cluster management and job scheduling system for large and small Linux clusters. Slurm requires no kernel modifications for its operation and is relatively self-contained. As a cluster workload manager, Slurm has three key functions. First, it allocates exclusive and/or non-exclusive access to resources (compute nodes) to users for some duration of time so they can perform work. Second, it provides a framework for starting, executing, and monitoring work (normally a parallel job) on the set of allocated nodes. Finally, it arbitrates contention for resources by managing a queue of pending work. 

## MPI
MPI use depends upon the type of MPI being used. There are three fundamentally different modes of operation used by these various MPI implementations.

* Slurm directly launches the tasks and performs initialization of communications through the PMI2 or PMIx APIs. (Supported by most modern MPI implementations.)
* Slurm creates a resource allocation for the job and then mpirun launches tasks using Slurm's infrastructure (older versions of OpenMPI).
* Slurm creates a resource allocation for the job and then mpirun launches tasks using some mechanism other than Slurm, such as SSH or RSH. These tasks are initiated outside of Slurm's monitoring or control. 

## Architecture 
Slurm is a system used to manage and organize work on a cluster — a group of computers working together to perform complex tasks.

At the core of Slurm is a central manager, called slurmctld, which keeps track of available resources (like CPUs and memory) and assigns jobs to the appropriate computers. There can also be a backup manager that takes over if the main one fails.

Each computer in the cluster (called a node) runs a program called slurmd. This acts like a remote assistant: it waits for tasks, runs them, sends back results, and then waits for more.

To keep a record of all activity, an optional component called slurmdbd can be used. It stores accounting information — such as who used what resources and when — in a shared database.

Another optional component, slurmrestd, allows users and applications to communicate with Slurm over the web using a REST API.

Users can interact with Slurm using several simple commands:

* srun: starts a job,

* scancel: cancels a running or queued job,

* sinfo: shows the current status of the system,

* squeue: displays information about jobs currently running or waiting,

* sacct: provides detailed reports on finished jobs.

There’s also a graphical interface called sview that visually shows system and job status, including how the nodes are connected.

Administrators can use tools like:

* scontrol: to monitor or modify how the system is working,

* sacctmgr: to manage users, projects, and resource allocations.

Finally, for developers, Slurm also offers APIs that allow software to interact with the system automatically.

![](../fig/slurm_schema.gif)


## Commands
Man pages exist for all Slurm daemons, commands, and API functions. The command option --help also provides a brief summary of options. Note that the command options are all case sensitive.

To have a look to more general commands
[SLURM Quick Start Summary (PDF)](https://slurm.schedmd.com/pdfs/summary.pdf)


# 🧪 Slurm Basic Command Examples

This document provides basic examples of Slurm commands, showing both the input and expected output.  
---

## 1. View Available Resources

```bash
# Display the status of partitions (queues) and nodes
sinfo
```
### Expected Output:
```bash
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug        up   infinite      2   idle node[01-02]
batch        up   infinite      4   idle node[03-06]
```

## 2. Submit a Simple Command

# Run the 'hostname' command on a compute node
```bash
srun hostname
```
### Expected Output:
```bash
node03
```
## 3. Submit a Job Script
First, create a script file named job.sh:
```bash
#!/bin/bash
#SBATCH --job-name=test_job       # Name of the job
#SBATCH --output=output.txt       # Save output to this file
#SBATCH --time=00:01:00           # Set max execution time (HH:MM:SS)
#SBATCH --ntasks=1                # Number of tasks to run

hostname                          # Command to execute
```
Then submit the job:
```bash
sbatch job.sh
```
### Expected Output:
```bash
Submitted batch job 1234
```
## 4. Check the Queue
# Show running and pending jobs
```bash
squeue
```
### Expected Output:
```bash
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
1234     batch  test_job   alice  R       0:01      1 node03
```
5. Cancel a Job
# Cancel the job with ID 1234

```bash
scancel 1234
```
Expected Output:
```bash
# No output if successful
```
## 6. View Job History
```bash
# Show accounting info about completed jobs
sacct
```
### Expected Output:
```bash
       JobID    JobName  Partition    State  ExitCode
------------ ---------- ---------- -------- ---------
1234         test_job   batch       COMPLETED      0:0

```

# 🔍 Deep Dive: `sbatch` – Submit Jobs to SLURM

The `sbatch` command is used to **submit batch job scripts** to the SLURM job scheduler.

A **batch job** is a script that specifies what commands to run, what resources to request, and other scheduling options. When submitted with `sbatch`, SLURM queues the job and runs it on an available compute node.

---

## 📌 Basic Syntax

```bash
sbatch [options] your_job_script.sh

#!/bin/bash
#SBATCH --job-name=my_job          # Name of the job
#SBATCH --output=out.txt           # File to write standard output
#SBATCH --error=err.txt            # File to write standard error
#SBATCH --time=00:10:00            # Time limit (HH:MM:SS)
#SBATCH --ntasks=1                 # Number of tasks
#SBATCH --mem=1G                   # Memory per node
#SBATCH --partition=short          # Partition to submit to

echo "Running on $(hostname)"
sleep 30
```

| Option                    | Description                                                          |
| ------------------------- | -------------------------------------------------------------------- |
| `--job-name=NAME`         | Sets the name of the job                                             |
| `--output=FILE`           | Redirects stdout to `FILE`                                           |
| `--error=FILE`            | Redirects stderr to `FILE`                                           |
| `--time=HH:MM:SS`         | Sets the time limit for the job                                      |
| `--ntasks=N`              | Number of tasks to run (usually 1 per core unless MPI is used)       |
| `--cpus-per-task=N`       | Number of CPU cores per task                                         |
| `--mem=AMOUNT`            | Memory per node (e.g., `4G`, `2000M`)                                |
| `--partition=NAME`        | Specifies which partition (queue) to submit to                       |
| `--mail-type=ALL`         | Sends emails on job start, end, failure, etc.                        |
| `--mail-user=EMAIL`       | Email address to send notifications to                               |
| `--dependency=afterok:ID` | Runs this job only if job with ID completed successfully (`afterok`) |


## Useful Environment Variables
Inside your script, you can use these special variables:

| Variable               | Description                                    |
| ---------------------- | ---------------------------------------------- |
| `$SLURM_JOB_ID`        | The ID of the job                              |
| `$SLURM_JOB_NAME`      | The name of the job                            |
| `$SLURM_SUBMIT_DIR`    | The directory from which the job was submitted |
| `$SLURM_ARRAY_TASK_ID` | The task ID in an array job                    |
| `$SLURM_NTASKS`        | Total number of tasks requested                |


## Inline Submission (Without Script)
You can submit commands directly without creating a file:

```bash
sbatch --wrap="hostname && sleep 60"
```

## 🔁 4. Array Jobs in SLURM

Sometimes you need to run the **same job multiple times** with slight variations — for example, processing 10 different input files or running a simulation with different parameters.

Instead of writing 10 separate job scripts or submitting the same script 10 times manually, you can use a **job array**.

SLURM will treat each element in the array as a **separate job**, but with shared configuration.  
Each job will have a **unique task ID** accessible with the variable `$SLURM_ARRAY_TASK_ID`.

---

### 🧪 Example: Simple Job Array

Create a job script named `array_job.sh`:

```bash
#!/bin/bash
#SBATCH --job-name=array_test               # Name of the job
#SBATCH --array=1-10                        # Define array range: 10 jobs with IDs from 1 to 10
#SBATCH --output=logs/job_%A_%a.out         # Output file pattern: %A = job ID, %a = array task ID

echo "This is array task $SLURM_ARRAY_TASK_ID"
```

 What this does:

* This script will be launched 10 times (one per task).

* Each task gets a different value of $SLURM_ARRAY_TASK_ID (1 to 10).

* Each task will write its output to a different file, e.g.:

  * logs/job_1234_1.out

  * logs/job_1234_2.out

  * ...

  * logs/job_1234_10.out


# Exercise Task: Run a Python Matrix Inversion Job on SLURM

**Objective:**  
Use SLURM to run a Python program on the cluster that:

- Generates a random square matrix,
- Calculates its inverse,
- Prints both the matrix and its inverse.
---

<details>
<summary>🔽 Click here to show the full exercise instructions</summary>

### 1. Prepare Your Python Script

Create a file named `matrix_inverse.py` with this content:

```python
import numpy as np

N = 3
X = np.random.randn(N, N)
print("X =\n", X)
print("Inverse(X) =\n", np.linalg.inv(X))
```


### 2. Create Your SLURM Batch Script
Create a file named job.slurm:

```bash
#!/bin/bash
#SBATCH --job-name=matrix_inv        # Name for the job
#SBATCH --output=matrix_out.txt      # Where standard output goes
#SBATCH --error=matrix_err.txt       # Where error messages go
#SBATCH --partition=your_partition   # e.g., short, batch, etc.
#SBATCH --nodes=1                    # Use one compute node
#SBATCH --ntasks-per-node=1         # Run one task on that node
#SBATCH --cpus-per-task=1           # Use one CPU core
#SBATCH --time=00:05:00             # Time limit: 5 minutes

module load python                  # Load Python environment or module

python matrix_inverse.py           # Execute the Python script
```

📌 Replace your_partition with a valid queue name on your cluster (e.g., debug, batch) 

### 3. Submit the Job
Run:

```bash
sbatch job.slurm
```
Expected output:
```text
Submitted batch job 5678
```

Here, 5678 is the job ID assigned by Slurm.

### 4. Monitor Progress
To see the job’s status:

```bash
squeue --job 5678
```

### 5. Check Output
Once the job finishes, view the results:

```bash
cat matrix_out.txt
cat matrix_err.txt
```

You should see the random matrix and its inverse printed in matrix_out.txt. matrix_err.txt should be empty unless an error occurred.

### 6. Next Steps (Optional)
* Modify the script to experiment with different matrix sizes, e.g., N = 5.

* Try array jobs: run multiple matrix inversions in parallel with varying N values.

* Add resource options: for example, #SBATCH --mem=2G to request 2 GB of memory.



{% include links.md %}
