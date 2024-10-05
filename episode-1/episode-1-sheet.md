>[!bug] Linux Malware Development Episode 1
>
>>[!info] Process Management
>>A process is a program in execution.
>
>>[!info] Main header files
>>- `<stdio.h>` For input and output operations.
>>- `<unistd.h>` Give access to `unix` standard functions (as those used in process management operations).
>>- `<sys/wait.h` Provides a wait function. 
>>```c
>>#include <stdio.h>
>>#include <unistd.h>
>>#include <sys/wait.h>
>>```
>
>>[!tip] Wait functions
>>- They can be use to make sure a child process is finished before it can get adopted.
>
>>[!info]
>>- An adopted process refers to what happens when a parent process terminates before its child process.
>>- In `unix` systems processes are automatically adopted by the `init` process or its model equivalent.
>>- The `init` process typically with is unique identifier (`PID:1`) becomes a new parent of those offing processes.
>
>>[!bug] This functions shows:
>>- How Linux can run multiple processes simultaneously.
>>- How processes can be synchronised.
>>```c
>>void main() {
>>
>>//getpid() is used to get our own process include
>>
>>printf("parent: My process id is %d\n", getpid());
>> 
>>//fork() is to create a new child process and return it's process id
>>
>>pid_t cpid = fork();
>>
>>switch(cpid) {
>>
>>case -1:
>>
>>perror("fork"); // perror is to print error
>>
>>return;
>>
>>//child process will execute code at case 0:
>>
>>case 0:
>>
>>// getppid() is used to get our parent process id
>>
>>printf("child: My process ud is %d and my parent process id is %d\n", getpid(), getppid());
>>
>>// parent process will execute at default:
>>
>>default:
>>
>>wait(NULL); // Wait for child to finish
>>
>>}
>>
>>return;
>>}
>>```
>>
>>>[!bug] Time Sharing
>>>- Each process operates as if it has exclusive access to the `CPU` unaware of other process is competed for resources
>>>- The `OS` rapidly changes attention between different processes given each a small slice of time to execute
>>>- This process happens so quickly that from the processor's perspective it appears to have been continuous uninterrupted access to the `CPU`.
>>>- This abstraction allows programmers to write software as if their program is the only one running.
>---
>
>>[!info] ELFs
>>- Stands for `Executable and linkable formats`
>>
>#### Used for:
>>[!info] Executable files
>>- Programs that can run directly to the `CPU` 
>>- Contains machine code instructions.
>
>>[!info] Shared object files/libraries
>>- Contain usable code that multiple programs can use.
>>- Allows multiple programs to be very efficient and save memory space by sharing the same code.
>>- Often carry the `.so` file extension
>>
>>>[!info] Dynamic Linking
>>>- Symbols (code) are resolved during runtime.
>>>- The program and its dependencies are linked together dynamically by the **dynamic linker/loader**.
>
>>[!tip]
>>- By using `ldd` command we can list the shared libraries used by a binary, in this case `ls`:
>>```
>>➜  ~ ldd /bin/ls
>>	linux-vdso.so.1 (0x00007a701d5c0000)
>>	libcap.so.2 => /usr/lib/libcap.so.2 (0x00007a701d56a000)
>>	libc.so.6 => /usr/lib/libc.so.6 (0x00007a701d379000)
>>	/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007a701d5c2000)
>>```
>#### How to use shared libraries and function hooks to modify program behaviour at runtime:
>>[!info] Main header files
>>- Declare a macro such as `RTLD_NEXT` to use *_symbol interposition_* by dynamic linking.
>>- `RTLD_NEXT` is used to indicate that the next occurrence of a symbol after the current one should be used.
>>- `dlfcn.h` is needed to load and use shared libraries at runtime.
>>
>>>[!tip] Symbol Interposition
>>>- Allows to override the behavior of an existing functions by providing your own version of the function in your program.
>>- **`<dirent.h>` Provides the `DIR`(represents a directory stream) and `struct dirent` Types**(holds the information returned by `readdir()`)
>>>[!important]
>>>- These types are essential when working with directory manipulation functions, and you need them in order to interact with directories using `opendir()`, `readdir()`, and `closedir()`.
>>- The **`<stdlib.h>`** header provides functions for general-purpose **utility tasks**.
>>The **`<stdio.h>`** header provides functionality for **input and output** operations, as we already mention.
>>```c
>>#define _GNU_SOURCE
>>#include <dlfcn.h>
>>#include <dirent.h>
>>#include <stdlib.h>
>>#include <stdio.h>
>>```
>
>>[!important] Before hooking:
>>- First, we need the signature from the function we are planning to hook.
>>- Creating a `typedef` with the original signature allows us to reference it later.
>>- Second, we need to define a pointer that will hold the  function data type.
>
>```c
>typedef struct dirent *(*readdir_t)(DIR *dirp);
>
>readdir_t OG_readdir = NULL;
>```
>
>>[!warning] The hook function
>>- bla
>>```c
>>struct dirent *readdir(DIR *dirp) {
>>
>>if (!OG_readdir) OG_readdir = (readdir_t)dlsym(RTLD_NEXT, "readdir");
>>
>>printf("Shared object hooked into readdir\n");
>>
>>return OG_readdir ? OG_readdir(dirp) : NULL;
>>
>>}
>>```
>
>>[!info]
>>```c
>>__attribute__((constructor)) is used to execute a function from the shared object file
>>once it has loaded into a binary
>>*/
>>
>>__attribute__((constructor))
>>
>>void in_the_front_door() {
>>
>>printf("Shared object loaded into the binary\n");
>>
>>}
>>```
>
>>[!info]
>>```c
>>__attribute__((destructor)) is used to execute a function from the shared object file once it has been unloaded from a binary
>>*/
>>
>>__attribute__((destructor))
>>
>>void out_the_back_door() {
>>
>>printf("Shared object unloaded from the binary\n");
>>
>>}
>>```
