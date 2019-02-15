# Signal Advanced 

## Core Dump Files 
certian signals cause a process to create a core dump and terminate. the core dump file is the memory image of the process that at the time of termination of the process. This file can then be used by a debugger to have a look at the process state at the time the core dump is done. 
One of the signals that causes a core dump is the SIGQUIT signal. This can be achieved by typing the Cnt+\ which is the quit character. 

**core dump file may fail to create when:** 
1. process does not have permission to write the core dump file in the directory in which it is supposed to generate. 
2. a regular file with the same name is already present and writeable, but there are more than one hard links to it. 
3. directory to which the core dump file needs to be written too does not exist. 
4. the RLIMIT_CORE which specifies the size of the core dump file is set to 0 
5. The binary executable file that the process is executing does not have read permission enabled. this causes the core dump content not to be read from the proces.
6. file system on which the current binary resides is read only, if full or has run out of space. 
7. if the user executing the process is not the owner of the file then core dumps are not generated for reasons of security. 

**Naming the core dump file** 
the core dump file can be names based on the pattern mentioned under the file /proc/sys/kernel/core_pattern the following formatters can be used in naming the core file: 
* %c - add the core size file to the name 
* %e - executable filename (without prefix) 
* %g - real group id of dumped file. 
* %h - name of the host 
* %p - process id of the process being dumped. 
* %s - number of signal that terminated the process. 
* %t - time when the signal was issued (secs since epoch) 
* %u - real userid of dumped process 
* %% - the single % character 

## Special cases for delivery, disposition and handling

**SIGKILL and SIGSTOP** - the kill and the stop signal's default actions cannot be changed. The neither the signal() nor sigaction() can change the disposition of the two signals. there is an error thrown if an attempt to change the disposition is made. These signal cannot be blocked either. This is a deliberate design decision and these signals can always be used to kill run away processes. 

**SIGCONT** - this is the continue signal which can resume a process that has been previously halted by the SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU) 
When a SIGCONT is issued to a process it will resume the process even if it is blocked or configured to ignore SIGCONT. 

When a SIGCONT is given to a process which is stopped it stop signals are discarded. Conversely if a stop signal is delivered to a process, then an pending SIGCONT signals are discarded. 

## Interruptable and Uninterruptable Process Sleep states 
Many a times the kernel puts a process to sleep and the sleep is of two kinds: 
* TASK_INTERRUPTABLE: the process is waiting for some event. For example the process could be waiting for a terminal input from the user. The process can spend an arbitrary amount of time in this state as there is no guarantee when the user will send over the message.
* TASK_UNTERRUPTABLE: here the process is waiting to do a short but important task like writing to I/O channel. 

In the second kind of sleep state the signals like SIGKILL and SIGSTOP do not reach the process until it emerges from the state. Generally the task uninterruptable timeframes are so small the delay in delivery of signals to such processes is invisible. But if there is a hardware failure or kernel bug which hangs uninterruptable process in suspended state then it will be stuck there forever. In such a case only way is to restart the server. 

 

