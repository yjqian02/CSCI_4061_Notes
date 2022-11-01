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
  - keyboard gets keystrokes
  - kernal manages internal buffer and updates screen
  - disk autosaves

## How do we quantify the benefit of threads? 
```T(req) = T(read) + T(serve) + T(write)```
```T(req) = T(read) + h*T(cache) + (1-h)*T(disk) + T(write)```

we want to answer 2 important questions:
1. can we improve the performance of a single request with threads? 
   1. observable from a client
2. can we improve the performance of the "service" requests/time? 
   1. measured at the server

## Give another example of threads
example: webserver
```
dispatcher(...){
  while(TRUE) {
    // 1.read (req ~URL)
    get_next_request(&req);
    handoff_work(&req, &buf);
  }
}

worker(...){
  wait_for_work(&buf, &req)
  // 2.service
  look_for_page_in_cache(&req, &answer);
  if(page_not_in_cache(&answer)) {
    read_page_from_disk(&req, &answer);
    put_page_in_cache(&req, &answer);
  }
}
// 3.write
return (&answer);
```
how are these threads interacting? 
- there is shared memory (shared buffer, cache, and disk)
- threads share globals, head, but not stack
- when might worker thread block? 
  - worker thread might block if it has to get something from the disk
- 


## What are the drawbacks to threads? 
Sharing
- **synchronization**(will be covered in detail next) needed to protect shared data structures
- we can't make assumptions, threads may be switched unpredictably
- there is no isolation of failure

thread-safety (related to sharing)
- not all system calls may be thread-safe
- system calls that can be executed concurrently by multiple threads
- if one thread has bug, all the threads will experience that issue (no isolation)
  - in *contrast* if one process has an issue, only that particular process will die, the rest of the processes stay up and running

global variables
- per thread globals might be necessary
example: race condition
!!! update from lecture recording
```
```

example: counter
```
int counter = 0; 
int increment_counter() {
    counter ++;
    return counter;
}
```
what's the issue here? what if threads T1 and T2 both call increment_counter? 
- are are operating on shared variables in a way that can cause race conditions
- then you have an unrolling counter


## Unrolling counter
load counter->R
increment R
store R-> counter

suppose t1, t2 each call increment_counter 50 times, what is the final values of counter? 
what are the possible values of counter? 
- 50
- 100

## Thread safety continued
example: counter fixed with lock
```
int counter = 0;
lock_type counter_lock;
int increment_counter(){
  // lock is held or free: if it's held, caller is blocked
  lock(counter_lock);
  counter++;
  unlock(counter_lock);
  return counter;
}
```

## When to use locks
Locks reduce concurrency and performance (since only one thread at a time can execute when you use a lock), so only use locks when you need to

## Drawbacks: per thread globals
example: errno
```
// T1
syscall_1
sets errno
...{T1 swithces to T2}
reads errno

// T2 
..
syscall_2
sets errno
..
```
- in Unix, errno is a global variable in shared library
- what are our options to guarantee error reporting a thread-safe? 
  - use locks
  - eliminate global: return error code
  - define errno "service" or macro (remapping errno)
    - `#define errno _special_thread_errno(thread_id)`
    - a function based on threadid that stores a table of tread values? 

---




## Alternatives to threads
  