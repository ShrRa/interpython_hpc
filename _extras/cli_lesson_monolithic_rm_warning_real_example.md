---
title: "Command line interface (CLI) "

objectives:
- Learn how to navigate around directories and look at their contents
- Explain the difference between a file and a directory.
- Translate an absolute path into a relative path and vice versa.
- Identify the actual command, flags, and filenames in a command-line call.
- Demonstrate the use of tab completion, and explain its advantages.
keypoints:
- Your current directory is referred to as the working directory.
- To change directories, use `cd`.
- To view files, use `ls`.
- You can view help for a command with `man command` or `command --help`.
- Hit `tab` to autocomplete whatever you're currently typing.
---

The Command-line Interface (CLI) is a text-based interface for interacting directly with the operating system by executing commands. This contrasts with the Graphical User Interface (GUI) you might be familiar with, where instructions are sent to the computer via a point-and-click mechanism. The most common shells are Bash and Zsh, typically used in Unix-based operating systems.

Shell scripting can be very useful in science, including:

- **Reproducibility** – Shell scripts can be saved and re-executed at a later date. Commands executed in the shell are also saved and can be referred to later.
- **Throughput** – Many tasks in science are repetitive. For example, if we were conducting a calculation on 100 samples and wanted to do some simple statistics on reads, we could use loops to perform this task on all sets of reads. This is much quicker than using a GUI.
- **Integration** – Shell scripting allows you to integrate several programs into workflows.
- **Efficiency** – GUIs can be resource-intensive. Using the shell frees resources that would otherwise be used for the GUI.

At this point in the lesson, we've just logged into the system. Nothing has happened yet, and we're not going to be able to do anything until we learn a few basic commands. By the end of this lesson, you will know how to "move around" the system and look at what's there.

When you log in to an HPC system or Linux-based server, you’ll be greeted by a prompt like:

```bash
{{ site.workshop_host_prompt }}
```

The dollar sign (`$`) is called the **prompt**, which shows that the shell is waiting for input. Your prompt may look slightly different depending on the system configuration. Always type only the commands after the prompt — do not include the `$` when copying and pasting.

Let’s start by checking who we are on this system:

```bash
$ whoami
```
{: .language-bash}

```text
yourUsername
```
{: .output}

The shell has now:

1. Found a program named `whoami`
2. Executed it
3. Printed the result
4. Returned to a new prompt, ready for more input

---

## Absolute and Relative Paths

**Absolute path:** begins at the root directory `/` and specifies the full location of a file or directory. It does not depend on the current working directory and always starts with `/`.

Example:
```
/home/angel/lsst2025
```

**Relative path:** begins at the current working directory and specifies the path from there to the target file or directory. It does not start with `/`.

Example: If I am currently in my home directory `/home/angel`:
```
lsst2025/week_1
```
is the relative path to `/home/angel/lsst2025/week_1`.

**When things go wrong:** The most common navigation error occurs when the path to a file is set incorrectly. You’ll see:
```
No such file or directory
```
If this happens, double-check the accuracy of the path.

---

## Three key navigation commands

- `pwd` – Print Working Directory (tells you where you are)
- `cd` – Change Directory
- `ls` – List directory contents

**Exercise 1:**  
Use the `pwd` command to print your current working directory to the terminal. Is the path absolute or relative? How can you tell?

Example:
```bash
$ pwd
/home/yourUsername
```
{: .output}

---

To list the contents of your directory:
```bash
$ ls
```
{: .language-bash}

> ## Differences between remote and local systems
> Try opening a terminal on your local computer (not connected via SSH) and run `ls`. Compare the output to what you see on the HPC system.
>
> > ## Solution
> > On a local machine you might see:
> > ```
> > Desktop  Documents  Downloads  Music  Pictures  Public
> > ```
> > On a remote system the contents differ, and the prompt may also look different.

If `ls` shows nothing, the folder is empty. Let’s create one:
```bash
$ mkdir documents
$ ls
```

Change into it:
```bash
$ cd documents
$ pwd
/home/yourUsername/documents
```

---

## Shortcuts

`~` is your home directory:
```bash
cd ~
```
You can always get back to your home directory using:
```bash
cd
cd ~
cd /home/yourUsername
```

---

## Understanding the UNIX Filesystem

All files and directories exist within a hierarchy starting at `/`, the root directory.

Example:
```bash
$ cd /
$ ls
$ cd ~
```

Typical output:
```
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  tmp  usr  var
```

Most of the time, you’ll be working within:

- `/home/yourUsername` — your home space
- `/scratch/yourUsername` — temporary, high-speed space
- `/work/yourProject` — shared project space

---

## HPC-Specific Filesystem Areas

- **Home**: `/home/yourUsername` – persistent, backed up, limited in size  
- **Scratch**: `/scratch/yourUsername` – temporary, fast storage (purged periodically)  
- **Work**: `/work/yourProject` – shared space, often larger and not backed up  
- **Apps**: `/apps/` – software modules

Always check local documentation for storage policies.

---

## Special Directory Symbols

- `.` — current directory
- `..` — parent directory
- `~` — home directory

