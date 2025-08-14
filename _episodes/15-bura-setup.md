---
title: "Bura Setup"
teaching: 30
exercises: 15
questions:
- "How do I find and use software on a shared supercomputer?"
- "Why can't I just use `sudo apt-get install` like on my own machine?"
- "How do I manage different versions of the same software?"
- "How can I install Python packages for my project without affecting other users?"
objectives:
- "Understand the purpose of environment modules."
- "Use `module` commands to find, load, and manage software."
- "Understand the need for Python virtual environments."
- "Create and activate a Python virtual environment."
- "Install project-specific packages using `pip` and a `requirements.txt` file."
keypoints:
- "HPC systems use environment modules to manage shared software."
- "Use `module avail` and `module spider` to find software."
- "Use `module load` to add software to your environment and `module purge` to remove it."
- "Loaded modules are temporary and reset when you log out."
- "Python virtual environments (`venv`) isolate your project's dependencies."
- "Always activate a virtual environment before installing packages with `pip`."
---

After learning the basic commands for navigating the filesystem, it's time to learn how to actually *use* the software on a supercomputer like Bura. On your personal computer, you might use a package manager like `apt`, `yum`, or Homebrew, often with administrator (`sudo`) privileges. On a shared system used by hundreds of people, this isn't possible. Instead, we use a system called **environment modules**.

---

## What are Environment Modules?

Think of the cluster as a massive workshop with every tool imaginable stored in cabinets. To work on your project, you don't bring all the tools to your bench at once. You just get the specific ones you need, like a particular screwdriver or a specific wrench.

Environment modules work the same way. They let you "load" and "unload" specific software packages and versions into your current terminal session, setting up the necessary paths and variables so you can use them.

### Finding Available Software

To see the list of all available software "cabinets," you can use the `module avail` command. This will show you all the modules you can load. The list can be very long!

> ## Compactifying the Process
> The output of `module avail` can be overwhelming. You can pipe the output to `less` to scroll through it (`module avail | less`) or to `grep` to search for something specific (`module avail | grep python`).
{: .callout}

```bash
$ module avail
```

A shorter alias for `module avail` is `ml av`, which you might find more convenient.

```bash
$ ml av
```

This still gives a very long list. A more targeted way to find software is with `module spider`. This command helps you search for a specific package. Let's say we want to find what versions of the Python programming language are available.

```bash
$ module spider python
```
 ~~~
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  python:
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    Versions:
       python/2.7.18
       python/3.8.12
       python/3.9.7
       python/3.10.5
       ...

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  To find other possible module matches, use "module -r spider '.*python.*'"
  To learn more about a specific module, use "module spider mod-name"
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ~~~
 {: .output}

`module spider` shows us all the modules with "python" in their name and the versions available for each.

## Loading and Managing Modules
Now that we've found the software we want, we need to "load" it into our environment. Let's load Python version 3.10.5. The command for this is `module load`.

```bash 
$ module load python/Python-3.10.5
```

How can we be sure the module is loaded? We can use `module list` to see all the currently active modules in our session.

```bash
$ module list
```

~~~
Currently Loaded Modulefiles:
  1) python/Python-3.10.5
~~~
{: .output}

If you want to unload a module, you can use the command `module unload`

```bash
$ module unload python/Python-3.10.5
```

If you want to start fresh and unload all your currently loaded modules, you can use `module purge`.

```bash 
$ module purge
$ module list
```

~~~
No Modulefiles Currently Loaded.
~~~
{: .output}

> ## Note
> When you load a module, it's only for your current login session. If you log out of Bura and log back in later, your modules will be gone. You will need to module load them again for your new session.
{: .callout}

## Python Virtual Environments 

We've loaded a system-wide version of Python. Great! But what if your project needs a specific version of a package like numpy, and another one of your projects needs a different version? If you install them globally, you'll have conflicts.

To solve this, we use virtual environments. A virtual environment is a self-contained directory that holds a specific Python interpreter and all the packages you install for a particular project. It's like giving your project its own private toolbox.

Creating a Virtual Environment
Let's create a virtual environment for our workshop project. First, make sure you have the Python module loaded, as the virtual environment will be based on it.

```bash
$ module load python/Python-3.10.5
```

Now, we can create the environment using Python's built-in venv module. Let's call our environment `hpc_workshop_env`.

```bash
$ python -m venv hpc_workshop_env
```

If you now use `ls`, you will see a new directory has been created.

```bash
$ ls
```

~~~
hpc_workshop_env
~~~
{: .output}

## Activating and Deactivating the Environment 

Just creating the environment isn't enough; you have to activate it. Activating the environment modifies your shell's prompt to let you know it's active and points it to the Python and `pip` executables inside that specific environment.

```bash
$ source hpc_workshop_env/bin/activate
```

You'll know it worked because your command prompt will change to show the environment's name.

```bash
(hpc_workshop_env) $
```

Now, any Python packages you install will go into the `hpc_workshop_env` directory, leaving the system's Python installation clean.

To exit the environment, simply use the `deactivate` command.

```bash
(hpc_workshop_env) $ deactivate
$
```

> ## Exercise: Parallelization Challenge
>
> Use the following steps to practice basic HPC environment and Python virtual environment commands.
>
> ### Challenge
>
> 1. Use `module spider` to find the available versions of `cmake`.
> 2. Create a directory for a new project called `my_test_project`.
> 3. Move into that directory.
> 4. Create and activate a Python virtual environment inside it named `test_env`.
> 5. Deactivate the environment.
>
> ## Solution
>
> ```bash
> $ module spider cmake
> $ mkdir my_test_project
> $ cd my_test_project
> $ python -m venv test_env
> $ source test_env/bin/activate
> (test_env) $ deactivate
> ```
> {: .solution}
{: .challenge}

## Installing Project Packages with `pip`

Most Python projects depend on a set of external libraries. The standard way to manage these is with a `requirements.txt` file and Python's package installer, `pip`.

First, let's create the requirements file. Make sure you are in your project directory (e.g., `hpc_workshop_env` is visible when you type `ls`). Use `nano` to create a file named `requirements.txt`.

```bash 
$ nano requirements.txt
```

Now, copy and paste the following list of packages into the `nano` editor by pressing `Shift` + `Insert`.

```bash
contourpy==1.3.2
cycler==0.12.1
fonttools==4.59.0
kiwisolver==1.4.9
llvmlite==0.44.0
matplotlib==3.10.5
mpi4py==4.1.0
numba==0.61.2
numpy==2.2.6
packaging==25.0
pillow==11.3.0
pyparsing==3.2.3
python-dateutil==2.9.0.post0
six==1.17.0
```
Save the file and exit nano (press Ctrl+X, then Y, then Enter).

Now, to install these packages, first activate your virtual environment. 

```bash 
$ source hpc_workshop_env/bin/activate
```

With the environment active, you can now use `pip` to install everything listed in your requirements file. The `-r `flag tells `pip` to read from a file.

```bash
(hpc_workshop_env) $ pip install -r requirements.txt
```

pip will connect to the internet, download all the specified packages and their dependencies, and install them into your hpc_workshop_env. You are now ready to move on to the next section. This setup ensures that your work is self-contained and reproducible.


{% include links.md %}
