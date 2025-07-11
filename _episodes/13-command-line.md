---
title: "Command line basics"
teaching: XX
exercises: YY
questions:
- "How can I change directories from the command line?"
- "How can I create directories and files from the command line?"
- "How can I view my identity?"
- "How can I create and move files?"
- "How can I who is doing what on a computer or HPC?"
- "How can I print to the shell?"
objectives:
- "Learn essential shell commands used in data management and processing on a High Performance Computing Environment"
keypoints:
- "Shell skills enable efficient navigation and manipulation local and remote file systems"
- "The shell can be used to identify who you are and what you have access to"
- "The shell can be used to determine who is doing what is being done on a computing resource?"
---


# Introducing the Shell

The shell or command line is a way to interact with a computer by typing text commands into a terminal or console window. This is in contrast to using a graphical user interface (GUI) with buttons and menus. Although many of the same tasks can be performed with both a shell interface or a GUI interface, the shell gives the most basic and universal access because it does not require any graphics. 
Whether you're navigating a High Performance Computing (HPC) repo, inspecting  files, or debugging processing failures, these shell commands will be indispensable.

# File Navigation
When you view your file system via a graphical interface, you are used to clicking on one folder to look inside and then clicking on another folder inside that one. This folder (or directory) structure is called a directory tree. In the same way that you can click to navigate around your file system, you can type commands into the shell.

## What directory am I in?
The `pwd` command stands for "print working directory". You can always use this command to ask the shell "where am I?" (you will be surprised how often this comes up).
```bash
> pwd
```
```output
> TODO
```

## What is in my directory?
The `ls` command is short for listing - this lists all of the files and directories in the directory that you are currently in. This is really helpful if you are looking for something or can't remember the name of a file or directory. 
```bash
> ls
```
```output
> TODO
```

By default, `ls` does not show you any directories or files starting with `.`. These are called hidden files and directories. If you want to see everything, even the hidden files, you can use the `-a` flag (for all).
```bash
> ls -a
```
```output
> TODO
```
You will notice that you have a `.ssh` directory that you created when you set up your ssh keys.

## Creating a Directory
When you start a project one of the first things you want to do is set up directories to organize it. For example, you may want a top level directory for the project and then sub-directories for data and code. You can create a new directory using the `mkdir` command (for make directory). Let's make a directory for the work we do in this course:
```bash
> mkdir hpc_course
```

> ## Spaces in directory names
> You may have noticed that we separate different parts of a command with spaces. The command line uses spaces to parse each part of the command. For this reason, you should not create directories with spaces in them, because if you then try to do something with them from the command line you need to add special characters to group the multiple words together. It is common to use underscores or dashes between words. 
{: .callout}

## Changing Directories
Creating a directory does not move you into the new directory. To change directories you use the `cd` command. For example:
```bash
> cd hpc_course
```

To move backwards (or up) a directory (for example to move back to your home directory) use `cd ../`

> ## Exercise
> Move into your `hpc_course` directory. Verify that you are in the correct directory, then create two new directories: code and data. Verify that your directories have been created.
{: .challenge}

>## Solution
> ```bash
> cd hpc_course
> pwd
> mkdir code
> mkdir data
> ls
> ```
{: .solution}

> ## Using tab to auto-complete
> It can be tiring to type out the name of every file and every directory and it can also be frustrating when you mistype a word. The shell will auto-complete a filename or directory name if you have typed enough of the word to uniquely define it by pressing the tab button. If there is more than one possibility, press the tab button twice to display the different options.
{: .callout}






#### `ls`
List contents of a directory. Useful flags:
- `-l`: long format
- `-a`: include hidden files
- `-F`: append indicator (e.g. `/` for directory, `@` for symlink)

```bash
$ ls -alF
```
- `-a`: Show all files, including hidden ones (those starting with `.` like `.bashrc`)
- `-C`: Display in columns
- `-F`: Append file type indicators:
  - `/` for directories
  - `@` for symbolic links
  - `*` for executables



