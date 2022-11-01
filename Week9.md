# Week 9 Threads

*see week 8 slides part 2*
## Top down motivation
- web server must exploit concurrency to service requests efficiently
  

## Thought question about NxN matrix multiplication on a uniprocessor
issues involve
- there is little concurrency
- there is thread creation overhead

- but if you had 4 cores (no longer uniprocessor) then you'll have parallelism

*switch to week 9 slides*
## Pthread: Creation
creating a thread is a combination of fork() and exec()

example: create pthread
```
#include <pthread.h>
int pthread_create(pthread_t *thread, pthread_attr_t *attr, void *(*function)(void *), void *arg)
```
`pthread_t *thread` = which thread id to return
` pthread_attr_t *attr` = attributes you want to establish for thread beyond the default (give it more priority, more stack, etc)
`void *(*function)(void *)`

## Parameters
```
void *thread_fn(void *arg) {
    printf(“%d”, *((int  *)arg );
}
```

