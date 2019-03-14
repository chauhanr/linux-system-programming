# Process Termination 

## Terminating a process using exit() or _ exit() 
A process can terminate is two ways an abnormal way when the signal causes the process to terminate or a normal process using _ exit() 

```
#include <unistd.h> 

void _exit(int status); 

void exit(int status); 

``` 

The status parameter in the methods above defines the termination status of the process and is available to the parent process when it 
calls the wait() system call to get the status of the child processes. A zero exit status indicates that the process is successful where 
as a non zero status indicates that the process exited improperly. 

The _ exit() function is never called directly from the process but it is called indirectly from the exit() system call. The following 
steps are executed when exit() is called: 
1. exit handlers (func registered with atexit() or on_exit()) are called in the reverse order of their registration. 
2. The stdio buffers are flushed. 
3. the _ exit() system call is called with the status passed in the exit() method. 

When a c program is run the exit of the program is determined by the return value of the main method. If the main method returns a value n 
then the process terminates by calling exit(n) where as if there is no return n value then the program returns value that is dependent on 
the version of C version used. 

## Details of process termination 
During the normal termination of the process the following actions occur: 

* Open file descriptors, directory streams, message catalog descriptors, and conversion descriptors are closed. 
* As a consequences of closing the file descriptors, any locks on files will be released. 
* any attached system v shared memory segments are detached. 
* If this process is the controlling process for a terminal then a SIGHUP signal is sent to controlling terminal's foreground process. 
* Any POSIX semaphores in the calling process are closed. 
* Any POSIX message queues are closed using the mq_close(). 
* Any memory locks established using the mlock() or mlockall() are removed. 
* Any memory mappings established by the terminating process using mmap() are unmapped. 


 
## Exit Handlers 
There are a lot of times that clean up activities need to be performed when a process terminates. A good pattern is to have the an exit handler 
registered to the process which would be called when the processes exits using the exit() method. However this handler is not called when the process exits by calling the _ exit() or is terminated by a signal. 

There are two ways that exit handlers can be registered for a process. 

```
#include <stdlib.h> 

int atexit(void (*func)(void)); 

```

The function that is registered can neither have an input parameter nor a return variable. multiple exit handlers can be registered to a process and each can be called in the reverse order they are registered. 
The limitations of the the atexit() are: 
1. the function that is registered does not have any idea of the return or exit status of the process. 
2. It does not have any input variable or parameters which limits the handlers utility. 

The on_exit() handler works to overcome the afore mentioned limitations:

```
#define _BSD_SOURCE 
#include <stdlib.h> 

int on_exit(void (*func)(int, void *), void *arg); 

// the function can be like 
void func (int status, void *arg){
   // perform some action 
}

``` 





 