### File Manipulation

#### `cp`, `mv`, `rm`
Basic operations:
```bash
$ cp image01.fits image02.fits
$ mv image02.fits image_raw.fits
$ rm image_raw.fits
```

#### `ln`
Create symbolic links to avoid data duplication:
```bash
$ ln -s /datasets/lsst/raw/image01.fits ./image01.fits
```

### Viewing and Extracting Data

#### `cat`, `less`, `grep`
View and search YAML config or log files:
```bash
$ cat config.yaml
$ less job.log
$ grep "FATAL" job.log
```

### Permissions and Metadata

#### `chmod`, `chown`, `stat`
Manage and inspect file attributes:
```bash
$ chmod 644 config.yaml
$ stat image01.fits
```
### LSST-Specific Use Cases


> Familiarity with `bash`, `grep`, `find`, and `awk` will accelerate your workflow.

---

## Exercises

### Exercise 1: Set up LSST-style directory

1. Create a folder structure:
```
lsst_cli/
├── visit001/
│   ├── raw/
│   ├── calexp/
│   └── logs/
├── visit002/
│   ├── raw/
│   ├── calexp/
│   └── logs/
```

2. Populate each `raw/` with `image01.fits`, and symbolic link to `calexp.fits` in `calexp/`.

3. Add a `process.yaml` and log file in each `logs/`.

Use `tree` to verify.

### Exercise 2: Analyze Logs

Using `grep` and `less`, identify all lines with "WARNING" or "FATAL" in the log files across visits.

---

## Further Learning

Explore additional CLI tools:
- `awk`, `cut`, `xargs`
- `eups`, `conda` for environment setup

### ls

List all the files in a directory. Linux as many Operating Systems organize
files in files and directories (also called folders).

~~~
$ ls
file0a  file0b  folder1  folder2 link0a  link2a
~~~
{: .source}

Some terminal offer color output so you can differentiate normal files from folders. You can make the difference more clear with this

~~~
$ ls -aCF
./  ../  file0a  file0b  folder1/  folder2/ link0a@  link2a@
~~~
{: .source}

You will see a two extra directories `"."` and `".."`. Those are special folders that refer to the current folder and the folder up in the tree.
Directories have the suffix `"/"`. Symbolic links, kind of shortcuts to other files or directories are indicated with the symbol `"@"`.

Another option to get more information about the files in the system is:

~~~
$ ls -al
total 16
drwxr-xr-x    5 andjelka  staff   160 Jun 16 08:53 .
drwxr-xr-x+ 273 andjelka  staff  8736 Jun 16 08:52 ..
-rw-r--r--    1 andjelka  staff    19 Jun 16 08:53 config.yaml
-rw-r--r--    1 andjelka  staff     0 Jun 16 08:53 image01.fits
-rw-r--r--    1 andjelka  staff    37 Jun 16 08:53 job.log
~~~
{: .source}

Those characters on the first column indicate the permissions. The first character will be "d" for directories, "l" for symbolic links and "-" for normal files. The next 3 characters are the permissions for "read", "write" and "execute" for the owner. The next 3 are for the group, and the final 3 are for others.
The meaning of "execute" for a file indicates that the file could be a script or binary executable. For a directory it means that you can see its contents.

### cp

This command copies the contents of one file into another file. For example

~~~
$ cp file0b file0c
~~~
{: .source}

### rm

This command deletes the contents of one file. For example

~~~
$ rm file0c
~~~
{: .source}
There is no such thing like a trash folder on a HPC system. Deleting a file should be consider an irreversible operation.

Recursive deletes can be done with

~~~
$ rm -rf folder_to_delete
~~~
{: .source}
Be extremely cautious deleting files recursively. You cannot damage the system as the files that you do not own you cannot delete. However, you can delete all your files forever.

### mv

This command moves a files from one directory to another. It also can be used to rename files or directories.

