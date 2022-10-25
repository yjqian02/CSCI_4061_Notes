## [Week 1 | OS Overview](Week1.md)

### There are 2 views on operating systems
1. internal: what is inside the OS
2. user: what can the OS do for me
   1. focuses on abstractions
   2. uses APIs/Libraries

### Concurrency and Parallelism

**concurrency**
- an application is making progress on more than one task at the same time (concurrently)
- resources are shared
- activities occur at the same time
- known as *synchronous* execution
- Concurrency means executing multiple tasks at the same time but not necessarily simultaneously.

**parallelism**
- an application splits its tasks up into smaller subtasks which can be processed in parallel, for instance on multiple CPUs at the exact same time
- need multiple CPUs
- parallelism is a type of concurrency where tasks are executed simultaneously

**with a single CPU view**
- pov which this class takes
- concurrency makes it seem like things are happening in the same time
- parallelism means things *actually* happen at the same time
- we don't really get into parallelism--go to parallel programming for that

**References**
- [comparison](https://medium.com/@itIsMadhavan/concurrency-vs-parallelism-a-brief-review-b337c8dac350)


### Asynchrony
**asynchrony**
- dealing with unpredictable events in time
- ex: exceptions, devices (can't predict when you will use printers, etc), I/O

### Communication
- the transfer of information
- ex: network programming
- probably brings in issues about security


### Operating System Breakdown
**definition**
program that acts as intermediary between computer user and computer hardware
**stakeholders**
| stakeholders   | goals                                                       |
| ------ | ----------------------------------------------------------- |
| user   | cares about speed and ease of use                           |
| system | cares about fairness (for all users to be treated the same) |
| owner  | cares about utilization and reliability of OS               |
| admin  | cares about security                                        |

**roles**
1. referee
   1. resource allocation among users & applications
   2. isolation of different users,, applications from each other
   3. communication between users & applications
2. illusionist
   1. makes each application appear to have the entire machine to itself
   2. infinite # of processors
   3. near inf amount of memory
   4. reliable storage
   5. reliable network transport
3. handy-person
   1. libraries
   2. user interface widgets
   3. drivers

Certain stakeholders may have different goals, creating tension

**interface visualized**
![image](./images/1200px-Kernel_Layout.svg.png)
- by kernal we mean the OS
- CPU, memory, and devices make up the architecture
- OS implements the "idea" of a virtual machine (different from a VM) that's easier to program than raw hardware
### Key OS Concepts
#### kernal
a library of procedures shared by all user programs that is protected
- user code can't accesss internal data structures directly
- user code can *invoke* the kernal only at **well-defined entry point (aka system calls)**
- kernal has direct access to all hardware and handles interrupts and hardware exceptions
- CPU is either executing OS code in **kernal mode** or your code **user mode**

#### Viewpoint of a systems programmer
- can use system calls directly in ASM
  - can use these calls in user mode
  - calls are executed by OS in kernal mode
  - do this when you need to improve efficiency
- language specific libraries can also be used to access system calls
  - for C there is libc.a and glibc.a

*sometimes low-level library calls are referred to as system calls*
*this is technically wrong since system calls are assembly calls*

**process**
- an executing program
- abstracted, it's a container for computing resources
**threads**
- an executing stream of instructions normally within a process
- threads can exist in the OS
**synchronization**
- concurrency (when resources are shared)
  - race condition (MN nice, when nobody knows when to go)
  - deadlock (stuck and overlapping)
**communication**
- files and directories abstract communication between two processes
- two processes are connected by a pipe channel
- processes need to communicate to avoid traffic problems

**memory management**
- will get to later
- the abstracted idea is virtual memory though
**system calls**
- how user programs interact with the OS
- available as ASM instructions
- can also use c library interfaces

