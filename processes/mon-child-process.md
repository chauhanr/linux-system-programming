# Monitoring Chid Processes 

In many applications where a parent creates child processes, it is useful for the parent to be able to monitor the child to find out 
when and how they terminated. This can be done using the wait() and related system calls. 

## wait() system call 
The wait() method system call waits for one of the children of the calling process to terminate and return the termination status of the
child in the buffer pointed to by the status. 

```
#include <sys/wait.h> 

pid_t wait(int *status) 
		return process id of the process terminated or -1 in case of error 

```

The wait system call does the following: 
1. If no (previously unwaited for) child of the calling process has yet terminated, the call blocks until one of the child terminates. If the child has already terminated by the time the wait() call is executed the wait() returns immediately. 
2. if the status is not NULL, information about how the child terminated is returned is the integer to wait status points. 
3. the kernel adds the process CPU times and resources usage statistics to running totals for all children of this parent process. 
4. As it function result, wait() returns the process id of the child that has terminated. 

On error wait() returns -1. One possible error is that the calling process has no children, which is indicated by the errno value ECHILD. So we can use the following loop to check if all process of a parent have terminated. 

```
while ((childPid == wait(NULL)) != -1) 
	continue; 
if (errno != ECHILD) 
	errExit("wait") 
```


  
