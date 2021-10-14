# Get user and domain by pid

```cpp
// vs2019
#include <windows.h>
#include <comdef.h>
#include <stdio.h>


#define MAX_NAME 256

int getPidUserDomain(DWORD pid, LPWSTR lpName, LPWSTR lpDomain)
{
    DWORD dwSize = 0;
    HANDLE hToken;
    HANDLE hProcess;
    PTOKEN_USER pUserInfo;
    SID_NAME_USE sidType = SidTypeUser;

    hProcess = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION, FALSE, pid);
    if (NULL == hProcess) {
        printf("OpenProcess Error %u\n", GetLastError());
        return -1;
    }

    // Open a handle to the access token for the calling process.
    if (!OpenProcessToken(hProcess, TOKEN_QUERY, &hToken)) {
        printf("OpenProcessToken Error %u\n", GetLastError());
        return -1;
    }

    // Call GetTokenInformation to get the buffer size.
    if (!GetTokenInformation(hToken, TokenUser, NULL, dwSize, &dwSize)) {
        if (GetLastError() != ERROR_INSUFFICIENT_BUFFER) {
            printf("GetTokenInformation 1 Error %u\n", GetLastError());
            return -1;
        }
    }

    // Allocate the buffer.
    pUserInfo = (PTOKEN_USER)GlobalAlloc(GPTR, dwSize);
    if (nullptr == pUserInfo) {
        return -1;
    }

    // Call GetTokenInformation again to get the group information.
    if (!GetTokenInformation(hToken, TokenUser, pUserInfo, dwSize, &dwSize)) {
        GlobalFree(pUserInfo);
        printf("GetTokenInformation 2 Error %u\n", GetLastError());
        return -1;
    }

     // Lookup the account name.
     dwSize = MAX_NAME;
     if (!LookupAccountSid(NULL, pUserInfo->User.Sid, lpName, &dwSize, lpDomain, &dwSize, &sidType)) {
         GlobalFree(pUserInfo);
         printf("LookupAccountSid Error %u\n", GetLastError());
         return -1;
      }


    if (pUserInfo)
        GlobalFree(pUserInfo);

    return 0;
}

int main(void)
{
    WCHAR lpName[MAX_NAME];
    WCHAR lpDomain[MAX_NAME];

    auto ret = getPidUserDomain(4, lpName, lpDomain);
    if (!ret) {
        printf("USER: %S\n", lpName);
        printf("DOMAIN: %S\n", lpDomain);
    }
    return 0;
}
```