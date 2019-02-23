# Timer and Sleeping 

Timer allows a process to schedule a notification for itself sometime in the future. Sleep on the other hand allows a process to suspend execution for a period of time. 

## Interval Timers 
A setitimer() syscall sets an interval timer, which is a timer that expires at a future point in time and (optinally) at regular intervals beyond that. 

```
#incudle <sys/time.h> 

int setitimer(int which, const struct itimeinterval *new_value, const struct itimeinterval *old_value); 
			// return 0 in case of succes and -1 in case of error

// itime inteval structure 
struct itimeinterval{
   struct timeval it_interval;  /*Interval for perodic timer*/
   struct timeval it_value;    /*Current value (time until next expiration)*/
}

struct timeval{
  time_t      tv_sec;   // seconds
  suseconds_t tv_usec;  // micro seconds
}

```
* the it_value under new_time substructure specifies the delay until the timer expires, the it_interval signifies whether the current timer will be a periodic one or not. 
* if both fields under the it_interval substructure are set to 0, then the timer is useable only once and expires once at the time specified by it_value. 
* however if any of the fields of the it_interval structure as a non zero value then the timer first sends a signal at the it_value exipration followed periodically based on the time set in the it_interval. 
* if a timer has to be disabled then the setitimer() needs to be called with both the new_value.it_value fields being set to 0. 
* the old_value structure signifies the old value of the time. setitimer() can be used to get this value by sending a non NULL value to the call and where as if you are not interested in the old value then just send a NULL. 

As timer progresses, it counts down from the initial value (it_value) towards 0. When the timer reaches 0, the corresponding signal is sent to the process and if the it_interval is non zero then the timer it_value is reloaded and the timer starts again. 

The getitimer() system call can be used to just read about the timer, but to change the value we must use the setitimer. 

There are 3 different type of timers that can be specified by the setitimer() syscall: 
1. ITIMER_REAL - Create a timer that counts down in real time (wall clock time) and when the time expiries it will send a SIGALRM signal. 
2. ITIMER_VIRTUAL - create a timer that counts down the time in virutal time. when this timer expires then a SIGVTALRM signal is sent. 
3. ITIMER_PROF - create a profiling timer (this will count process time both in user and kernel mode of CPU time). A SIGPROF signal is sent when this timer exipres. 

**Simple timer interface: alarm()** 
If you donot want the complexity of the interval timer then you can use the alarm() interface to set a timer after which the timer expires there is no option of subsequent timers. 

```
unsigned int alarm(unsigned int seconds) 
	// always succeeds and returns the number of seconds that is left on an already exiting timer and 0 if none is set. 
``` 

* When alarm() is called then all previous set timers on the process are cancelled. 
* In order to cancel all timers we can call alarm(0) 
* After alarm() expires there is a SIGALRM signal that is emitted. 

## Schedulig and Accuracy of Timers 
Depending on system load and scheduling of processes, a process my not schduled to run until some short time after actual expiration of the timer. Interval timers are not subject to creeping errors. 
Although the timeval structure supports time defintion upto the milli second precision, but the accuracy of the timer signal is dependent on the software clock. If the clock does not support the granularity of the time set then the timeval is rounded off to the next value. for example a timeval of 19100 micrseconds will genrally be rounded to 20000 ms. 

**High resoultion timers** 
Starting from Kernel 2.6.21 the dependency of the timer on the kernel jiffy has been removed. The timer can be as accurate as the time accuracy of the underlying hardware. 

## Setting Timeouts on Blocking Operations 
In order to have a time limit on the blocking operations we can use the following pattern: 

1. Call the sigaction() with SA_RESTART not set so that the system calls are not restarted and SIGALRM as handler. 
2. Call the alarm() or sigitimer() to establish a timer 
3. Make the blocking call. 
4. After system call returns or signal is received call the alarm() or sigitimer() once more to disable the timer in case the blocking operations has returned successfully
5. Check to see whether the blocking system call failed with errno set to EINTR

Following is snippet 
```
sa.sa_flags >= (argc>2) ? SA_RESTART:0;
sigempty(&sa.sa_mask);

sa.sa_handler = handler; 
if(sigaction(SIGALRM, &sa, NULL == -1)
	errExit("sigaction")

alarm((argc>1) ? getInt(argv[1], GN_NONNEG, "num-secs") : 10); 
numRead = read(STDIN_FILENO, buf, BUF_SIZE-1); 

alarm(0)  /* disable the alram becuse we do not know if the blokcing call has returned or a signal has been raised.*/
// finally work withe the data returned. 
```

