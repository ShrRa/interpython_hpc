---
title: "Setting the Scene"
start: false
colour: "#FBED65"
teaching: 10
exercises: 0
questions:
- "What are we teaching in this course?"
- "To whom will this course be useful?"
objectives:
- "Setting the scene and expectations"
- "Making sure everyone has all the necessary software installed"
keypoints:
- "Astronomical research requires large computing resources, which are not always available within a PC form-factor."
- "In order to run your code in a High-Performance Computing setting, special tools and techniques are needed."
---

## Introduction

Astronomy has always been a data-hungry field. From the discovery of Neptune to state-of-the-art cosmological simulations, such as [Illustris](https://www.illustris-project.org/) or [FLAMINGO](https://flamingo.strw.leidenuniv.nl/), we make discoveries by working with datasets that push the limits of our processing capabilities, and sometimes exceed them. To handle gigabytes of observations or millions of simulated particles, we must optimize our code as much as possible, pay close attention to how we use available RAM and processors, and eventually turn to more powerful computers than we can fit on our desks.  

In this course, we provide an introduction to **High-Performance Computing** — a set of approaches and techniques for using *supercomputers* and *computer clusters*. While modern HPC facilities are often built from the same components as “ordinary” PCs and can run your code with minimal adjustments, careful planning is required to avoid bottlenecks such as data transfer between *nodes* and to take full advantage of massive parallelization.  

A full course on High-Performance Computing would take many hours of lectures and practical exercises. Here, over the next two days, we will cover the basics of the field and practice what we’ve learned on one of the LSST HPC facilities, the Croatian supercomputer **Bura**.


The course is organised into the following sections:

### [Section 1: HPC Intro](../10-section1-intro/index.html)
On day 1, we will do a general overview of what is considered to be HPC, which approaches for speeding up your software exist, and how to understand whether your algorithm
will work faster if you try to run it on a cluster or supercomputer. After a brief refresher on Command line usage, we are also going to get familiar with the **Bura** supercomputer 
and learn about one of the most common *job manager* tools called **Slurm**.

### [Section 2: Running and adapting your code to HPC](../20-section2-intro/index.html)
The second-day episodes are dedicated to running code examples on Bura and studying how various implementations utilise the resources available in an HPC environment. 
We will compare code performance when it is run on a single CPU, multiple CPUs or GPUs, how different parallelization instruments work, and learn how to use resource management tools for determining which aspects of your algorithm require
further improvements.

## Before We Start

A few notes before we start.

> ## Prerequisite Knowledge
> This is an intermediate-level software development course
> intended for people who have already been developing code in Python (or other languages)
> and applying it to their own problems after gaining basic software development skills.
> So, it is expected that you  have some prerequisite knowledge on the topics covered,
> such as Python imports, variables, and loops, virtual environments, and executing commands in your OS terminal.
> While we attempted to make the materials clear and understandable to a wide range of expertise levels,
> if you are not familiar with Python and the command line, we recommend that you e.g. go through one of the entry levels
> Carpentries workshops, such as [Programming with Python](https://swcarpentry.github.io/python-novice-inflammation/).
{: .callout}

> ## Setup, Common Issues & Fixes
> Have you [setup and installed](../setup.html) all the tools and accounts required for this course?
> Check the list of [common issues, fixes & tips](../common-issues/index.html)
> if you experience any problems running any of the tools you installed -
> your issue may be solved there.
{: .callout}

> ## Compulsory and Optional Exercises
> Exercises are a crucial part of this course and the narrative.
> They are used to reinforce the points taught
> and give you an opportunity to practice things on your own.
> Please do not be tempted to skip exercises
> as that will get your local software project out of sync with the course and break the narrative.
> Exercises that are clearly marked as "optional" can be skipped without breaking things
> but we advise you to go through them too, if time allows.
{: .callout}

> ## Outdated Screenshots
> Throughout this lesson we will make use and show content
> from various interfaces, e.g. websites, PC-installed software, command line, etc.
> These are evolving tools and platforms, always adding new features and new visual elements.
> Screenshots in the lesson may then become out-of-sync,
> refer to or show content that no longer exists or is different to what you see on your machine.
> If during the lesson you find screenshots that no longer match what you see
> or have a big discrepancy with what you see,
> please [open an issue]({{ site.github.repository_url }}/issues/new) describing what you see
> and how it differs from the lesson content.
> Feel free to add as many screenshots as necessary to clarify the issue.
> 
{: .callout}

> ## Let Us Know About the Issues
> The materials were prepared specifically for this workshop. They weren't used before,
> and there may be typos, code errors, or underexplained or unclear moments.
> Please, let us know about these issues. It will help us to improve the materials and make
> the next workshop better.
{: .testimonial}

{% include links.md %}