~~~
$ mv file0b file0c
~~~
{: .source}

### pwd

It is easy to get lost when you move in complex directory structures. pwd will tell you the current directory.

~~~
$ pwd
/Users/andjelka/Documents/LSST/interpython/interpython_hpc
~~~
{: .source}

### cd

This command moves you to the directory indicated as an argument, if no argument is given, it returns to your home directory.

~~~
$ cd folder1
~~~
{: .source}

### cat and tac

When you want to see the contents of a text file, the command cat displays the contents on the screen. It is also useful when you want to concatenate the contents of several files.

~~~
$ cat star_A_lc.csv
time,brightness
0.0,90.5
0.5,91.1
1.0,88.9
1.5,92.2
2.0,89.3
2.5,90.8
3.0,87.7...
~~~
{: .source}

To concatenate files you need to use the symbol `">"` indicating that you want to redirect the output of a command into a file

~~~
$ cat file1 file2 file3 > file_all
~~~
{: .source}
The command tac shows the files in reverse starting from the last line back to the first one.

### more and less

Sometimes text files, as those created as product of simulations are too large to be seen in one screen, the command "more" shows the files one screen at a time. The command `"less"` offers more functionality and should be the tool of choice to see large text files.

~~~
$ less OUT
~~~
{: .source}

### ln

This command allow to create links between files. Used wisely could help you save time when traveling frequently to deep directories. By default it creates hard links. Hard links are like copies, but they make references to the same place in disk. Symbolic links are better in many cases because you can cross file systems and partitions. To create a symbolic link

~~~
$ ln -s file1 link_to_file1
~~~
{: .source}

### grep

The grep command extract from its input the lines containing a specified string or regular expression. It is a powerful command for extracting specific information from large files. Consider for example

~~~
$ grep time  star_A_lc.csv
time,brightness
$ grep 88.9  star_A_lc.csv
1.0,88.9
  ...
~~~
{: .source}




Create a light curve directory with empty csv files created with touch command/use provided csv files:
```bash
mkdir -p lightcurves
cd lightcurves
touch star_A_lc.csv star_B_lc.csv star_C_lc.csv
ln -s star_A_lc.csv brightest_star.csv
```

### `ls` – List Light Curve Files

List files:
```bash
$ ls
star_A_lc.csv  star_B_lc.csv  star_C_lc.csv  brightest_star.csv
```

Use `-F` and `-a` for extra detail:
```bash
$ ls -aF
./  ../  star_A_lc.csv  star_B_lc.csv  star_C_lc.csv  brightest_star.csv@
```

Long format with metadata:
```bash
$ ls -al
-rw-r--r--  1 user  staff  1024 Jun 16 09:00 star_A_lc.csv
lrwxr-xr-x  1 user  staff    15 Jun 16 09:01 brightest_star.csv -> star_A_lc.csv
```

---

### `cp` – Copy a Light Curve File

```bash
$ cp star_B_lc.csv backup_star_B.csv
```

---

### `rm` – Delete a Corrupted Light Curve

```bash
$ rm star_C_lc.csv
```

---

### `mv` – Rename Light Curve

```bash
$ mv star_B_lc.csv star_B_epoch1.csv
```

---

### `pwd` – Show Working Directory

```bash
$ pwd
/home/user/...../lightcurves
```

---

### `cd` – Move between directroies

```bash
$ cd ../images
```

---

### `cat` and `tac` – Inspect or Reverse Light Curve

```bash
cat star_A_lc.csv
tac star_A_lc.csv
```

Combine curves:
```bash
cat star_A_lc.csv star_B_epoch1.csv > merged_lc.csv
```

---

### `more` and `less` – View Long Curves

```bash
$ less star_A_lc.csv
```

---

### `ln` – Create Alias for Light Curve

```bash
ln -s star_B_epoch1.csv variable_star.csv
```

---

### `grep` – Extract Brightness Above Threshold