## Suspending Execution for Fixed Interval (sleeping) 
The best way to suspend a process for a fixed period of time sleep() is the best option. 

**Low resolution sleep** 
sleep() suspends the execution of a process for a number of seconds mentioned in the input or a signal is caught(interrupting the call) 

```
unsigned int sleep( unsigned int seconds); 
```
the sleep() method will either returns a 0 if it returns successfully meaning that the sleep timer has expired or it will return number of seconds still remaining in the sleep call if the process has been interrupted. 
Because in most general Linux implementations sleep() is implemented using the alarm() and setitimer() methods it is advisible not to use sleep with these methods closely. 

**High resolution sleeping: nanosleep** 
nanosleep() is a function that does exactly what the sleep function does but it gives higher resolution on the time to sleep. 

```
int nanosleep(const struct timespec *request, struct timespec *remain); 
	return 0 when successfully completes the sleep() or returns -1 when errors out or interrupted. 

struct timespec {
	time_t tv_sec; 
	time_t tv_nsec; // can take value 0 to 999,999,999 
}
``` 
Another advantage of using nanosleep is that it is not implemented using signal handlers and alarm() so it can be used safely with alarm() and sigitime() methods. 

A limit to the use or accuracy of the nanosleep() is that it is dependent on the accuracy of the software clock and the granularity that the software clock supports. If the software clocks makes rounding errors (precision) then the nanosleep() method would not be able to give the exact accurate level of precision. 

From Linux 2.6 a new method clock_nanosleep() has been introduced that uses the TIMER_ABSTIME which bypasses the earlier mentioned limitation and refers to the hardware clock. 

## POSIX clocks 
POSIX clocks provide and API for accessing clocks that measure the time is nanoseconds.

**Retriving the Clock value and types of POSIX clocks**
The following are the supported methods used to get the clocks: 

```
#include<time.h> 

int clock_gettime(clockid_t clockid, struct timespec *tp); 
int clock_getres(clockid_t clockid, struct timespec *res); 
``` 
The clock_getime() will return timespec that specfies the time of the clock where are clock_getres() gives the resolution of the clock. 
There are 5 types of clock types in POSIX clocks: 
1. CLOCK_REALTIME - settable time which is system wide real-time clock. 
2. CLOCK_MONOTONIC - Nonsettable monotonic clock which gives the time since system start up and this cannot be changed. 
3. CLOCK_PROCESS_CPUTIME_ID - per process CPU time that the process has consumed. 
4. CLOCK_THREAD_CPUTIME_ID - per thread CPU time clock 
5. CLOCK_MONOTONIC_RAW - (since kernel 2.6.12) this is similar to MONOTONIC but has access to pure hardware based time. 

Methods that return the clock id per thread and process. 
```
int clock_getcpuclockid(pid_t pid, clockid_t *clockid); 

int pthread_getclockid(pthread_t thread, clock_id *clockid); 
```

**Improved high resolution sleeping: clock_nanosleep()** 
This method is similar to nanosleep() method explained above. There are a few differences
```
#include<time.h> 

int clock_nanosleep(clockid_t clockid, int flags, const struct timespec *request, struct timespec *remain); 
```
* the request and remain time specs are similar to nanosleep(). However the flag passed to the method can either consider relative time flag is set to 0 or it can consider absolute time TIMER_ABSTIME 
* In the case where nanosleep() is used as a method of sleeping and waking of the process is controlled by a loop then depending on the signal frequency there may be innacurracies that lead to issues. In the case of clock_nanosleep() we can avoid this by first getting the clockid of the current time adding appropriate timer to the time and then calling the clock_nanosleep() using absolute timer flag. 
* Unlike nanosleep the cock_nanosleep() can be involved with various clock types like REALTIME, ABSOLUTE TIME, CLOCK CPU TIME.

```
/* Retrieve the clock time */
if( clock_gettime(CLOCK_REALTIME, &request) == -1) 
	errExit("clock_realtime");

request.tv_sec +=20 // adding 20 sec 

s = clock_nanosleep(CLOCK_REALTIME, TIMER_ABSTIME, &request, NULL); 
if (s!=0) {
  if (s == EINTR){
    printf("Interrupted by signal handler"); 
  }else{
    errExitEN(s, "clock_nanosleep"); 
  }
}
```


