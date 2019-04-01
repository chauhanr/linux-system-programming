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

## waitpid() system call 
The wait() system call has limitations that waitpid() is designed to overcome. 

* If a parent process created multiple children, it is not possible to wait() for completion of a
  specific child; we can only wait for the next child that terminates. 
* If non of the children of a process have terminated wait() always blocks. It is perferable to
  sometimes to perform a non blocking wait so that if no children have terminated we can obtain
  immediate indication of this fact. 
* Using wait() only gives information about which children have terminated. It is not possible to
  be notified when a child stops due to a signal like SIGSTOP or SIGINT or when the child is
  resumed using signal like SIGCOUNT. 

```
#include <sys/wait.h> 

pid_t waitpid(pid_t pid, int *status, int options); 
                    returns process ID of child 0 or -1 in case of error
```
  
Since the waitpid() takes in input of pid therefore can wait on the pid with the following rules: 
* if pid is greater than 0, wait for the child with the specified process id. 
* if the pid is 0 then wait for any children in the same process group as the caller (parent) 
* if pid is less than -1 wait for any child whose process group is identifier equals the absolute
  value of the pid specified. 
* if pid equals -1 wait for any child. this is same as the wait() system call. 

The options argument in the waitpid system call is a bit mask that can include (or) zero or more of
the following flags: 

* WUNTRACED - in addition to returing information about terminated children also returning
  information when the child is stopped signal. 
* WCONTINUED - Also returns status information about the stopped children that have been resumed by
  delivery of signal SIGCONT 
* WNOHANG - If no child specified by pid has yet changed state, then return immediately, instead of
  blocking. In this case, the return value of the waitpid is 0. If the process on which waitpid() is
  called has no chlidren then the syscall returns error number. 


## Process Termination from Signal Handlers 
Some signals terminate the process by default. In such circumstances if we need to perform some
clean up before the process ends and then terminate the process. If the child needs to inform the
parent that it has terminated because of a signal, the child must disestablish itself and then raise
same signal once again, which will terminate the signal once again. 

```
void handler(int sig) {
    /* perform clean up */ 

    signal(sig, SIG_DLF);  /* disestablish the signal */ 
    raise(sig);            /* raise the signal again*/ 
}
```

## wait3() and wait4() 
These system calls do the same thing as waitpid() except that they also return resource usage as
well. The information for usage includes: 
* amount of CPU time used by the process 
* memory management information 

However the wait3 and wait4 usage is limited because of a lack of standardization. 