```bash
grep ',[89][0-9]\.[0-9]*' star_A_lc.csv
```





Regular expressions offers ways to specified text strings that could vary in several ways and allow commands such as grep to extract those strings efficiently. We will see more about regular expressions on our third day devoted to data processing.

### More commands

The 10 commands above, will give you enough tools to move files around and
travel the directory tree.
The GNU Core Utilities are the basic file, shell and text manipulation
utilities of the GNU operating system.
These are the core utilities which are expected to exist on every
operating system.

If you want to know about the whole set of coreutils execute:

~~~
info coreutils
~~~
{: .source}

Each command has its own manual. You can access those manuals with

~~~
man <COMMAND>
~~~
{: .source}

> ## Output of entire files
>
> ~~~
> cat                    Concatenate and write files
> tac                    Concatenate and write files in reverse
> nl                     Number lines and write files
> od                     Write files in octal or other formats
> base64                 Transform data into printable data
> ~~~
> {: .source}
{: .callout}

> ### Formatting file contents
>
> ~~~
> fmt                    Reformat paragraph text
> numfmt                 Reformat numbers
> pr                     Paginate or columnate files for printing
> fold                   Wrap input lines to fit in specified width
> ~~~
> {: .source}
{: .callout}

> ## Output of parts of files
>
> ~~~
> head                   Output the first part of files
> tail                   Output the last part of files
> split                  Split a file into fixed-size pieces
> csplit                 Split a file into context-determined pieces
> ~~~
> {: .source}
{: .callout}

> ## Summarizing files
>
> ~~~
> wc                     Print newline, word, and byte counts
> sum                    Print checksum and block counts
> cksum                  Print CRC checksum and byte counts
> md5sum                 Print or check MD5 digests
> sha1sum                Print or check SHA-1 digests
> sha2 utilities                   Print or check SHA-2 digests
> ~~~
> {: .source}
{: .callout}

> ## Operating on sorted files
>
> ~~~
> sort                   Sort text files
> shuf                   Shuffle text files
> uniq                   Uniquify files
> comm                   Compare two sorted files line by line
> ptx                    Produce a permuted index of file contents
> tsort                  Topological sort
> ~~~
> {: .source}
{: .callout}

> ## Operating on fields
>
> ~~~
> cut                    Print selected parts of lines
> paste                  Merge lines of files
> join                   Join lines on a common field
> ~~~
> {: .source}
{: .callout}

> ## Operating on characters
>
> ~~~
> tr                     Translate, squeeze, and/or delete characters
> expand                 Convert tabs to spaces
> unexpand               Convert spaces to tabs
> ~~~
> {: .source}
{: .callout}

> ### Directory listing
>
> ~~~
> ls                     List directory contents
> dir                    Briefly list directory contents
> vdir                   Verbosely list directory contents
> dircolors              Color setup for 'ls'
> ~~~
> {: .source}
{: .callout}

> ## Basic operations
>
> ~~~
> cp                     Copy files and directories
> dd                     Convert and copy a file
> install                Copy files and set attributes
> mv                     Move (rename) files
> rm                     Remove files or directories
> shred                  Remove files more securely
> ~~~
> {: .source}
{: .callout}

> ## Special file types
>
> ~~~
> link                   Make a hard link via the link syscall
> ln                     Make links between files
> mkdir                  Make directories
> mkfifo                 Make FIFOs (named pipes)
> mknod                  Make block or character special files
> readlink               Print value of a symlink or canonical file name
> rmdir                  Remove empty directories
> unlink                 Remove files via unlink syscall
> ~~~
> {: .source}
{: .callout}

> ## Changing file attributes
>
> ~~~
> chown                  Change file owner and group
> chgrp                  Change group ownership
> chmod                  Change access permissions
> touch                  Change file timestamps
> ~~~
> {: .source}
{: .callout}

