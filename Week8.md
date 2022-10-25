# Week 8 Threads
## What are threads? 
- as an abstraction, they're for executing the instruction stream
- threads exist within a process and share its resources (memory)
    - with the exception that a thread has its own stack and PC
    !!! what's mean by PC here? 
- default is that there is always one thread

## Visual examples of threads
<!-- insert images later -->
what causes the system to switch threads? 
- reading from a blocking pipe
- different thread priorities
- if a thread is completely finished with instructions

## What are the benefits of threads?
- concurrency
  - when one blocks, another runs
  - immediately switching from one thread to another
  - trying to reduce the impact of blocking and waiting
- modularity
  - think of your application as decomposing to a set of functional units
  - calling these units threads
- parallelism
  - threads run in parallel (need multiple cores)
- scale
  - more threads than processors/processes
- overhead
  - cheaper than processes
  - !!! what do we mean by overhead here? 

*can can do simlar things with processes, but at a much coarser granularity (you can have many more threads than processes)

## Give an example application where threads are useful? 
code editor
- there is one thread for the kernal, one for the disk, another to take in keyboard input

## How do we quantify the benefit of threads? 
T(req) = T(read) + T(serve) + T(write)
T(req) = T(read) + h*T(cache) + (1-h)*T(disk) + T(write)


## Give another example of threads
example: webserver
```
```
- there is shared memory (shared buffer and cache)
- threads share globals, head, but not stack


## What are the drawbacks to threads? 
Sharing
- synchronization needed to protect shared data structures
- we can't make assumptions, threads may be switched unpredictably
- there is no isolation of failure

thread-safety (related to sharing)
- not all system calls may be thread-safe
- system calls that can be executed concurrently by multiple threads

global varaiables
- per thread globals might be necessary


example: counter
```
int counter = 0; 
int increment_counter() {
    counter ++;
    return counter;
}
```
what's the issue here? what if threads T1 and T2 both call increment_counter? 

something about an unrolling counter

## Thread safety continued

## When to use locks

## Drawbacks: per thread globals

## Alternatives to threads
  