# Signal Handlers 

## Designing the Signal Handlers 
Signal handlers should be designed such that they do not cause race conditions and exit as quickly and cleanly as they can. The common patterns
for the handlers are: 
* The signal handler generally sets global flags, that other main methods refer to perioidically, and exit. There is however a situation that the main method does not read any global flags and might be stuck for an I/O input, in such a case the handler can write a single byte to the i/o device the main program is reading. 
* The signal handler performs some kind of clean up and either terminates the process and returns the control back to the main program. 

### Signal are not Queued 
as was mentioned in the overview of signals, signals can occur multiple times. If a signal handler is executing all signals of this handlers that occur during that time are marked as pending and only one instance of it is kept. Keeping this is mind the signal handlers need to be coded such that it handles the case where multiple signals may have occured between the time the signal handler was running. 

### Reenterant and Async Signal Safe function
 
**Reenternant and non reenternat functions**  
In the scenario of an operating system which is generally a multi threaded one a process may be running mutliple threads and the signal handlers that run will by another thread within that process. Because of the concurrent nature of operations with threads the concept of reenterant or non reenterant functions become important. 
A function is called reenterant if it only works on local variables and does not update global variables/flags or central data structures or resources. Where as a non reentrant function is one where it can update a variable that is not fully owned by the function. Therefore it becomes extremely important when we write signal handlers we carefully think befor using non reentrant functions. 

Examples of such functions: 
1. malloc() and free() method family that handle memory allocation are non reenterant. 
2. functions that return values that are statically allocated like crypt(), getpwnam(), gethostbyname() etc. 
3. functions that use statically allocated data structures like the printf() and scanf() that hold reference to stdin and stdout. If these are used in the handler and also in the main program there can be situation handlers simultaneouly might cause issue when writing on the console. 

```
# file name is nonreenterant.c
static char *str2; 
static int handled=0;

static void handler(int sig) {
  crypt(str2, "xx")
  handled++
}

int main(int argc, char *argv[]){
    .. 
    str2 = argv[2];
    cr1 = strdup(crypt(argv[1], "xx");

    ... // set up the signal 
    if sigaction(SIGINT, &sa, NULL) == -1) 
	errExit("sigaction")
    
    for (callNum=1, mismatch=0; ; callNum++){
       if(strcmp(crypt(argv[1], "xx"), cr1) != 0) {
           mismatch++; 
           printf('Mismatch on call %d (mismatched=%d, handled=%d)\n', callNum, mismatch, handled);
        }
    }
}

```


