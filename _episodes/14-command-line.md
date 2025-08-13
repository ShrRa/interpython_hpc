---
title: "Command line basics"
teaching: 45
exercises: 15
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
- "The shell can be used to determine what is happening on a system and how you are using the system"
---


## Introducing the Shell

The shell or command line is a way to interact with a computer by typing text commands into a terminal or console window. This is in contrast to using a graphical user interface (GUI) with buttons and menus. Although many of the same tasks can be performed with both a shell interface or a GUI interface, the shell gives the most basic and universal access because it does not require any graphics. 
Whether you're navigating a High Performance Computing (HPC) repo, inspecting  files, or debugging processing failures, these shell commands will be indispensable.

You have already opened a shell to ssh into bura. Now that your shell is pointing to the bura file system, we will learn how to navigate it, manipulate files, and interrogate the machine for information about you, the file system, and the tasks it is running.

## File Navigation
When you view your file system via a graphical interface, you are used to clicking on one folder to look inside and then clicking on another folder inside that one. This folder (or directory) structure is called a directory tree. In the same way that you can click to navigate around your file system, you can type commands into the shell.

### What directory am I in?
The `pwd` command stands for "print working directory". You can always use this command to ask the shell "where am I?" (you will be surprised how often this comes up).
```bash
$ pwd
```
```output
> edu02
```

### What is in my directory?
The `ls` command is short for listing - this lists all of the files and directories in the directory that you are currently in. This is really helpful if you are looking for something or can't remember the name of a file or directory. 
```bash
$ ls
```
```output
> ekran.txt     mc.slurm   program.c    sc
> JobArr.slurm  mpi.slurm  program.exe  sc.slurm
```

By default, `ls` does not show you any directories or files starting with `.`. These are called hidden files and directories. If you want to see everything, even the hidden files, you can use the `-a` flag (for all).
```bash
$ ls -a
```
```output
> .   .bash_history  JobArr.slurm  mpi.slurm  program.exe  sc.slurm
> ..  ekran.txt      mc.slurm      program.c  sc
```

Another useful option is `-F` flag - this adds symbols to the output to identify different types of entries. For example it will put a `/` after directories. 

> ## Using Multiple Flags
> Sometimes you want to use more than one flag for a command (for example maybe you want to use the `-a` and `-F` flags) to show all hidden files and tell you which ones are directories. If the flag is a single letter then you can string them together like `ls -aF` or if you prefer you can write `ls -a -F`. The order you put the flags in doesn't matter.
{: .callout}

### Creating a Directory
When you start a project one of the first things you want to do is set up directories to organize it. For example, you may want a top level directory for the project and then sub-directories for data and code. When you log onto another computer you should not put everything in your home directory. A little organization at the beginning can save you a lot of time later when you try to figure out which files belong to what project. You can create a new directory using the `mkdir` command (for make directory). Let's make a directory for the work we do in this course:
```bash
$ mkdir hpc_course
```

> ## Spaces in directory names
> You may have noticed that we separate different parts of a command with spaces. The command line uses spaces to parse each part of the command. For this reason, you should not create directories with spaces in them, because if you then try to do something with them from the command line you need to add special characters to group the multiple words together. It is common to use underscores or dashes between words. 
{: .callout}

### Changing Directories
Creating a directory does not move you into the new directory. To change directories you use the `cd` command. For example:
```bash
$ cd hpc_course
```

To move backwards (or up) a directory (for example to move back to your home directory) use `cd ../`

> ## Exercise
> Move into your `hpc_course` directory. Verify that you are in the correct directory, then create two new directories: code and data. Verify that your directories have been created.
>> ## Solution
>> ~~~
>> ```bash
>> cd hpc_course
>> pwd
>> mkdir code
>> mkdir data
>> ls
>> ```
>> ~~~
>>{: .output}
> {: .solution}
{: .challenge}

> ## Using tab to auto-complete
> It can be tiring to type out the name of every file and every directory and it can also be frustrating when you mistype a word. The shell will auto-complete a filename or directory name if you have typed enough of the word to uniquely define it by pressing the tab button. If there is more than one possibility, press the tab button twice to display the different options.
{: .callout}

