# Windows Epoch to Unix Epoch

- **Windows Epoch** - The NT time epoch on Windows NT and later refers to the Windows NT system time from `00:00:00 1 January 1601`
- **Unix Epoch** - The Unix epoch (Unix time/POSIX time/Unix timestamp) is the number of seconds that have elapsed since `00:00:00 January 1, 1970`

## Codes

```c
#include <Windows.h>
#include <stdio.h>

ULONGLONG filetimeToUnixEpoch(FILETIME ft)
{
    // takes the last modified date
    LARGE_INTEGER date, adjust;
    date.HighPart = ft.dwHighDateTime;
    date.LowPart = ft.dwLowDateTime;

    // 100-nanoseconds = milliseconds * 10000
    adjust.QuadPart = 11644473600000 * 10000;

    // removes the diff between 1970 and 1601
    date.QuadPart -= adjust.QuadPart;

    // converts back from 100-nanoseconds to seconds
    return date.QuadPart / 10000000;
}

int main(void)
{
    ULONGLONG ep;
    FILETIME ft;
    SYSTEMTIME lt;

    GetLocalTime(&lt);
    SystemTimeToFileTime(&lt, &ft);

    ep = filetimeToUnixEpoch(ft);

    printf("%llu\n", ep);
    return 0;
}
```

## References

https://docs.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-systemtime

https://stackoverflow.com/questions/6161776/convert-windows-filetime-to-second-in-unix-linux/6161842
