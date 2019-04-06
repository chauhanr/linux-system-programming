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


