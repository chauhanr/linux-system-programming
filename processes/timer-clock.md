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


