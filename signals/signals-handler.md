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

**Standard Async Signal safe functions** 
A function is set to be async signal safe because it is either reenterant or it is not interrutable by signal handlers. There are a number of functions with is properly like chdir, rmdir, fork, and many many more. Therefore signal handlers should either be reenterant or they must call only async signal safe methods. 
Most of the string manipulations or functions that interact with stdio or stdout are not async signal save functions. 

**Global variables and sig_atomic_t type** 
Global variables in the C programs can be used by signals to indicate the change in state. But for them to work properly the data type of the global variable needs to be such that the operations on them must be atomic. Therefore for the purposes of an integer we use the sig_atomic_t type to ensure that the integer is atomic when the signal handler or the main program is chaning it simultaneously. 

## Methods to terminate the Signal handlers 
Simply returning from the signal handler is not always helpful or recommended specially in the case of hardware interrupts. therefore there are several ways of doing this: 
1. use the _ exit() method to get out of the handler which is different from the exit() method used in C language to get out of a program. The exit() method call will ensure that the stdio buffers are flushed before calling the exit() which may not be desireable in the case of signal handlers. 
2. use kill() or the raise() to send signal that kill the process 
3. perform a non local go to from the signal handlers. 
4. use the abort() function to terminate the process with a core dump. 

**Performing the non local jump from signal handlers** 
when a process is running on a shell and we press Control+C the SIGINT signal is emitted and the process uses a longjmp() is called to move from th process to the shell to receive the next command. 
However this is not a good approach because then the signal is issued and the proces responds to it to call the signal handler a signal mask is added to the process for the signal just received. longjump() method although is able to help us get out of the process back to the caller does not reset the process signal mask and this behavior is not consistent because in a BSD system the signal is reset where as in System V this may not be the case. therefore the longjmp() is not portable approach. 

To solve this there are a couple sigsetjmp() and siglongjmp() that make the pbehavior we described above portable.

**Using the abort() function** 
If the abort() method is called from a program a SIGABRT signal is issued and is guaranteed to terminate the process except in the case where a signal handler is specified and it ends without returning from the handler (non local go to method). 
In all other cases the abort() will end the process even if it fails the first time abort() resets the signal from SIGABRT to SIG_DFL and raises a second abort(). The abort also flushes all the stdio streams. 

## Handling a signal on an Alternate Stack: sigaltstack() 
As signal handler is a function it is allocated a stack in heap memory for it to do its job. However if the process is already running high memory usage and it as the limit of the heap such that calling a signal handler stack overflows the stack the process will fail. Example is that of SIGSEV signal which is issued when the last function call just overflows the limit of the heap. Because the SIGSEGV does not have space to run the process terminates without handling of the signal. In order to handle this issue the following is done: 
1. allocate a memory, called the alternate signal stack to be used for stack frames of signal handlers 
2. use the sigaltstack() system call to inform the kernel of the existance of such a stack. 
3. When establishing the signal handler, specify the SA_ONSTACK flag to inform the kernel that the stack for this signal handler must be on the alt stack that has been established. 

```
int sigaltstack(const stack_t *sigstack, stack_t *old_stack); 
	       // this method not only creates a sigstack if one is not created but returns the old stack if one is present. 
```

Kernel generally does not resize the alternate stack and if that too overflows then there are unintended consequences like writing of signal data to  memory locations that are assigned to other variables. 
