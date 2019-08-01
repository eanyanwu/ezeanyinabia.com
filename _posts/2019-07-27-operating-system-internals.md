---
layout: post
title: Operating system internals
excerpt_separator: <!--more-->
---

I spend a lot of time using computers, but I'm frankly still scared of what goes on beneath the covers.  
To change that, I am following along with the free <a class="txt-link" href="http://pages.cs.wisc.edu/~remzi/OSTEP/">Operating Systems: Three Easy Pieces</a> book offered by Computer Science professors at the University of Wisconsin-Madison

<!--more-->

# Contents
1. [Introduction](#introduction)
1. [Virtualization](#virtualization)
    1. [Virtualizing The CPU](#virtualizing-the-cpu)
        1. [Mechanisms And Policies](#mechanisms-and-policies)
            1. [Process Creation](#mechanism-process-creation)
            1. [Limited Direct Execution](#mechanism-limited-direct-execution)
            1. [Scheduling](#policy-scheduling)
    1. [Virtualizing Memory](#virtualizing-memory)
        1. [Mechanisms And Policies](#mechanisms-and-policies)
            1. [Memory Allocation Api](#mechanism-memory-allocation-api)
            1. [Address Translation](#mechanism-address-translation)
1. [Concurrency](#concurrency)
1. [Persistence](#persistence)


# Introduction

When we say a program is "running" we mean that a program is doing the following over and over again:
- fetch an instruction 
- decode the instruction
- execute the instruction 

This is the Von Neumann model of computing. _That.is.it._

However, just having a single program running at a time is boring.
As it stands it is a simple <a class="txt-link" href="https://en.wikipedia.org/wiki/Batch_processing">batch processing system</a>. You can't run multiple programs at once and you can't interact with the system while it is running your program. We want more. We need more.

The body of software that allows us to interact with our laptops in the way we do is called
the operating system. 
It enables multiple programs to run seemingly at the same time -- giving us the desktop "environment" we are used to. 

As I am writing this in a text editor, my browser is open to the page contining this
lesson, music is playing from another tab on the browser, I have a status bar on the
bottom showing the time, volume, date, battery, and wifi status. I also
have another workspace with a terminal window open. None of this would be
possible without the wild wild things the operating system does behind the scenes
while programs are seemingly just fetching, decoding and executing instructions.

So how does an operating system (or OS) do this magic? Three easy pieces; virtualization, concurency and persistence.


# Virtualization 

Virtualization: "Sharing one peach among many peach-eaters while making them think
they each have a peach to themselves"


## Virtualizing the CPU

Program -- A program is a lifeless thing, sitting on disk. A bunch of
instructions and maybe some data, waiting to spring into action

Process -- A _running_ program.

It is the operating system that takes a program and gets it running,
transforming it into something useful. The operating system needs a
CPU to "run" these programs in.

A usable computer should be able to run many programs at a
time. So how do we create the illusion of having multiple CPUs for these
programs to run on? How do we give the illusion of access other physical
resources that the computer might be connected to ?
The answer is clearly "virtualization"...but how does the
OS do that?

### Mechanisms and Policies

Mechanisms are low-level operations, methods or protocols to implement a needed
piece of functionality. They represent the step-by-step details of how to do somthing.

Policies are the algorithms for making some kind of decision. For example, deciding
which process to run next.

Back to processes. Some terms

__Machine state__: The parts of the machine that are important to the execution
of a running process. Some examples are 
- The memory it can access, also called the __address space__
- The registers it makes use of. 
- The files it has open

__Process API__: The way you expect to be able ot interact with a process. For
example, we should be able to create, destroy, and wait for a process. Other
operations might also be possible. 

__Process List__: Data struction that the OS uses to track information about
processes that are running, ready or blocked. The individual members in this
list are sometimes called PCBs (process control blocks) or process descriptors.


#### Mechanism: Process Creation 

To have multiple processes running on the same computer, the operating system must be able to _create_ them. These is the main Application Programming Interface for process creation:

- `fork()` -> Create a child process that starts executing at the same line that the parent process called the method

- `wait()` -> Wait for child processes to finish

- `exec()` -> Load another program into the current running program's address space and start executing that instead.


#### Mechanism: Limited Direct Execution 

When a process is running on the CPU, nothing else else is, including the operating system. So how can the OS (which is not running) manage processes (which are running)?

A mechanism called Limited Direct Execution is used.
The elements of this are:
- Trap instructions.
    - These trap instructions give control to the hardware, which in turn gives control to the OS with additional priviledges (kernel mode). The OS does its thing, then executes a return from trap instruction, which gives control back to the hardware, which in turn gives control back to the running program.
    - Normal prgrams can pass arguments to these trap instructions by placing values on the stack that are understood by the OS.
    - These are called "System Calls"
- Switching control between process:
    - Cooperative approach: The OS waits for processes to make a system call or perform some illegal action that will give rise to a trap instruction. 
    - Non-cooperative approach: Timer-interrupt. The OS sets up a timer that will interrupt the running code on the CPU every few milliseconds and execute a trap-like statement that will give control back to the OS. Hardware cooperation is needed for this one.
    - Once the OS regains control in some way, it needs to make a decision. To continue running the current process or switch to another one? This decision is the job of the scheduler. 
    - Once the next process to be executed has been chosen, a context switch needs to happen. This means saving some state form the current running process, and fetching some state for the soon to be running process. Then return from trap my guy.


#### Policy: Scheduling

Some Definitions:

> Turnaround Time: The time from when the job entered the system to when it completes

> Response Time: The time from when the job enters the system to when it first starts executing.

Different scheduling policies yield different results for these two metrics. In general, policies that are considered "fair" are terrible for turnaround time but good for response time. 

Polcies that are considered "efficient" are great at turnaround time, but terrible at response time. 

_Multi-Level-Feedback Queue Scheduler_

The crux of the problem: "How can we design a scheduler that both minimizes response time for interactive jobs while also minimizing turnaround time without previous knowledge of job length?

We do so by coming up with the concept of job priorities. 

Here are the rules:
1. If Priority(A) > Priority(B), A runs (B doesn’t)
1. If Priority(A) = Priority(B), A & B run in round-robin fashion using the time slice of the given queue.
1. When a job enters the system, it is placed at the highest priority (the topmost queue).
1. Once a job uses up its time allotment at a given level (re-gardless of how many times it has given up the CPU), its priority is reduced (i.e., it moves down one queue).
1. After some time period S, move all the jobs in the system to the topmost queue


The Pros:
- Appears to offer some balance between response and turnaround time which is what we wanted.

The Cons:
- Voo-doo constants (term was created by John Ousterhout). What values should we pick for a time-slice for each queue? How many queues should we have? What should be the time allotment per-queue? What is the period `S` after which all jobs in the system move to the topmost queue? The questions go on...

_Lottery Shcheduling_

Exaclty what it sounds like. We pick the next process to run at random with the added twist that we can assign "weights" to certain processes so they are more likely to be picked.

The longer the processes run for the more "fair" this scheduling is as each process runs for exactly the proportion of time its weights corresponds to. These weights can be reffered to as "tickets". The more tickets a process has, the more likely it is to be picked when the scheduler is deciding what job to run next.

The Pros:
- Easy to understand. No voo-doo constants

The Cons:
- There is none. All our schedulers should be of the lottery kind `._.` 

> Aside: Linux uses a variant of this and calls it the CFS (Completely Fair Scheduler)
As each process runs, it accumulates vruntime. Once a time-slice is over, the scheduler simply picks the process with the lowest vruntime.

> The time-slice is determined by another knob called `sched_latency`, which is (usually) the period over which the scheduler will be completely fair. It takes this value and divides it by the number of processes, which will give it the time-slice to use.

> Yet another knob is used to make sure time-slices are not too short. This is the `min_granularity`, which is the minimum amount of time a time-slice should be. If the set `sched_latency` implies a value lower than `min_granularity`, it is set to `min_granularity` instead.


## Virtualizing Memory 


We are not yet done with vitualization though. The OS also needs to virtualize any shared hardware that processes might want to use. This includes memory. The virtual memory manager is responsible for giving proceses the illusion that they have a sequential block of memory to themselves. This is called the "address space" for process.

This goal of this illusion is _easy of use_ and _protection_.   
It would be terrible to program in an environment where you needed to make sure you didn't overwrite some other program's memory.  
It would also be terrible if other programs could overwrite your own memory.  

In the early days of computing, this didn't matter much. Computers only needed to only run a few jobs non-interactively. But humans are demanding and eventually we wanted our computers to be interactive! we wanted them fast and capable of running multiple programs at the same time! and they must support multiple concurrent users! `.__.` So OS designers had no choice. Virtual memory it is. Gone are the simple days


### Mechanisms and Policies

#### Mechanism: Memory Allocation Api

Generally, when running a program, there are two types of memory "stack" and "heap". The underlying operating system does not make this distinction. It's all just memory.   
However most programming languages introduce this concept to make a distinction between short-lived and long-lived memory.

Short-lived memory or stack memory only lasts within a function call.  
Long-lived memory or heap memory can last between function invocations.

Stack memory is usually automatically allocated AND de-allocated for you. Heap memory however, you need to take care of yourself.

In UNIX/C programs, two methods are available for allocating heap memory

- `malloc` -> You pass it the size of memory you would like allocated on the heap. If it succeeds, it returns the pointer to that memory
- `free` -> Marks the memory pointed to the pointer argument as free to used again i.e. not-allocated

These are not system calls. They are provided by the C standard library as easier interfaces to the `brk` and `sbrk` system calls.

#### Mechanism: Address Translation

The idea is the same with limited directed execution. The operating must partner with the hardware to be able to provide a good memory interface to programs. 

Specifically, running programs are made to think they have a continuous chunk of memory starting at 0 which they can use. Perhaps unsurprisingly, this is not actually the case.  

If multiple programs are to share memory, the OS needs to be able to place them at different spots in memory, which means the addresses the programs use will be wrong. To fix this issue, the OS uses _address translation_ or _base and bounds_.  

The idea is that the running program thinks it is accessing address 128 to get its next instruction. However, the next instruction for that program might not be at physcal address 128 because of where the program was placed in memory. So the Operating System with great help from the hardware translates that 128 virtual address its physical address.  

This is done by recording a base and bounds for every program. We can think of the _base_ as the offset from 0 to where the program actually resides in memory. So when a program whose base is 10000 tries to access memory location 128, the hardware will actually translate that to the physical memory location 10128. 
The _bounds_ helps enforce the rule that processes can only access their own memory. If a process tries to access memory greater than the _bounds_, the hardware can run a handler (set up by the OS) that will kill the offending process.


