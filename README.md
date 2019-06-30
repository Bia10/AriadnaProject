# 🧶 The Ariadna Project
This is the Win32 threading library with functions, wrappers and classes with support of:
* Usermode scheduling (UMS-threads)
* Fibers
* Normal threads

#### How to:
* Usermode scheduling:
```cpp
#include <Windows.h>
#include "Ariadna.h"

DWORD WINAPI UmsThread(PVOID Arg)
{
    // Your code here
}

int main()
{
    using namespace Ariadna;
    UmsScheduler& Scheduler = UmsScheduler::GetInstance();
    
    // Initializing:
    Scheduler.StartUmsSheduler();
    
    // Starting threads:
    for (unsigned int i = 0; i < 5; ++i) {
        Scheduler.StartThread(UmsThread);
    }
    
    ...
    
    // Stopping scheduler:
    Scheduler.StopUmsScheduler();
    
    return 0;
}
```
* Fibers:
```cpp
#include <Windows.h>
#include "Ariadna.h"

int main()
{
    Ariadna::Fibers::CallInFiber([](PVOID) {
        printf("Fiber %p in thread %i\r\n", Fibers::Current(), GetCurrentThreadId());
    });

    printf("Non-fiber main thread %i\r\n", GetCurrentThreadId());
    return 0;
}
```
* Normal threads:
```cpp
#include <Windows.h>
#include "Ariadna.h"

using namespace Ariadna;

int main()
{
    volatile int i = 1337; // Value that will be pass to another thread

    // Run async task:
    Threads::Async([&]() -> DWORD { 
        Sleep(1000);
        printf("[%u] External value = %i\r\n", Threads::Id(), i);
        return 0;
    });

    i = 0; // Changing the 'global' value

    printf("[%u] Value 'i' changed to %i\r\n", Threads::Id(), i);

    // To prevent an app closing:
    MSG Message = {};
    while (GetMessage(&Message, NULL, 0, 0)) {
        TranslateMessage(&Message);
        DispatchMessage(&Message);
    }

    return 0;
}
```
or
```cpp
#include <Windows.h>
#include "Ariadna.h"

using namespace Ariadna;

class CustomThread : public AbstractThread {
private:
    int SampleValue;
protected:
    DWORD ThreadProc() override
    {
        printf("Hi from a CustomThread!\r\n");
        printf("- SampleValue = %i\r\n", SampleValue);
        return SampleValue;
    }
public:
    CustomThread(int Value) {
        SampleValue = Value;
    }
};

int main()
{
    CustomThread thread(1337);
    thread.Start();
    thread.Wait();
    printf("CustomThread is finished with value %i!\r\n", thread.GetExitCode());
    return 0;
}
```
