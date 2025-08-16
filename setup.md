---
title: Setup
---

You will need the following software installed and working correctly on your system to be able to follow the course.

> ## Common Issues & Tips
> If you are having issues installing or running some of the tools below,
check a list of [common issues](./common-issues/index.html) other course participants encountered and some useful tips for using the tools and working through the material.
{: .callout}

## Command Line Tool
You will need a command line tool (shell/console) in order to run Python scripts and connect to Bura. For the purposes of this course, Windows command line tool `cmd`
provides sufficient functionality. On macOS and Linux, you will already have a command line tool available on your system. You can use a command line tool such as [**Bash**](https://www.gnu.org/software/bash/),
  or any other [command line tool that has similar syntax to Bash](https://en.wikipedia.org/wiki/Comparison_of_command_shells),
  since none of the content of this course is specific to Bash. Note that starting with macOS Catalina,
  Macs will use [Zsh (Z shell)](https://www.zsh.org/) as the default command line tool instead of Bash.

To test your command line tool, start it up and type:
~~~
$ date
~~~
{: .language-bash}

If your command line program is working - it should return the current date and time similar to:
~~~
Wed 21 Apr 2021 11:38:19 BST
~~~
{: .output}

## Bura HPC setup
While many of the code examples in this course can be executed on a local PC, the parts related to the Slurm workload manager
require having access to an HPC facility. The registered participants of this course received an email with instructions on how to connect to the Croatian in-kind contribution
HPC called _Bura_. If you encountered problems while following these instructions, please contact the organizers of the workshop.

{% include links.md %}
