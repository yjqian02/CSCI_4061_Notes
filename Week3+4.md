## [Week 3 & 4 | I/O High Level](Week3.md)
### Goals
1. Understand how to manipulate processes using fork, exec*, wait.
2. Understand the purpose of the high-level I/O library and the features it 
provides
3. Understand the power of file descriptors and how to do I/O redirection
4. Time-permitting -- Understanding terminal I/O control
### I/O Device Abstractions
as an abstraction, I/O is a source/sink for data in raw bytes
I/O operations
- open/close devices
- read from devices
- write to devices
- control devices

### High Level I/O
- goes futher with the abstraction
- hides "device" or low-level I/O
- while low-level is an abstraction for the source/sink for data, high-lvel is more than raw bytes
- some features of high-level I/O
  - stream abstraction
  - formatted/typed I/O
  - string-based I/O
  - line-oriented buffering

### Low level I/O
- system call interface
- high-lvel I/O `<stdio.h>` calls low-level interface
- can also call low-level interfact
- the abstraction is that it's a source/sink for data or a file descriptor

key ideas of low level I/O
- I/O devices and descriptors
- there is no notion of streams (FILE)
- no formatted I/O, just bytes
- more control (read 1 char, don't buffer)

### Files and Streams
what is a unix `FILE` object? 
- delibers an ordered sequences of bytes
- define in `stdio.h>`
- user space library
- built on top of low-level file descriptors
- there are 3 default streams
  - `stdin`
  - `stdout`
  - `stderr`

#### `FILE` objects
- point to actual files
- have current offset
- have mode: read, write, append, etc.
- buffers
- write buffering (open for write)
  - internal character buffer size of BUFSIZE
  - write are done to in memory buffer
    - when buffer is full, write out buffer
    - or if line buffered stream (`stdout` ex) when `\n` is written
- read buffering (open for read)
  - reads are done from in-memory read buffer
  - when a buffer is empty, we read a chunk of BUFSIZE
  - !!! why is internal buffer for I/O advantageous?
#### Buffering: FFLUSH
- sometimes we want to force buffers to spill to the OS to see data immediately
`int fflush(FILE *stream);`
example: flushing
```
fprintf(f, "your output is %d");
fflush (F); // force flush
```
`stderr` always causes buffer to be flushed
`stdout` with `\n` as well
### Open/Close
example: open and close a file
```
#include <stdio.h>
// null on failure, errno is set
FILE *fopen(const char *filename, const char *mode);
mode = "r", "w", "a" // (can do combos with r+ w+)
// returns - on success EOF otherwise (-1)
int fclose(FILE *stream);
```
I/O access is assumed to be sequential
the `FILE*` keeps track of the current file offset for read and write
#### Types of file I/O
##### 1 | Character based I/O
example: character based I/O
```
#include<stdio.h>
// a subset of calls that return EOF on failure or end
int fgetc(FILE *stream);
int fputc(int char, FILE *stream);
int getc(); // stdin
int putc(int char); // stdout
```
##### 2 | String based I/O
- reads `nsize-1` characters or up to a newline
- if newline, then `\n` goes into `buf`
- caller allocs `buf`
example: string based I/O
```
// returns buf or NULL if at EOF or an error occurs
char *fgets(char *buf, int nsize, FILE *inf);

// outputs buf 
// returns last char written or EOF if error
int fputs(const char *buf, FILE *outf);

// also look into gets and puts

```
`stdin` is a line-oriented device
- `\n` EOL input is read when `\n` is seen or EOF
- `^D` is EOF

`fgets` on `stdin` will include `\n` in the string while `gets` does not
also remember stsring library functions **are not** system calls
so remember to allocate `\0` on your own
string library functions:
- `strcpy`
- `strncpy`
- `strlen`
- `strtok`
- `strcmp`
- `strdup`

#### Formatted I/O
`int fprintf(FILE *outf, const char *fmt, args);`
`int sprintf(char *string, const char *fmt, args)`
can also use `printf`, but this prints to stdout console instead
example: formatted output
```
int x = 4;
char str[100];
FILE *f;
F = fopen("myfile", "w");
fprintf(F, "%d", x);
sprintf(str, "%d %d %s", 12, x, "hello");
```
output: 
12 4 hello

#### Formatted input
to read from a file
`fscanf(FILE *in, const char *fmt, <ptr args .. allocated>);`

to read from or parse a string
`sscanf(const char*, const char *fmt, <ptr args ... allocated >);`

example: formatted input
```
int i1, i2;
float flt;
char str1[10], str2[10];
char *str1, *str2;

sscanf("11 12 34.07 keith ben", "%d %d %f %s %S", &i1, &i2, &flt, str1, str2);
```

##### 3 | Binary I/O
advantages are
- binary I/O can write and read fewer bytes
  - this saves space and is faster
- this works for fixed-size data items and lots of them
- but it must be memory-contiguous (array)

`fread/fwrite(void *buff, size_t size, size-t nitems, FILE *f);`
example: binary I/O
```
typedef struct S{
    intss;    // 8 digits
    int phone; // 10 digits
    } 
info_t;
info_tmine = {12345678, 5384937474};
// fopenF for write
fprintf(F, “%d %d”, mine.ss, mine.phone);
fwrite((void *)&mine, sizeof(info_t), 1, F);

// asciifile: 12345678 538493747 = (8+10)*1 bytes
// binary file: ^a93e&^%8 = (4+4)*1
fread((void *)&mine, sizeof(info_t), 1, F);
printf(“%d %d\n”, mine.ss, mine.phone);

suppose info_tmine [100000];

```
##### 4 | Random I/O
advantages: 
- access data in a non-sequential pattern
example: random I/O
```
#include <stdio.h>
int fseek(FILE *stream, long offf, int whence);
// tells you current file offset
long ftell (FILE *stream);

```
whence is either:
- `SEEK_SET`
- `SEEK_CUR`
- `SEEK_END`
---

### Opening, Reading and Writing from Files
#### Open
example: open a file
```
#include <sys/types.h>
#include <stat.h> // depends on gccversion
#include <unistd.h>
#include <fcntl.h>

// create a file if it doesn't already exist
int creat(char *pname, mode_t mode);

// open a file, returns a file descriptor, returns -1 and sets errno on failure
int open(char *pname, int flags, mode_t mode);

```

flags for opening a file:
-  O_RDONLY, O_WRONLY, O_RDWR, O_APPEND,O_CREAT 
-  O_CREAT: will create file if not already there 
-  O_TRUNC: O_WRONLY => truncates length to 0 
-  O_SYNC: flush writes  
-  O_NONBLOCK: non-blocking I/O

#### Close
`int close (int fildes);`

- there are a limited # of allowable open files and fd's
- all open files are closed when a process exits .. assuming only this process has that file open

#### How to get a fd from a file
` int fileno(FILE *stream);` returns the associated fd
or use `FILE *fdopen(int fd, const char *mode);`
#### Read
`ssize_tread (intfiledes, char *buffer, size_tn);`
- Returns # of bytes actually readup to EOF or n
- Returns 0 if at EOF, -1 if an error
- buffer must be allocated: output parameter!
- Reads are sequential w/r to current file pointer
- Reads block by default
- Read can fail when !!!

example: read
```
char buf1[12], buf2[12];
intfd, n1, n2, n3;
fd= open (“foo”, O_RDONLY);
n1 = read (fd, buf1, 11); // n1 is 11
n2 = read (fd, buf2, 11);// n2 is 11
n3 = read (fd, buf1, 11);// n3 is 9
```

example: read and count characters in a file (read large chunks)
```
#define BUFSIZE 512
void main () {
    intfd;
    int total = 0;
    fd= open (“somefile”, O_RDONLY);
    printf(“Size = %d\n”, total);
}

```

example: loop through file to read
```
#define BUFSIZE 512
void main () {
    int fd;
    int total = 0;
    ssize_tnread;
    char buffer [BUFSIZE]; 
    fd= open (“somefile”, O_RDONLY);
    // loop until EOF
    while (nread= read (fd, buffer, BUFSIZE)) > 0)
        total += nread;  // why not BUFSIZE?
    printf(“Size = %d\n”, total);
}
```

example: common read error
```
#define MAX_SIZE 1024
char *buffer;
ssize_tamt;
...
amt= read (fd, buffer, MAX_SIZE);
```
- this is a null buffer
- !!! not quite sure what will happen

### Write
`ssize_t write(int filedes, const char *buffer, size_t n);`
returns # of bytes actually written, if this is < n then that's usually an issue
example: write
```
#define PERM 0644
char header1[512]=“aaa...”, header2[512]=“bbb...”;
int fd;
ssize_tw1, w2;
...
fd= open (“newfile”, O_WRONLY|O_CREAT, PERM);
w1 = write (fd, header1, 512);
w2 = write (fd, header2, 512);

```
- writes are done sequentially

example: copy a file
```
#define BUFIZE 512
#define PERM 0644  
// user can read/write, group/other can only read
void copyfile(constchar *name1, constchar *name2) {
    int infile, outfile;
    ssize_t nread;
    char buffer [BUFSIZE];
    infile= open (name1, O_RDONLY);
    outfile= open (name2, O_WRONLY|O_TRUNC|O_CREAT, PERM);
    while (nread= read (infile, buffer, BUFSIZE)) > 0)
    write (outfile, buffer, nread);close (infile);close (outfile);
}
```

### Buffer writes and symbolic names
OS buffer writes
- use O_SYNC flag on open in write mode
- `outfile = open(name2, O_WRONLY|O_TRUNC|O_CREAT|O_SYNC, PERM)`
- or at any point in time `int fsync (int fd);`

symbolic names
- STDIN_FILENO, STDOUT_FILENO, STDERR_FILENO

---
### File descriptor inheritance
example: fd inheritance
```
int fd, fd1, fd2, pid;
fd = open (“my_file”, O_RDONLY, 0);
pid = fork ();
if (pid!= 0) {  
    read (fd, ...);
    fd1 = open (“foo”, ...);
}
else {
    read (fd, ...); // the same fd
    fd2 = open (“bar”, ...);
...}

```
- fds and fd table are inherited by children at fork()
### Redirection
involves 
- duplicated file descriptors 
- fd preservation

### duplicating fds
example: duplicating fds
```
#include <unistd.h>
int dup2 (intfd1, intfd2);
```
- closes fd2 if open and frees up fd2
- makes fd2 now point to what fd1 points to

example: output redirection with shell
```
shell>cat foo.bar > baz.out
```
where writes to stdout are redirected to `baz.out`

example: output redirection with file
```
fd = open (“baz.out”, O_CREAT|O_WRONLY, ...);
dup2 (fd, 1); // 1 now refers to fd
close (fd);   // might as well ...
write (1, ...); // writes to “standard out” => baz.out

```
since we lost stdout here, we can reopen it with
`fd = open (“/dev/tty”, O_WRONLY);`

example: input redirection with shell
`shell> wc -c`
`shell> wc -c < mydata.txt` if you want to use a file instead

example: input redirection function
```
fd = open (“mydata.txt”, O_RDONLY, ...);
dup2 (fd, 0);
close (fd); // not needed but good style

// reads from “standard in” directed to mydata.txt
read (0, ...);
```

**redirection summary**
- not so useful within a single process 
  - if I want to read/write to a file, I can just do it
- how does the shell do it for us? 
  - `shell> cat foo.bar > baz.out`
  - `cat` delibers output to stdout and we don't want to modify the code of `cat` or `ls`

### fd Preservation
example: fd preservation
```
int fd, pid;
char fd_str[2];pid = fork ();
if (pid!= 0) {     
    read (fd, ...);
}
else {
    fd = open (“source_file”, O_RDONLY, 0); 
    read (fd, ....); 
    sprintf(fd_str, “%d”, fd);
// fd’spreserved through exec!
    execl(“./foo (char*)”foo”, (char*)fd_str);
    ...
}
// inside foo.c
int main (int argc, char *argv[]) {
    int fd;
    fd = atoi(argv[2];
    read (fd, ... ); // will read from “source_file”
    ...
}
```
### inside cat
- by default cat write to stdout
- same for virtually all shell commands/programs

example: shell: fd duplication + preservation
`shell> myprog souce > output`
```
int pid;
pid = fork();
if (pid!= 0) {  // shell parent
    ...
    wait (NULL);
}else {
    // shell child
    // redirect name of file provided to the shell ‘>’ case
    int o_fd;
    char *s_file; // extract from argv-- source
    char *o_file; // extract from argv-- output
    o_fd = open (o_file, O_WRONLY); 
    dup2 (o_fd, 1);  
    execl(“myprog...”, (char*)&s_file, NULL);
```




