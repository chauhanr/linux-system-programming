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



