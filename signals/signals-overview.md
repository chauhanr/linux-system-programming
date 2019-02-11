# Signals Overview 

## Concepts
Signals are notifications to a process that an event has occured. There are two types of signals that we will discuss 
1. standard 
2. runtime. 

A signal is delivered to a process as soon as the process gets into execution mode until then the signal is kept blocked or 
if the process is already in execution mode then the signal is delivered immediately. 

When signals are received one of the following default action takes place: 
1. signal is ignored by the kernel 
2. process is terminated 
3. core dump file is generated and process is terminated. 
4. process is suspended 
5. execution of process is resumed

All above actions are default ones and they can be changed by the program by chaning the disposition of the signal: 
* the default action should occur 
* the signal should be ignored - this can be used for actions that would terminate the process. 
* a signal handler is called. 

## Signal Types and Default Actions 
There are 31 standard signals and the few important or popular ones are: 
* SIGABRT - process is sent this signal when the abort() function is called. This ends the process with the core dump
* SIGBUS - this signal is emmitted when we have memory access issues. 
* SIGCHLD - this signal is sent to the parent processes when an event to the child process occurs example the suspension. 
* SIGCONT - this occurs to restart an already suspended process. 
* SIGHUP - this signifies a terminal hang up 
* SIGINT - this represents the terminal interrupt character (ctnl+c)
* SIGKILL - a sure kill signal that can neither be blocked, ignored or intercepted by a handler. 
* SIGSTOP - this is a sure stop signal that cannot be blocked, ignored or caught by handler. 
* SIGTERM - this signal also terminates a process but before it does so it will call a handler function which will try and clear
  resources that the processes may be holding. 

Note that the signal numbers are the same on most architecture or Linux distributions. 

## Changing the signal disposition 
This means that the signals default behavior can change. We can register a handler that we define to the signal. 

```
#include <signal.h> 

void ( *signal(int sig, void (*handler)(int))) (int) 
```

The signal method will take the integer that signifies the signal and then the pointer to the handler func to replace the default action. 
On success the method will return the previous signal disposition and SIG_ERR in case of failure. 

Instead of passing a handler we could also pass one of the following values to the signal()
* SIG_DFL - this signifies the default handler or behavior this is done when the default behavior to the signal has to be reset. 
* SIG_IGN - this signifies to the kernel that the signal should be ignored altogether. 


## Signal Handlers 
signal handlers are functions that are called when specified signal is delivered to the process. 

```
static void signalHandler(int sig) {
   static int count = 0; 
   if (sig == SIGINT){
     count++
     printf("Caught SIGINT %d \n", count); 
     return; 
   }
   printf("Caught SIGQUIT - that's all\n"); 
   exit(EXIT_SUCCESS) "
}

int main(int argc, char *argv[]){
    // registering multiple handlers 
    if (signal(SIGINT, signalHandler) == SIG_ERR) 
       errExit("signal") 
    if (signal(SIGQUIT, signalHandler) == SIG_ERR) 
       errExit("signal")

   for(;;)
      pause(); 
}
```
For the code above both the SIGINT and SIGQUIT will be handled by the same handler and the program will only exit when 
SIGOUT is received. 

## Sending Signals 
The kill() method is used to send signals ot other processes from a program. 

```
int kill(pid_t pid, int sig); 
	      return 0 on success or -1 on error 
```

The call of the signal method is handled as follows: 
* If the pid sent to the method is > 0 then the signal is sent to the process with the pid specified. 
* if pid == 0 then signal is sent to all processes that are in the same process group as the current process. 
* if pid < -1 it is sent to all the process in the process group whose ID equals the absolute value of pid. 
* if pid == 1 then the signal is sent to every process that the calling process has permission to send a signal too. However the process will not send the signal to itself or init in this case. 

The permission rules for sending the kill() are as follows: 
1. A priviledged process (CAP_KILL) can send a message to all processes. 
2. init process (pid==1) which runs as user and group root, It can only send signal out for which it has handlers instaled. this is to prevent accidental killing of the init process. 
3. An unpriviledged process can send a signal to another process if the effective user id of the sending process is the same as the effective user id for the receiving process. 
4. SIGCONT - this is a special signal that any process can send to another process that belong to the same session, regardless of user id check. 

## Checking for the existance of a Process 
Here we look at different ways in which we can determine if other processes are present or not. 
* kil() - if we send the kill signal with the sig argument as 0 (which signifies a null signal) then no signal is sent to the speficied pid, kernel just checks to see if the process with pid can be sent a signal or not. 
	* if the kill() fails with a ESRCH error means that process with the pid cannot be searched. 
	* if the kill() fails with a EPERM error means that process was present but we donot have permissions on it 
	* if kill() succeeds then we know that the process exists and we have permissions to signal it. 
* Wait() can be called only on the child processes of the current process that is trying to determine the existence of the process. 
* Semaphores and file locks- If a process that holds a semaphore or lock continuously and we are able to get the lock means that the process has terminated. 
* IPC channels such as pipes and FIFO's 
* The /proc/PID interface - if we can call a stat() call on the file /proc/PID/${PID} then we know that the process exists. 

## Other ways to sending signals raise() and killpg() 
**raise()**
Sometimes when it is essential for a process to signal itself the raise() function can perform the task. 
In a single thread program, a call to rais() is equivalent to the following call:
```
    kill(getpid(), sig);  
``` 
in a multi threaded mode the call to raise() is equilvalent to 
```
     pthread_kill(pthread_self(), sig) 
```
In the case of multithreaded process the raise() method will be sent to the process in general and can be forwarded to any of the threads of the process. 

The raise() can return a non zero value as error EINVAL which signifies invalid signal, If we send any one of the SIGxxx signals then we do not need to check for an error. The sginal is immediately handled by the kernel. 

**killpg()** - will send a signal to all processes in the process group 
```
#include <signal.h> 

int killpg(pid_t pgrp, int sig) 
```
This method is equivalent to calling  
```
    kill(-pgrp, sig); 
```
if pgrp is set to 0 then the signal is sent to all process in the same process group as the caller. 

## Signal Sets 
Many signal related system calls need to have a notion of a group of signals. For example sigaction() and sigprocmask() or sigpending() methods all return a set of signals that are relevant to the context. Therefore Linux kernel has a sigset_t type defined. 
In order to initialize the set we can use the following methods: 
```
#include <signal.h> 
int sigemptyset(sigset_t * s);  // this will initialize an empty set. 
int sigfillset(sigset_t * s); // this will initialize the set with all signals 

# other methods include 
int sigaddset(sigset_t * s, int sig); 
int sigdelset(sigset_t * s, int sig);  

int sigmember(const sigset_t * s, int sig); // checks if the sig is a member of the set or not.  
```
There are other methods too that perform set operations on the sigset_t like the sigandset() and sigorset() 




