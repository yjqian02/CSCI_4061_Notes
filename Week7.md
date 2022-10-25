## Week 7 Communication Continued
### Summary of what we've covered about IPC so far
- files
- pipes
- limitations

### Message-Passing
UNIX has a mail-box like mechanism for message passing
- message-queue
- sender puts messages into ququ
- receiver pulls them out

```
#include<sys/msg.h>
int msgget (key_t key, int permflags);
// returns queue_id used for rend/receive
```

About message queues:
- message queue is typically larger than a pipe buffer
- unrelated processes can share queue
  - !!! what do you mean by unrelated processes? what qualifies as unrelated?
- queues are persistent: they may outlive the process that created them
- they are meant for discrete messages (compare them with a near infinite data stream)
  - !!! what is a near-infinite data stream? 
- queues only work on the same machine though

`int msgsnd(int qid, const void *message, size_t size, int flags)`
`int msgrcv(int qid, void *mesage, size_t size, long msg_type, int flags)`
- for flags, there is only 1 useful flag
- both these functions return an error if the queue no longer exists
- the data type of the message is 
  - `typedef char data_t [SOMESIZE];`
  ```
  struct mymsg_t {
    long mtype; // used for tag selection
    data_t data; // a collection of contiguous bytes
  }
example: sending message
```
mymsg_t m1 = {15, “hello”}, mymsg_t m2 = {20, “goodbye”};
// permissions allow u,g,o to read/write into queue
int mid;
key_t key = 100;
mid = msget(key, 0666| IPC_CREAT); 
msgsnd(mid, (void *)&m1, sizeof(data_t), 0);
msgsnd(mid, (void *)&m2, sizeof(data_t), 0);
// msgsnd will block if queue is full, otherwise:
msgsnd(mid, (void *)&m1, sizeof(data_t),IPC_NOWAIT);
```
program returns -1 if it cannot send (errno = ENOMSG)

example: receiving message
```
data_t msg;
int mid;
key_t key = 100;
mid = msget(key, 0666 | IPC_CREAT); 
// read msgs with tag 15 and 20
// will block if such messages are not there
msgrcv(mid, (void *)&msg, sizeof(data_t), 20, 0);
msgrcv(mid, (void *)&msg, sizeof(data_t), 15, 0);

// non-blocking:
res = msgrcv(mid, (void *)&msg, sizeof(data_t),30, IPC_NOWAIT);
// Returns -1 if not on queue (and errno= ENOMSG)
```
Different cases depending on msg_type:
if msg_type = 0 then we return the oldest message with:
- `msgrcv(mid, (void *)&msg, 0, 0);`
if msg_type < 0 then we return the message with the smallest tag up to X where X = abs(tag)
- `msgrcv(mid, (void *)&msg, -99, 0);`
- return msg with smallest tag <= 99
- implements prioritties

### remove queue
`msgctl(int qid, IPC_RMID, 0);`

### Shared-Memory in UNIX
- shared memory allows two or more processes to share a segment of physical memory
- !!!why is this the more efficient form of IPC? 
- !!!why do we have to use it carefully? 
- Which IPC methods should we use? 
    - it's more of a personal preference

#### Steps for shared memory
1. create shared memory segment
```
#include <sys/shm.h>
int shmget(key_t key, size_t size, int permflags);
// returns segment id (shmid) for subsequent calls
// key is a unique key
// size is mem size
```
- do this once, return the handle afterward
2. each process must attach to the segment (extends their VAS)
!!! what is a VAS?
```
void *shmat(int shmid, const void *daddr, int shmflags);
```
// returns start address of segment error(void *) -1, can be different in diff processes (virtual addresses)
// daddr is
- use the handle for this
3. detach from shared-memory segment
`int shmdt (void *arg);`
arg is returned ptr from shmat
4. remove shared memory segment for good
`shmctl(shmid, IPC_RMID, 0)`

example: put a shared buffer in a shared memory region
```
#define MaxItems 1024
struct buffer_t{
    int next_slot_to_store;
    int next_slot_to_retrieve;
    item_t items [MaxItems];
    intnum_items;
}
item_t remove_item(buffer_t*b);
void produce_stuff(buffer_t*b, item_t new_item);

// main program using the buffer
void main () {
    int BUFFER_KEY = 100;
    buffer_t *b;
    item_t item;
    shmid= shmget(BUFFER_KEY, sizeof(buffer), 0666 | IPC_CREAT);
    b = (buffer_t*) shmat(shmid, 0, 0);
    b->next_slot_to_store= 0;
    b->next_slot_to_retrieve= 0;
    // initialize item to store
    produce_stuff(b, item);
    ...
    item = remove_item(b);
    ...
    shmdt((void*) b);
} // process can’t use b

// example cont
void produce_stuff(buffer_t*b, item_tnew_item) {
    if (b->num_items== MaxItems)
    return ERROR; // later, we’ll block
    b->items [b->next_slot_to_store] = new_item;
    b->next_slot_to_store++;
    b->next_slot_to_store%= MaxItems;
    b->num_items++;
    return;
}

// another function in example
item_t remove_item(buffer_t*b) {
    item_t item;
    if (b->num_items== 0)
    return ERROR; // later, we’ll block
    item = b->items [b->next_slot_to_retrieve];
    b->next_slot_to_retrieve++;
    b->next_slot_to_retrieve%= MaxItems; 
    b->num_items--;
    return item;
}

```

### Multiple Processes 
- for shared memory to make sense, we need multiple processes
- for example, multiple processes doing
  - `produce_stuff(b, item);`
  - `item = remove_item (b);`

example: assuming shared memory seg is created and buffer is initialized, what might happen? 
!!! don't understand what happens here
```
// first version
void main () { // producer
    int BUFFER_KEY = 100;
    buffer_t *b;
    item_t item;
    shmid= shmget(BUFFER_KEY, sizeof(buffer), 0666 | IPC_CREAT);
    b = (buffer_t*) shmat(shmid, 0, 0);
    while(1) {
        // initialize item to store
        produce_stuff(b, item);
    }
    
} 

// second version
void main () { // producer
    int BUFFER_KEY = 100;
    buffer_t *b;
    item_t item;
    shmid= shmget(BUFFER_KEY, sizeof(buffer), 0666 | IPC_CREAT);
    b = (buffer_t*) shmat(shmid, 0, 0);
    while(1) {
        // get item
        item = remove_item(b);
    }
    
} 
```