### Going backwards
Once you have gone into a directory, how do you get out? `../` is the shells way of saying "go back a directory". For example, we are currently in the `hpc_course` directory. If you type `cd ../` you will be in your home directory.
```bash
$ pwd
$ cd ../
$ pwd
$ ls
$ cd hpc_course
```


## File Manipulation
Let's create a simple script that prints "hello world" to the screen. We will use the editor nano. The great thing about nano is that it tells you how to save and exit in the screen, it is also ideal for ssh as it opens directly in the shell window you are using. 
```bash
$ nano shell_example.sh
```
In the window that pops up, let's type `echo "hello world`. Then press control-o to save (Write out) and control-x to exit. 
Oops - we just create that script in our top level directory and it belongs in our code directory (because it is a piece of code). We can move the file to the code directory with the `mv` command. The format is `mv` thing-you-want-to-move where-you-want-to-move-it
```bash
$ mv shell_example.sh code
```
`mv` can also be used to rename a file, you can think of this as moving it from one file name to another filename. In this case where-you-want-to-move-it is the new name of the file. Let's rename the file to something more descriptive `hello_world.sh`. Don't forget we moved the file to our code directory, so we have to go there first before we can rename it.
```bash
$ ls
$ cd code
$ ls
$ mv shell_example.sh hello_world.sh
```
Instead of moving or renaming a file, you can create a copy of the file with the `cp` command. The format is the same as `mv`
```bash
$ cp hello_world.sh hello_world_copy.sh
```
> ## including paths in cp and mv
> You do not always have to be in a directory to copy or move a file. If the file you want to move is not in your current directory, you can refer to the file you want to move with both the path from your current directory and the filename. Similarly, where you want to move a file can also include a path. Let's say I was in my `hpc_course` directory and I want to copy my hello_world.sh file to hello_world_3.sh. The format looks like this: 
> ```bash
> $ pwd
> $ cd ../
> $ cp code/hello_world.sh code/hello_world_3.sh
> ```
{: .callout}

### deleting files
You may accidentally create file and want to delete it. This can be done with the `rm` command which stands for remove. Be careful, the `rm` command permantly deletes a file - this is not like putting it in the trash can or recycle bin where you can recover it. For that reason, we recommend you use the `-i` flag which double checks with you before it deletes a file. Now we can remove our hello_world_3.sh file.

```bash
$ cd code
$ rm -i hello_world_3.sh
```

> ## Exercise
> Use nano to edit your hello_world_copy.sh file to print something else. Rename your file to something descriptive of what it prints. 
>> ## Solution
>> ~~~
>> ```bash
>> nano hello_world_copy.sh
>> ```
>> change "hello world" to "training script"
>> Save and exit
>> ```
>> ```bash
>> mv hello_world_copy.sh training.sh
>> ~~~
>> {: .output}
>{: .solution}
{: .challenge}

## File permissions - who owns what?
Different files on different systems belong to different people and you don't want anyone to be able to do anything to any file. File permissions restrict access to files and directories based on an individual or a defined group. This is like having a locked office door. There are 3 types of permissions: read (r), write (w), and execute (x). Reading a file allows you to look at the file (or directory) but not modify it. Write permissions allow you to modify the file (or directory). Execute allows you to execute a script. There are also 3 sets of permissions to set: permissions for the owner of the file, permissions for the group that the file belongs to, and permissions for everyone else. Let us take a look at the permissions of the files in our directory. To view the current permissions you can type:

```bash
$ ls -l
```
```output
> TODO
```
The output has the following format `<type><permissions> <link> <owner> <group> <size> <date modified> <name>`. The first character is the type - we will skip this and go directly to the 9 characters after that. The first three are the permissions for the owner. They will always be listed in the order read, write, and execute. If the letter is there than that permission is enabled. For instance if the first three charcters were `rw-` then the owner would have permission to read and write a file or directory but not permission to execute it. The next three characters are the groups permissions. Anyone who belongs to the group listed in the fourth column is assigned these permissions. The permissions work the same way as the owner's permissions. For instance, if the middle three characters are `r-x` then anyone in the group has permission to view the file and to execute it, but not to modify it. Finally, the last three charcters are for everyone else. 

