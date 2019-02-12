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

## Signal Mask (blocking signal delivery)
The kernel keeps a signal mask (set of signals) whose delivery is currently blocked. If a signal that is blocked is sent to the process, the delivery to the process is delayed until it is unblocked by being removed from the process signal mask. (signal mask is actually a per thread attribute that can be set pthread_sigmask() method) 
A signal is added to the mask when: 
* When the signal handler is invoked, the signal that caused the invocation is added to the signal mask. 
* When the signal handler is established with sigaction() there is a provision to block not only the signal that caused it but also another set of signals. 
* sigprocmark() system call can be used at anytime to explicitly add signals to, remove signals from, the signal mask. 

```
int sigprocmark(int how, const sigset_t *set, sigset_t *oldset); 
                           Return 0 on success or -1 on error 
```

* how - determines the changes that sigprocmark() makes to the signal mask: 
	* SIG_BLOCK - this will add the signals in the set to the oldset (by way of union) and block all of them. 
	* SIG_UNBLOCK - signals pointed to in the set are removed from the oldset. This will unblock them, unblocking a system that is not currently blocked will not cause any error. 
	* SIG_SETMASK - signal set pointed to by the set is assigned to the signal mask. 

* if the set is sent in as NULL then sigprocmark() will return just the set of signals that are blocked and how will be ignored. 

However remember the signal that are unblockable i.e. SIGKILL or SIGSTOP are silently ignored. The following code shows how the sigprocmark works 

```
sigset_t blockSet, prevMask; 

// initialize a signal set to contain SIGINT 
signalempty(&blockSet); 
sigaddset(&blockSet, SIGINT); 

// block the SIGINT 
if (sigprocmark(SIG_BLOCK, &blockSet, &prevMark) == -1) 
	errExit("signprocmark1")
// code that should not be interrupted by SIGINT 

// restore previous signal mask, unblock SIGINT
if (sigprocmark(SIG_SETMASK, &prevMask, NULL) == -1) 
	errExit("sigprocmark2"); 

```

## Pending Signals 
If a signal is delivered to a process in between the time that the signal mask is set to block the signal, the signal is kept in pending state. Once the signal is unblocked it is delivered to the process. To determine the signals pending for a process, we call sigpending() 

```
int sigpending(sigset_t *set); 
	returns 0 on success and -1 on error 
```

**Signals are not Queued** The pending signals is a set, it only tells us whether it has occured or not but not the times that is has occured. Therefore the signal may have occured multiple times but is delivered just once when the signal is unblocked later. However runtime sgnals are queued. 

## Changing the disposition of the signal - sigaction() 
There is a signal() method that can be used to change the disposition but portability of the method is not very good so the sigaction is ued. 

```
int sigaction(int sig, const struct sigaction *act, struct sigaction *oldset); 

// sigaction struct is as follows: 
struct sigaction{
   void (*sa_handler)(int);  // address of handler 
   sigset_t sa_mask;  // signals blocked during invocation.  
   int sa_flags;      // flags controlling handler invocation. 
   void (*sa_restore)(void) 
}

```

