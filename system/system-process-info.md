# System and Process Information 

## /proc file system 
The proc file system is a virtual file system that was built to answer questions like: 
* How many processes are running on the system and who owns them. 
* What files a process have open. 
* which files have locks on them and which processes hold these locks? 
* which sockets are being used by the system. 

### Obtain the process information  /proc/{pid} 
This folder contains all the information you will need to know about the process. e.g. /proc/1 will give the information about the init process that stated the Linux system. 
The following folders can be seem under the /proc/{pid} folder. 

|File    |  Description                                                 | 
|--------| :------------------------------------------------------------|
|cmdline | command line arguments that were used when process started.  | 
|cwd     | symbolic links to the current working directory              | 
|environ | environment variables <key=value> pair.                      | 
| exe    | symbolic link to the file being executed.                    | 
| fd     | dire that holds symbolic links to all the files opened by the process.     | 
| maps   | memory maping | 
| mem    | process virtual information | 
| mounts | mount point of this process | 
| root   | symbolic link to the root director. 
| status | various informations (process id, credentials, memory, signals | 
| task   | contains sub directory of all threads that run in the process | 

**/proc/1/status** - This give the following info 
```
Name: systemd 
State: S (Sleeping) 
Tgid:  1                       /*thread group id*/
Pid:   1                        /* this gives the rpocess id*/

TracePid:  0                   /* the id of the tracing prorams*/
FDSize : 256                   /*total number of descriptor slots currently available*/
VmPeaK: 852k                     /* peak virtual memory size of this process */ 
... 
```

**/proc/1/fd** - this is a folder that gives a list of all file descriptors that a process has opened or is accessing. 

**/proc/{pid}/task** - this list gives information about a process that is not easy to get. the task correspond to threads in the proecess. 


 