> ## Disk usage
>
> ~~~
> df                     Report file system disk space usage
> du                     Estimate file space usage
> stat                   Report file or file system status
> sync                   Synchronize data on disk with memory
> truncate               Shrink or extend the size of a file
> ~~~
> {: .source}
{: .callout}

> ## Printing text
>
> ~~~
> echo                   Print a line of text
> printf                 Format and print data
> yes                    Print a string until interrupted
> ~~~
> {: .source}
{: .callout}

> ## Conditions
>
> ~~~
> false                  Do nothing, unsuccessfully
> true                   Do nothing, successfully
> test                   Check file types and compare values
> expr                   Evaluate expressions
> tee                    Redirect output to multiple files or processes
> ~~~
> {: .source}
{: .callout}

> ## File name manipulation
>
> ~~~
> basename               Strip directory and suffix from a file name
> dirname                Strip last file name component
> pathchk                Check file name validity and portability
> mktemp                 Create temporary file or directory
> realpath               Print resolved file names
> ~~~
> {: .source}
{: .callout}

> ## Working context
>
> ~~~
> pwd                    Print working directory
> stty                   Print or change terminal characteristics
> printenv               Print all or some environment variables
> tty                    Print file name of terminal on standard input
> ~~~
> {: .source}
{: .callout}

> ## User information
>
> ~~~
> id                     Print user identity
> logname                Print current login name
> whoami                 Print effective user ID
> groups                 Print group names a user is in
> users                  Print login names of users currently logged in
> who                    Print who is currently logged in
> ~~~
> {: .source}
{: .callout}

> ## System context
>
> ~~~
> arch                   Print machine hardware name
> date                   Print or set system date and time
> nproc                  Print the number of processors
> uname                  Print system information
> hostname               Print or set system name
> hostid                 Print numeric host identifier
> uptime                 Print system uptime and load
> ~~~
> {: .source}
{: .callout}

> ## Modified command
>
> ~~~
> chroot                 Run a command with a different root directory
> env                    Run a command in a modified environment
> nice                   Run a command with modified niceness
> nohup                  Run a command immune to hangups
> stdbuf                 Run a command with modified I/O buffering
> timeout                Run a command with a time limit
> ~~~
> {: .source}
{: .callout}

> ## Process control
>
> ~~~
> kill                   Sending a signal to processes
> ~~~
> {: .source}
{: .callout}

> ### Delaying
>
> ~~~
> sleep                  Delay for a specified time
> ~~~
> {: .source}
{: .callout}

> ## Numeric operations
>
> ~~~
> factor                 Print prime factors
> seq                    Print numeric sequences
> ~~~
> {: .source}
{: .callout}

> ## Exercise: Using the Command Line Interface
>
> 1. Create 4 folders `A`, `B`, `C`, `D` and inside each of them create a three more: `X`, `Y` and `Z`. At the end you should have 12 subfolders. Use the command tree to ensure you create the correct tree.
>
>> ## Solution
>>  You should get:
>>
>> ~~~
>> $ tree
>>.
>>├── A
>>│   ├── X
>>│   ├── Y
>>│   └── Z
>>├── B
>>│   ├── X
>>│   ├── Y
>>│   └── Z
>>├── C
>>│   ├── X
>>│   ├── Y
>>│   └── Z
>>└── D
>>    ├── X
>>    ├── Y
>>    └── Z
>> ~~~
>> {: .source}
> {: .solution}
>
> 2. Lets copy some files in those folders. From the data folder lightcurve and two csv files
>`1.IntroHPC/1.CLI`, there are 3 files `t17.in`, `t17.files` and `14si.pspnc`.
>Using the command line tools create copies of "t17.in" and "t17.files" inside each of those folders and symbolic link for `14si.pspnc`. Both "t17.in" and "t17.files" are text files that we want to edit, but `14si.pspnc` is just a relatively big file that we just need to use for the simulation, we do not want to make copies of if, just symbolic links and save disk space.
>{: .source}
{: .challenge}


