# Get Linux Process Start Time

## Steps

1. read `/proc/%d/stat` to get the process `starttime`(it is clock ticks since system boot).
2. divide `starttime` by sysconf(_SC_CLK_TCK) to get process startup seconds since system boot.
3. read `/proc/stat` to get system boot time.
4. add two times tother.

## Commands

```bash
# lookup the process started time
ps -o cmd,lstart -p <PID>

# example
ps -o cmd,lstart -p 1
CMD                                          STARTED
/usr/lib/systemd/systemd -- Mon Oct 11 16:59:57 2021
```

## Codes

```c
#include <time.h>
#include <stdio.h>
#include <stdint.h>
#include <unistd.h>


uint32_t convertStartTime(uint64_t startTime)
{
    static uint32_t btime = 0;
    static uint32_t clockTick = 0;
    uint32_t        calc;

    FILE *fp = NULL;

    if (btime == 0) {/* get system startup time from /proc/stat */
        fp = fopen("/proc/stat", "r");
        if (fp) {
            char buf[256];
            while (fgets(buf, sizeof(buf) -1, fp)) {
                if (sscanf(buf, "btime %u", &btime) == 1) {
                    break;
                }
             }
             fclose(fp);
         }
    }

    if (clockTick == 0) {
        clockTick = sysconf(_SC_CLK_TCK);
    }

    calc = (uint32_t)(startTime / clockTick);
    return btime + calc;
}

uint32_t getProcessStatTime(pid_t pid)
{
    FILE *fp = NULL;
    int   ppid, tmpint, ret;
    char  name[64] = {0};
    char  state;
    long  tmplong;
    unsigned long tmpulong;
    unsigned long long starttime;

    snprintf(name, 63, "/proc/%d/stat", pid);
    fp = fopen(name, "r");
    if (NULL == fp) {
        return -3;
    }
    ret = fscanf(fp, "%d (%[^)]) %c %d %d %d %d %d " //(1)pid,(2)name,(3)state,(4)ppid,(5)pgrp,(6)session,(7)tty_nr,(8)tpgid
             "%lu %lu %lu %lu %lu %lu %lu %lu " //(9)flags,(10)minflt,(11)cminflt,(12)majflt,(13)cmajflt,(14)utime,(15)stime,(16)cutime
             "%lu %ld %ld %ld %ld %llu", //(17)cstime, (18)priority, (19)nice, (20)num_threads,(21)itrealvalue, (22)starttime
             &pid, name, &state, &ppid, &tmpint, &tmpint, &tmpint, &tmpint,
             &tmpulong, &tmpulong, &tmpulong, &tmpulong, &tmpulong, &tmpulong, &tmpulong, &tmpulong,
             &tmpulong, &tmplong, &tmplong, &tmplong, &tmplong, &starttime);
    fclose(fp);

    if (ret != 22) {
        return -2;
    }

    return convertStartTime(starttime);
}

int main(void)
{
    time_t timestamp;
    struct tm lt;
    char res[32];

    timestamp = (time_t)getProcessStatTime(1); //systemd

    localtime_r(&timestamp, &lt);
    strftime(res, sizeof(res), "%F %T", &lt);
    printf("%s\n", res);
    return 0;
}
```



