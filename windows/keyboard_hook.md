# keyboard hook

## APIs

| API                                                                                                              | Description                                                                       |     |
| ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | --- |
| [SetWindowsHookEx](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowshookexw)      | Installs an application-defined hook procedure into a hook chain.                 |     |
| [UnhookWindowsHookEx](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-unhookwindowshookex) | Removes a hook procedure installed in a hook chain.                               |     |
| [CallNextHookEx](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-callnexthookex)           | Passes the hook information to the next hook procedure in the current hook chain. |     |

----

## Resources

[Virtual-Key Codes](https://docs.microsoft.com/en-us/windows/win32/inputdev/virtual-key-codes)

[Keyboard Scan Code Specification](https://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/scancode.doc)

[MapVirtualKey](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-mapvirtualkeyw)

---

## Example

```cpp
#define _CRT_SECURE_NO_WARNINGS

#include <windows.h>
#include <stdio.h>

HHOOK keyboard_hook_handle = 0;

LRESULT CALLBACK keyboard_proc_cb(int code, WPARAM wparam, LPARAM lparam)
{
    const char* desc;
    char extra[128] = "";
    PKBDLLHOOKSTRUCT key = (PKBDLLHOOKSTRUCT)lparam;
    switch (wparam) {
    case WM_KEYDOWN:
        desc = "Key Down    ";
        break;
    case WM_KEYUP:
        desc = "Key Up      ";
        break;
    case WM_SYSKEYDOWN:
        desc = "Sys Key Down";
        break;
    case WM_SYSKEYUP:
        desc = "Sys Key Up  ";
        break;
    default://nerver go here
        desc = "UNKONW      ";
    }

    if (key->flags & LLKHF_EXTENDED) {
        strcat(extra, "(extended-key)");
    }
    if (key->flags & LLKHF_UP) {
        strcat(extra, "(transition-state)");
    }
    if (key->flags & LLKHF_ALTDOWN) {
        strcat(extra, "(alt down)");
    }
    if (key->flags & LLKHF_LOWER_IL_INJECTED) {
        strcat(extra, "(lower inject)");
    }
    if (key->flags & LLKHF_INJECTED) {
        strcat(extra, "(inject)");
    }
    printf("[%s] vkCode=%#0.2lx, scanCode=%#0.2lx%s\n", desc, key->vkCode, key->scanCode, extra);

    return CallNextHookEx(keyboard_hook_handle, code, wparam, lparam);
}

int main(void)
{
    keyboard_hook_handle = SetWindowsHookEx(WH_KEYBOARD_LL, keyboard_proc_cb, NULL, 0);
    if (keyboard_hook_handle == NULL) {
        printf("SetWindowsHookEx failed, %ld\n", GetLastError());
        return 0;
    }

    //message loop
    MSG msg;
    //If GetMessage retrieves the WM_QUIT message, the return value is zero.
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    //exit
    UnhookWindowsHookEx(keyboard_hook_handle);

    return 0;
};
```

---

## How to Remove INJECTED flags

https://www.unknowncheats.me/forum/c-and-c-/326487-trying-remove-flag-input.html

https://www.unknowncheats.me/forum/c-/436468-removing-llkhf_injected-flag-keyboard-mouse-events.html

[GitHub - Sadulisten/LowLevelMouseHook-Example](https://github.com/Sadulisten/LowLevelMouseHook-Example)