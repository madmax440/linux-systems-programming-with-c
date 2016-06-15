# timers

A timer counts down to 0 from a specific value. In operating systems, the timer upon expiry, allows a program to perform specific actions. There are two timers, specifically one shot and periodic.

A one shot timer runs only once. A periodic timer repeats itself upon every expiration.


## alarm



`alarm()` arranges for a `SIGALRM` signal to be delivered to the calling process. The `alarm` can be thought of as one shot timer.

Below example provides a oneshot timer implementation with the `alarm`.

```c
#include <stdio.h>
#include <unistd.h>
#include <time.h>
#include <signal.h>

void sigalrm_handler(int sig)
{
    printf("Alarm signal\n");
}

int main(int argc, char **argv)
{
    signal(SIGALRM, sigalrm_handler);
    alarm(2);
    while(1);
    return 0;
}
```


## setitimer


The `setitimer` API provides either a one shot or an interval timer. When the timer fires, the OS activates the SIGALRM signal for each expiry. Before the `setitimer` we register a signal handler for the SIGALRM. The `setitimer` thus invokes the signal handler indirectly upon each expiry.

the `setitimer` API prototype is as follows.

```c
int setitimer(int which,
               const struct itimerval *cur,
               struct itimerval *old);
```

the `which` argument takes one of the three aguments which are `ITIMER_REAL`, `ITIMER_VIRTUAL` and `ITIMER_PROF`. Generally, for a system timer we use `ITIMER_REAL`.

The second argument is the structure pointer of type `struct itimerval`. The structure contains the following.

```c
struct itimerval {
    struct timeval it_interval;
    struct timeval it_value;
};

struct timeval {
    time_t tv_sec;
    suseconds_t tv_usec;
};

```

The structure contains an it_value member describing the initial value for the timer and an it_interval member that is used as a repeatable value. The timer is initialized with the it_value and when the timer expires, the it_interval is loaded as a next expiry value and is repeated again and again.

for ex:

```c
struct itimerval it = {
    .it_interval.tv_sec = 1;
    .it_interval.tv_usec = 0;
    .it_value.tv_sec = 2;
    .it_value.tv_usec = 0;
};

```

The it_value is initialized to 2 seconds and so after the 2 secs the timer expires and calls the signal handler that is registered. upon the expiry, the it_interval is loaded into the timer as the new expiry time that is 1 sec. At every expiry the 1 sec timeout value is loaded back as a next triggering timeout.

The `setitimer` behaves as a one shot timer when the it_interval argument is 0.

Below is the example of the `setitimer`.


```c
#include <stdio.h>
#include <signal.h>
#include <string.h>
#include <unistd.h>
#include <time.h>
#include <sys/time.h>

void signal_handler(int sig)
{
    struct timeval tv;

    gettimeofday(&tv, 0);
    printf("timer interrupt %ld.%ld\n", tv.tv_sec, tv.tv_usec / 1000);
}

int main(int argc, char **argv)
{
    int ret;
    int timerval;
    int repeat = 0;

    struct sigaction new_sig;

    if (argc != 3) {
        fprintf(stderr, "%s <msec timeout value> <repeat enable (1 / 0)>\n", argv[0]);
        return -1;
    }

    timerval = atoi(argv[1]);
    repeat = atoi(argv[2]);

    memset(&new_sig, 0, sizeof(new_sig));

    new_sig.sa_handler = signal_handler;

    ret = sigaction(SIGALRM, &new_sig, NULL);
    if (ret != 0) {
        fprintf(stderr, "failed to setup SIGACTION for SIGALRM\n");
        return -1;
    }

    if (timerval != 0) {
        struct itimerval iv;

        memset(&iv, 0, sizeof(iv));

        iv.it_value.tv_sec = 0;
        iv.it_value.tv_usec = timerval * 1000;
        if (repeat) {
            iv.it_interval.tv_sec = 0;
            iv.it_interval.tv_usec = timerval * 1000;
        }

        ret = setitimer(ITIMER_REAL, &iv, NULL);
        if (ret != 0) {
            perror("setitimer");
            fprintf(stderr, "failed to setitimer\n");
            return -1;
        }
    }

    while (1);

    return 0;
}
```

The program first registers the SIGALRM signal via `sigaction` and registers the `setitimer` with the given input values. The periodicity of the timeout is controlled via the repeat argument.


## timer_create



## timerfd_create
