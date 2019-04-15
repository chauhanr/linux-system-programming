# Process Execution 

## Executing a New Program : execve() 
The execve() system call loads a new program into process memory. During this call the old program
is discarded and data, heap and stack for the new process are created. 

Most often the execve() syscall follows the fork() command. Most of the syscalls that start new
processes all start with exec and are variation of the execve() and therefore they are called exec()
functions. 

```
#include <unistd.h> 

int execve(const char *pathname, char *const argv[], char *const envp[]); 
			-- never returns in the case of success but will give -1 on error

```
parametes in the execve() syscall are: 
* pathname - relative or absolute path to the executable of the program that needs to run 
* argv[] - array of argument variables that need to be passed to the program. 
* envp[] - array of env variables that need to be used by the syscall. 

The errorno that the execve can return are: 

1. EACESS - this occurs when the program that is given on the pathname does not have execute
   permission enabled or the elements of the path are not searchable. Alternatively file could also
   reside on file system that was mounted with a MS_NOEXEC flag. 
2. ENOENT - file does not exist at all 
3. ENOEXEC - the file is marked as executable but is in a format that Linux does not understand. 
4. ETXTBSY - file is open by another process for writing. 
5. E2BIG - The space required for the argument and env variables is more that the allowable limit. 


## Interpreter Scripts 
Shell scripts and interpreter scripts on the Linux operating system are ones which are not compiled
into machine code instead the code is interpreted line by line and executed. The script file starts
with a #! command followed by the absolute path to the interpreter. e.g. #! /bin/bash 

All of these scripts file will eventually be executed by the execve() method of Linux and the
arguments and optional parameters is passed to the execve() by the interpreter. 

## File Descriptors and exec() 
By default, all the file descriptors opened by the programs that call exec() remain open across the
exec() and are available for use by the new program. This feature is taken advantage of by the shell
to handle I/O redirection for the programs that executes. 

```
 ls /tmp > dir.txt 
```

in the command above shell performs the following steps: 
1. A fork() is performed to create child process that is also running a copy of the shell. 
2. child process opens dir.txt for output using file descriptor 1 (standard output) 
3. The child shell executes the ls program. The ls command will write the output to the standard
   output that is redirected to the dir.txt file. 

Not all command on the Linux OS will following the fork method there are a number of shell commands
that are not run by running fork(). This is done for 2 reasons: 
1. efficiency 
2. to have the indented side effect of the command on the shell itself. 

e.g. of built in shell commands include pwd, cd, echo, test, exec, exit, read, set etc. 



