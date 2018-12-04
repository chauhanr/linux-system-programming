# System

Linux and Unix systems impose limits to various system features and resources. 

List of SUSv3 limits 

| Number of limits | Min value | sysconf()/pathconf() name | Description | 
| -----------------|:---------:|:-------------------------:|:-----------|
| ARG_MAX          | 4096      | _SC_ARG_MAX               |             |
| none             | none      | _SC_CLK_TCK               | unit of measurement of time | 
| LOGIN_NAME_MAX   | 9         | _SC_LOGIN_NAME_MAX        | maximum size of login name | 
| OPEN_MAX         | 20        | _SC_OPEN_MAX              | maximum number of fd that a process can open | 
| NGROUPS_MAX      | 8         | _SC_NGROUPS_MAX           | maximum number of supplementary groups of which a process can be a group of | 
| none 		   | 1	       | _SC_PAGESIZE		   | size of the virtual memory page | 
| RTSIG_MAX        | 8         | _SC_RTSIG_MAX             | maximum number of distinct realtime signals | 
| SIGQUEUE_MAX     | 32        | _SC_SIGQUEUE_MAX          | maximum number of queued realtime signals | 
| STREAM_MAX       | 8 	       | _SC_STREAM_MAX            | maximum number of studio streams opened at a time |
| NAME_MAX         | 14        | _PC_NAME_MAX              | maximum number of bytes in pathname | 
| PIPE_BUF         | 512       | _PC_PIPE_BUF              | maximum number of bytes that can be written atomically| 
| PATH_MAX         | 256       | _PC_PATH_MAX              | maximum number of bytes in filename | 

 The sysconf() function can be called programmatically from C language. 

```
#include <unistd.h> 
long sysconf(int name);   /** returns limit value for the name specified returns -1 in case none is found.*/
``` 
In golang there is a github library that can be used to get the sysconf values across various linux variants.
[go-sysconf](https://github.com/tklauser/go-sysconf) 


In order to get file realted limits at run time we can use pathconf() and fpathconf() 
```
#include <unistd.h> 

long pathconf(const char *pathname, int name); 
long fpathconf(int fd, int name); 
```

## System options 
These are optional features that the Linux implementation may support 
* _POSIX_ASYNCHRONOUS_IO (_SC_ASYNCHRONOUS_IO)  -  async io 
* _POSIX_CHOWN_RESTRITED (_PC_CHOWN_RESTRICTED) - open previledged processes can use chown. 
* _POSIX_JOB_CONTROL (_SC_JOB_CONTROL)- job control. 
* _POSIX_MESSAGE_PASSING (_SC_MESSAGE_PASSING) - posix message queue support. 
* _POSIX_PRIORITY_SCHEDULING (_SC_PRIORITY_SCHEDULING) - process scheduling 
* _POSIX_THREADS (_SC_THREADS) - posix threads. 
* _XOPEN_UNIX (_SC_XOPEN_UNIX) - xsi extension is supported. 