Example:
```bash
$ cd ./documents
$ pwd
$ cd ..
$ pwd
```

---

## Flags and Options

Commands can take options (flags). With `ls`:

- `-a`: show hidden files
- `-l`: long listing format
- `-h`: human-readable sizes

Examples:
```bash
$ ls -a
$ ls -l
$ ls -lh
$ ls -l -a ~/documents
```

You can combine options:
```bash
ls -lah
```

---

## Getting Help

Use:
```bash
man ls
```
(Press `q` to quit)  
or:
```bash
ls --help
```

If you use an invalid option:
```
$ ls -j
ls: invalid option -- 'j'
Try 'ls --help' for more information.
```

---

## Writing and Reading Files

Move back to your home directory and create a scratch directory:
```bash
$ cd ~
$ mkdir hpc-test
$ cd hpc-test
```

### Creating and Editing Text Files

Use `nano`:
```bash
$ nano draft.txt
```
- `Ctrl + O` — Save
- `Ctrl + X` — Exit
- `Ctrl + K` — Cut line
- `Ctrl + U` — Paste line

Check the file:
```bash
$ ls
draft.txt
```

---

### Reading Files

```bash
$ cat draft.txt
```

Multiple files:
```bash
$ cat draft.txt draft.txt
```

---

## Creating Directories

```bash
$ mkdir files
$ ls
draft.txt  files
```

---

## Moving, Renaming, Copying

Move:
```bash
$ mv draft.txt files
$ cd files
$ ls
draft.txt
```

Rename:
```bash
$ mv draft.txt notes_v1.txt
```

Copy:
```bash
$ cp notes_v1.txt backup.txt
$ cp notes_v1.txt ..
```

---

## Deleting Files and Directories

Delete a file:
```bash
$ rm notes_v1.txt
```
Delete a directory:
```bash
$ rm -r files
```

 **Caution:** `rm -rf` deletes everything recursively without asking.

---

## Viewing Large Files

- `head file.txt` – top 10 lines
- `tail file.txt` – last 10 lines
- `less file.txt` – scrollable (press `q` to quit)

---

## Downloading and Extracting Files

Download:
```bash
$ wget https://example.com/sample.tar.gz
```
or:
```bash
$ curl -O https://example.com/sample.tar.gz
```

Extract:
```bash
$ tar -xvf sample.tar.gz
```

Cheatsheet:
- `.gz`: `gunzip file.gz`
- `.zip`: `unzip file.zip`
- `.tar.gz` / `.tar.bz2`: `tar -xvf file.tar.gz`

---

## Summary

- Use `pwd`, `ls`, `cd` to navigate
- Understand absolute and relative paths
- Use `nano` to create/edit files
- Use `mkdir`, `mv`, `cp`, `rm` to manage files
- View files with `cat`, `head`, `tail`, `less`
- Download and extract with `wget`, `curl`, `tar`

---

## The Famous Warning About `rm -f`

The `rm` command **permanently deletes files and directories** — there is no "undo" button in the command-line interface.  
One of the most famous cautionary tales in Unix history involves the command:

```bash
rm -rf /
```

This command tells the system to:

- `rm`: remove files
- `-r`: recursively descend into directories
- `-f`: force deletion without prompting

Running `rm -rf /` as a superuser (root) would **attempt to delete the entire filesystem**, including system files, configuration files, and user data. On older Unix systems, this could completely wipe the machine in seconds. Modern Linux systems protect `/` by default, but variations of this command can still cause catastrophic, irreversible data loss.

**Golden Rules:**

1. **Always double-check** the path you're passing to `rm`.
2. Use `ls` first to verify which files you are about to delete.
3. When in doubt, remove interactively with:
   ```bash
   rm -i file.txt
   ```
4. Avoid running `rm -rf` as `root` unless absolutely necessary.

As  Unix users say:  
> "With great power comes great responsibility — and `rm -rf` is the nuclear option."

### Real-World Example: `rm -rf` Disaster

The warnings about `rm -rf` are not just theory — they have caused *real* and costly disasters.

**1. Story **  
A developer intended to delete a temporary directory with:
```bash
rm -rf / home/user/tmp
```
They accidentally typed a space after `/`, turning the command into:
```bash
rm -rf /
```
This began deleting *everything* on the server from the root directory. It resulted in total system loss and required full restoration from backups.

More https://www.linkedin.com/pulse/famous-rm-rf-story-bipin-patwardhan
**2. GitLab 2017**  
There is an urban legend that a junior administrator was connected to the **production server** and ran a cleanup command:
```bash
rm -rf /var/lib/postgresql
```
They believed they were on a staging environment. This command wiped 300 GB of **live production databases** in seconds. GitLab's public postmortem documented the long recovery process from incomplete backups.
https://www.bleepingcomputer.com/news/hardware/gitlab-goes-down-after-employee-deletes-the-wrong-folder/

These examples underscore that `rm -rf` is a **power tool** that can cause irreversible destruction. Always triple-check the path, and consider safer alternatives like:
```bash
rm -ri path/
```
or moving files to a "trash" directory before permanent deletion.
