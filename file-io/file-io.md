# File Input and Output 

File to document that File I/O in the Linux Operating System

## Atomicity and race condition 
All system calls on the operating system are guaranteed to run atomically. OS ensures that all steps of the system call will be run as a single unit without being interrupted by another process or thread.

The operations of open() and write() are subject to race conditions. There are flags that can help with dealing with race conditions like the O_APPEND which ensures that the file is appended to rather than seeking a location and then appending programatically. 

## File control operations fcntl() 
This system call does a wide range of operations on the file descriptor. 

``` 
int fcntl(int fd, int cmd, ...) 

/*  cmd indicates the command to run on the file descriptor and the ellipsis indicates the arguments. 
    also this system call ensures that the command you specify runs atomically. 
*/
```

One of the use cases for the use of fcntl() is to change the access mode of the file and this is relevant in 2 scenarios: 
* file was not opened by the calling programming, so the program has no control over the flags used in the open() call. 
* the file descriptor was obtained from a system call other than open() like a pipe or socket. 

```
flags = fcntl(fd, F_GETFL); 
if(flags == -1) 
    errExit("fcntl"); 
flags |= O_APPEND; 
if(fcntl(fd, F_SETFL, flags) == -1)
   errExit("fcntl"); 

```

## Relationship between file descriptors and open files 
There can be multiple file descriptors to one open file and many a times it is necessary. For this to happen the kernel has 3 data stuctures: 
* per process file descriptor - this olds reference to an open file descriptor for the current process. 
	* holds a set of flags controlling the file desc. 
	* a reference to the open file descriptor 
* system wide table of open file descriptions - this is a system wide opn file table that hold handles to all files open in the system. This table holds information like: 
	* current file offset 
	* status flags set at the time of opening the file. 
	* the access mode of the file. (read only, write-only etc.) 
	* settings related to signal driven i-o 
	* reference to the inode object.  
* file system i-node table - an inode signifies the presence of a file on the operating systems file system. It holds information like: 
	* file type (FIFO, regular, socket etc) and permissions 
	* a pointer to a list of locks on the file. 
	* various properties of the file including its size and timestamps relating to different types of files. 

![File Data structures](./images/file-data-structures.png)

* In process A descriptors 1 and 20 both refer to the same open file descriptor 23 and this can occur when we call the dup() or dup2() methods 
* Descriptor 2 of process A and B point to the same file descriptor 73. This scenario occurs when we have a fork() command being executed. 
* Descriptor 0 or process A and descriptor 3 in process B point to different opne file descriptors 0 and 86 but both these eventually map to one inode 1976. this scenario occurs when we call open() on same file from different processes. 