> ## What groups do I belong to?
> To figure out what groups you are part of (which can be useful to understand if you have permission to do something) you can type 
> ```bash
> $ groups
> ```
{: .callout}

You modify the permissions on a file or directory using the `chmod` command. You pass to this command whose permissions you want to modify, owner (o), group (g), everyone else (o), or all users (a), what permission you want to modify (r, w, or x) and whether you want to add (+) that permission or remove (-) it. For example, to give everyone else the ability to execute our hello_world.sh script we would type:

```bash
$ ls -l hello_world.sh
$ chmod o+x hello_world.sh
$ ls -l hello_world.sh
```
> ## Exercise
> What are the permissions on the training.sh? Who owns the file? What group does it belong to? Modify the permissions to remove the groups ability to read the file. Double check that the permissions changed. Then add the permissions back.
>> ## Solution
>> ~~~
>> ```bash
>> $ chmod g-r training.sh
>> $ ls -l
>> $ chmod g+r training.sh
>> ```
>> ~~~
>>{: .output}
>{: .solution}
{: .challenge}

## Understanding what is happening on the whole system
Later in this lesson you will learn how to monitor specific tasks that you run on the HPC. Sometimes you want information about the file system or what processes are running outside of the HPC task manager.
When you are working on an HPC you are using a shared resource. It can be helpful to know how much of that resource you are using. You can do this with the `du -h <directory>` command. The -h makes the output format human readable (e.g. the size is in Kb, Mb, Gb). First, we will look at the size of our home directory.

```bash
$ du -h /home/edu02
```
```output
> 40K	/home/edu02
```
> ## Interrupting a command
> Help! you forgot to add a directory and now it is printing the size of every file. `ctl+c` will interrupt the command and return your cursor and command line.
{: .callout}

Another really useful command is seeing what processes are running and who is running them. You can do with the `top` command. The important parts of the output are the PID (process id), USER (who is running the process), %CPU (what percentage of the CPU is being used by that process), %MEM (what percentage of the memory is being used by that process), TIME (how long has the process been running), and COMMAND (what is the command that was run). If you are worried something you did is taking too long or the computer is running slower than you expect, running `top` is a really good way to get an overview of who is doing what on the system. Note that this will continue to run until you tell it to stop. Type `q` to exit.

## Getting files to and from the HPC
HPCs are a great resouce for computing - but they are not a long term storage solution. You will want to move the files from the HPC to a file system that you control. You may also want to prototype a script locally and then move it to the HPC and run it. There are three ways you can move files back and forth: `scp`, `rsync`, and using GitHub (or other version control).

`scp` stands for secure copy. The command format is `scp <what you want to copy> <where to put it>` and these paths are always specified from where you are. Because you will be going from one system to another - one of the locations will include both the address to the system and the path, separated by a colon. For this part, we will exit Bura. You can do that by typing `exit`.

Now we will use `scp` to copy our `hello_world.sh` script to our local directory (`.`). After executing the `scp` command you will be asked for your password. Use your ssh password. 
```bash
$ scp edu02@172.16.55.121:/home/edu02/hpc_course/hello_world.sh .
```
```output
> edu02@172.16.55.121's password: 
> hello_world.sh                                     100%  130     0.1KB/s   00:01
```
Another option for moving files is `rsync`. This actually checks that the file or directory has been updated and only moves new things. The format is the same as `scp`: `rsync <what you want to copy> <where to put it>`.

Finally, if you are using version control to track your development and have a remote server (e.g. GitHub, Bitbucket). Then you can use this to create another copy of your repository on the HPC and transfer files via the remote server.

> ## Exercise
> Use `scp` or `rsync` to move the files you downloaded for this course to Bura.
>> ## Solution
>> ~~~
>> ```bash
>> $ TODO
>> ```
>> ~~~
>> {: .output}
>{: .solution}
{: .challenge}

## Printing to the screen
Sometime you want to write a message to the screen. This can be done with the `echo` command with the fomat `echo <thing to print>`. For example, to print "hello world":

```bash
$ echo "hello world"
```
```output
> hello world
```

{% include links.md %}
