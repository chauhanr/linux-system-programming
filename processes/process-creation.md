# Process Creation 

## Overview 
Lets have a look at exec(), fork(), wait() and execve()

* fork() - system call allows one process (parent) to create a new process (child). This is done by
  making the new child process (almost) an exact duplicate of the parent. The child process will
  contain copies of the parent heap and memory stack. 
* exit() - The exit library function terminates a process and releases all resources (memory, data,
  open fd) and makes them available for reallocation by the kernel. The parent process can get that
  status used to exit the process by calling the wait() system call.
* wait() - this system call has two purposes: 
	* first if a child of this process has not yet terminated by calling exit() then the current
	  process waits for the child process to exit. 
	* the termination status of the child is returned as the status agrument to the system call. 
* execve(pathname, argv, envp) - the execve system call loads a new program (at pathname, with
  argument variables argv and environment list envp) into a process's memory. The existing programs
  text is discarded and a new process stack and heap are created with the details of the new
  program. Diagram below shows how the system calls mentioned above operate 

![syscall-pc](images/syscall-overview.jpg]


## Create a new process - fork() 
The fork() system call creates a new process, the child, which is almost exactly duplicate of the
calling process, the parent. 

```
#include <unistd.h> 

pid_t fork();
	in the parent process returns process id when successful and -1 on error where as in the
	child process it always returns 0 
```
The key point to note about the fork method is that when it completes its work there are 2 processes
that exist. In each of the process that the fork system call produces the work starts from the point
where fork() returns. In the parent's stack the fork returns the pid of new child process as it
needs to mantain its children where as the child can get its pid using the getPid() method. 

Once a child is created it has its own data, stack and heap and any writes or changes to them will
not effect the parent. 

The fork might fail for various reasons but most common one is that the RLIMIT_NPROC which is the
limit to the number of processes a user id can spwan is exceeded. 

```
#Pattern for calling the fork

pid_t childPid;  
switch (childPid == fork()) {
    case -1: 
    	/* handle error as fork has failed */
    case 0: 
    	/* child of successful fork - so perform child related tasks*/
    default: 
    	/*perform the actions of the parent. */
	sleep(3); 
	break;
}
```
Another important aspect to note here is that the frok() after reutrn is indeterminate about which
process to run (child or parent) once it returns. Therefore some programs us sleep() in the parent
to enable the child to get the CPU before the parent. But this is not a fool proof way of ensuring
this consdition. 

**File sharing between parent and child** 

