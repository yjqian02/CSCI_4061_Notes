## [Week 5 | File Systems](Week5.md)

### What do we mean by file systems? 
an abstraction
- a "file" is a container for realted
- we name them to make them human-readable
- there are attributes to each file
- files are persistent
  - **what does it mean for files to be persistent?**

### Links
- files have multiple names
- each name is an alias

```
#include <unistd.h>
int link (const char *original_path, const char *new_path);

link ("foo", "bar"); // bar refers to file foo
unlink("bar"); // remove name bar
// if file is open by someone, it won't actually get deleted
// until all fd's to it are closed
```

### File attributes

```
#include <sys/stat.h>
int fstat(int filedes, struct stat *buf)
int stat (constchar *pathname, struct stat *buf)

// Structure contains file/directory info:
off_t st_size;    // file size
nlink_t st_nlink; // links 
mode_t st_mode;   // type + permission
time_t st_mtime;  // last modification time
fcntl // can also be used to set or get lower-level attrs

```
example: print when fle size changes
Write a program that monitors a given file every minute [sleep(60)] and if the file size has changed, it outputs the new size to stdout
```
void mon_size(char *fname) {
  struct stat sb;
  off_tpsize= -99;
  while (1) {
    sleep (60);
  }
}
```

### Filesystems
- directories are also files
  - have inode

file systems as a whole have
- files
- directories
- free disk sectors (known as free list)
- root directory

on-disk organization of filesystem
- inodefor root dirof filesystem“/” stored in well-known sector on the disk
- inodefor disk sector free-listalso stored in a well-known sector on the disk
- inodetable or file (inode#, sector)
- These are stored in the superblock



### Types of files
indicated by the first character when you run `ls -l`
| Symbol | File Type              |
| ------ | ---------------------- |
| -      | regular                |
| d      | directory              |
| c      | character special file |
| b      | block special file     |
| p      | pipe                   |
| s      | socket                 |
| l      | symbolic link          |

example: reading ls -l info
`drwx-xr-x 3 jon fac 4066 Nov 2 09:14 st`
`drwx-xr-x` = file type
`3` = # hard links
`4066` = allocation size

### File permissions
Operations (r, w, x): read, write, execute
Subjects (u: user/owner, g: group, o: others)
Users may belong to any number of groups(type groupsat the shell)

file permissions table
Number	Permission
| Number         | Permission Type |
| -------------- | --------------- |
| 0              | —               |
| 1              | –x              |
| 2              | -w-             |
| 3 (i.e. 2+1)   | -wx             |
| 4              | r–              |
| 5 (i.e. 4+1)   | r-x             |
| 6 (i.e. 4+2)   | rw-             |
| 7 (i.e. 4+2+1) | rwx             |

```
#include <sys/types.h>
#include <sys/stat.h>
int chmod(char *path, mode_tmode);
```
### IDs
- there are two types of ids
- real user id = user that actually initiated the process
- effective user id = user that system associates with the process for the purpose of protection
- sometimes we want the effective user id to be the file owner instead of the user for security reasons




### Masks
`creat ("test_file", 0777)`
we expect: `-rwxrwxrwx jon... my_file`
but instead we get: `-rw-------- ....... my_file`

why? 
- unix sets a default mask with your process to prevent against accidental exposure
- you can type `umask` to undo it

`creat("name",  PERM & (~mask));`!!! not sure what this does
to change the mask use: `mode_t umask(mode_t newmask);`

---

### What do we mean when we say directories? 
- abstraction of container for related files and other directories
- has the following components
  - name
  - location
  - contents
  - attributes
  - persistent

### Path names
- home directory
- current working directory
- relative file names !!! what are the advantages of these
- absolute file names

### Hard vs Soft links
hard links can't be made to directories or files in other file systems
soft links (sym links) allow a file/dir name to point to another file/dir name
`int symlink(const char *realname, const char *symname);`
`symlink(“/usr/jon/tmp1”,“/usr/bill/tmp2”)`

### Directory operations
- create/remove
- opendir/closedir
- readdir

#### create
example: make directory
```
#include <sys/stat.h>
int mkdir(constchar *pathname, mode_tmode);
mkdir(“tmp/dir1”, 0777);
```

#### remove
example: remove directory
```
int rmdir(constchar *pathname);
```
removes directory *only if* directory is empty

#### open/close directory
example: open directory and look at its contents
```
#include <dirent.h>
DIR *opendir(constchar *dirname);
struct dirent *readdir(DIR *dirptr);
int closedir(DIR *dirptr);
DIR *dp;
dp= opendir(“/tmp/dir1”);
struct dirent{
  ino_t d_ino;
  char d_name[NAMESIZE];
}
```

#### read
`#include <dirent.h>`
`struct dirent *readdir(DIR *dirptr);`

example: read directory
```
int my_ls(const char *name) {
  struct dirtent *d;
  DIR *dp;
  closedir (dp);
  return 1;
}
```


