# timestamp to datetime

convert unix timestamp to datetime, for example `1634180667` -> `2021-10-14 11:04:27`.

## Key Functions

```c
#include <time.h>

time_t time(time_t *tloc);

struct tm *localtime_r(const time_t *timep, struct tm *result);

size_t strftime(char *s, size_t max, const char *format, const struct tm *tm);
```

## Codes

```c
#include <time.h>
#include <stdio.h>  /* printf */
#include <stdlib.h> /* atoi   */

int main(int argc, char *argv[])
{
    time_t timestamp;
    const char *format = "%F %T"; // man strftime
    struct tm lt;
    char res[32];

    if (argc >= 2) {
        timestamp = (time_t) atoi(argv[1]);
    } else {
        timestamp = time(NULL);
    }

    localtime_r(&timestamp, &lt);
    strftime(res, sizeof(res), format, &lt);

    printf("%s\n", res);
    return 0;
}
```

## References

https://www.epochconverter.com/

https://stackoverflow.com/questions/18582119/how-to-convert-unix-timestamp-into-daymonthdateyear-format-in-c