> ## Solution
>
> ### Step-by-step CLI commands:
>
> ```bash
> # Step 1: Create the main folders
> mkdir -p A/X A/Y A/Z B/X B/Y B/Z C/X C/Y C/Z D/X D/Y D/Z
>
> # Step 2: Confirm structure
> tree
> ```
>
> Output should be:
>
> ```
> .
> ├── A
> │   ├── X
> │   ├── Y
> │   └── Z
> ├── B
> │   ├── X
> │   ├── Y
> │   └── Z
> ├── C
> │   ├── X
> │   ├── Y
> │   └── Z
> └── D
>     ├── X
>     ├── Y
>     └── Z
> ```
>
> ### File Preparation:
>
> ```bash
> # Make a dummy data directory and populate it
> mkdir -p 1.IntroHPC/1.CLI
> echo "dummy input" > 1.IntroHPC/1.CLI/test.in
> echo "file list" > 1.IntroHPC/1.CLI/test.files
> touch 1.IntroHPC/1.CLI/14si.pspnc
> ```
>
> ### Copy and link files
>
> ```bash
> for folder in A B C D; do
>   for sub in X Y Z; do
>     cp 1.IntroHPC/1.CLI/test.in $folder/$sub/
>     cp 1.IntroHPC/1.CLI/test.files $folder/$sub/
>     ln -s ../../../1.IntroHPC/1.CLI/14si.pspnc $folder/$sub/14si.pspnc
>   done
> done
> ```
>
> ### Verify
>
> ```bash
> tree A
> cat A/X/t17.in
> ls -l A/X/14si.pspnc
> ```


## Midnight Commander

GNU Midnight Commander is a visual file manager. mc feature a rich full-screen text mode application that allows you to copy, move and delete files and whole directory trees. Sometimes using a text-based user interface is convenient, in order to use mc just enter the command on the terminal

~~~
mc
~~~
{: .source}

There are several keystrokes that can be used to work with mc, most of them comes from typing the F1 to F10 keys. On Mac you need to press the "fn" key, on gnome (Linux), you need to disable the interpretation of the Function keys for gnome-terminal.


> ## Exercise: Using the Command Line Interface
>
> Use mc to create a folder E and subfolders X, Y and Z, copy the same files as we did for the previous exercise.
>
>{: .source}
{: .challenge}

{% include links.md %}

### Exercise: Create LSST-style Visit Directory Structure

Use the CLI to create the following:

```
lsst_cli/
├── visit001/
│   ├── raw/
│   ├── calexp/
│   └── logs/
├── visit002/
│   ├── raw/
│   ├── calexp/
│   └── logs/
```

Then:

- Add dummy files `image01.fits` into each `raw/` folder.
- Create symbolic links from `calexp/calexp.fits` to `../raw/image01.fits`.
- Create YAML files in each `logs/` folder with config info and dummy `job.log` files with `WARNING` and `FATAL` strings.

### Exercise: Analyze Simulated Pipeline Logs

Use `grep` to find all lines in all `job.log` files containing "FATAL" or "WARNING".

```bash
$ grep -rE 'FATAL|WARNING' lsst_cli/
```


## Midnight Commander

GNU Midnight Commander is a visual file manager. mc feature a rich full-screen text mode application that allows you to copy, move and delete files and whole directory trees. Sometimes using a text-based user interface is convenient, in order to use mc just enter the command on the terminal

~~~
mc
~~~
{: .source}

There are several keystrokes that can be used to work with mc, most of them comes from typing the F1 to F10 keys. On Mac you need to press the "fn" key, on gnome (Linux), you need to disable the interpretation of the Function keys for gnome-terminal.


> ## Exercise: Using the Command Line Interface
>
> Use mc to create a folder E and subfolders X, Y and Z, copy the same files as we did for the previous exercise.
>
>{: .source}
{: .challenge}

{% include links.md %}