## [Week 6 | Communication](Week6.md)
### What do we mean by communication? 
abstracting the idea of having a conduit for data exchange between 2+ processes or threads

### IPC
#### What is IPC?
Inter-process communication (IPC) is a mechanism that allows processes to 
communicate with each other and synchronize their actions

#### Pipes
- are the most basic form of IPC in UNIX
- have both a read and write end

example: create a pipe

```
#include <unistd.h>
int pipe (intends[2]); // returns -1 on failure
```
- ends is a 2-integer fd array that represents the ends of the pipe 
- ends[0] is the “read-end” (receive) and
- ends[1] is the “write-end” (send)
- Integrated into filesystem
- Link is “named” by the pipebut we do not name the reader/writer processes

example: single process pipe program
```
#include <unistd.h>
#include <stdio.h>
#define MSGSIZE 16
char *msg1 = "hello, world #1";
char *msg2 = "hello, world #2";
void main() 
    char inbuf [MSGSIZE];
    int ends[2];

    if(pipe(ends) == -1) {
        perror("pipe error");
        exit(1);
    }
    // write (send) down pipe
    write (ends[1], msg1, MSGSIZE);
    write (ends[1], msg2, MSGSIZE);
    // read (receive) from pipe
    read (ends[0], inbuf, MSGSIZE);
    fprintf(stderr, “%s\n”, inbuf);read (ends[0], inbuf, MSGSIZE);
    fprintf(stderr, “%s\n”, inbuf);
    
```
output is: 
    hello, world #!
    hello, world #2

example: multi process pipe program
```
#include <unistd.h>
#include <stdio.h>
#define MSGSIZE 16
char *msg1 = "hello, world #1";
char *msg2 = "hello, world #2";
void main() 
    char inbuf [MSGSIZE];
    int ends[2], j;
    pid_t pid; // for forking

    if(pipe(ends) == -1) {
        perror("pipe error");
        exit(1);
    }
    return 1;
```
##### Read/Writing from pipes
function: write
```
write (ends[1], msg, MSGSIZE);
```
function: read
```
read (ends[0], inbuf, MSGSIZE);
```
###### Knock-Knock example
**solution 1**
parent
```
pipe (ends);
fork (); // parent
write (ends[1], “k-k”, ...);
read (ends[0], buf, ...);
write (ends[1], “orange”, ...);
read (ends [0], buf, ...);
write (ends[1], “aren’t ...”, ...);
```
child
```
read (ends[0], buf, ...);
write (ends[1], “w-t?”, ...);
read (ends [0], buf, ...);
write (ends[1], “orange-who?”, ...);
read (ends [0], buf, ...);
```
issue with this solution is that you don't know when child reads so if parent wants to write and read from the same pipe it gets messy

**solution 2**
parent
```
intP_C[2], C_P[2];pipe (P_C);
pipe (C_P);
fork (); // parent
close (P_C[0]);close (C_P[1]);
write (P_C[1], ...);
read (C_P[0], ...);
```
child
```
close (P_C[1]);
close (C_P[0]);
read (P_C[0], ...);
write (C_P[1], ...);
```
##### Non-blocking pipes
- the default I/O behavior is blocking
- but non-blocking I/O can be useful
  - since pipe is a file, it can contol attributes

example
```
#include <fcntl.h>
intfcntl(intfd, intcmd, ...);
intends[2], flags, nread;
pipe (ends);
flags = fcntl(n_0, F_GETFL, 0); // correctd error on slide fd = n_0 instead
// fcntl allows you to change behavior of many different types of fds
fcntl(ends[0], F_SETFL, flags | O_NONBLOCK); 
...
nread= read (ends[0], buf, size);

```

controller example
```
int comm_r[N][2], comm_w[N][2];
for(i=1 to N) {
    pipe(comm_r[i]);
    // break if child process
    // why? we don't want the child to create any pipes, etc since
    // that is the controller's job
    if((pid = fork() == 0)){
        break;
    }
    // make read ends of each pipe non-blocking
    // because you don't know which pipe will write/read at
    // any point in time
    !!! don't understand how to do this
}
// if parent
if(pid > 0) {
    while(1) {
        for(i = 0 to N) {
            !!! don't understand how this works
            n = read(comm_r[i][0],/* whatever you read it into */);
            if(n == -1 && errorno == EAGAIN) {
                // space to do something
            }
        }
    }
}

```

##### Piping in shell

example: how to use pipes in shell
run the folling command in shell

`ps–u weiss039 | grep tcsh`

then run program
```
pipe (ends);
if (childpid= fork ()) == 0) {
    dup2(ends[1], 1);
    close (ends[0]);
    execl(“/bin/ps”, ....);
}
else { // forks again (probably?)
    dup2(ends[0], 0);
    close (ends[1]);
    execl(“/bin/grep”, ...);
}


```
example: sending message into a pipe or any fd

```
typedef struct{
    intx;
    inty;
    char str[20];
} 
message_t;
message_tm1, m2;
int ends[2];
pipe (ends); // check for error!

// send m1 into the pipe
write (ends[1], &m1, sizeof(message_t));

// pull data into m2 from the pipe
read (ends[0], &m2, sizeof(message_t));

// NOTE: Must be contiguous data
```
- idea is that you can write any data you want into a pipe
- the only req of what you write/read is that it is contiguous data in memory
  - !!! why is that a requirement?


