---
title: 'Chrom : Open source lib alternative for API/Function interception/hooking'
date: 2011-08-24 07:03:00.000000000 +05:30
---

Chrom, is an open source(under LGPL) library I wrote for API/Function interception/hooking.
It can be used as an alternative to Microsoft Research's detours.

Downloads
* [Chrom-lib](https://github.com/linuxexp/chrom-lib)
* [Firefox network interception](https://github.com/linuxexp/chrom-lib/blob/master/firefox_hook/chrom/chrom.cpp)

## How it works

Chrom overwrites the first 6 bytes of the destination function with JUMP instruction to jump to our function. Then when the original function is called, JUMP instruction jumps to our function's address.

```c
BYTE JMP_temporary[SIZE] = {0xE9, 0x90, 0x90, 0x90, 0x90, 0xC3};
memcpy(JMP_instruction, JMP_temporary, SIZE);

DWORD JMP_size = ((DWORD)destination_function - (DWORD)original_function - 5);
memcpy(JMP_instruction[1], JMP_size, 4);
```

## Chrom-lib API

```c
int Hook::Initialize(char * function, char * module_name, void * destination_function_ptr);
int Hook::Start();
int Hook::Reset();
int Hook::Place_Hook();
```

To place a hook successfully, you should first call initialize function from instance of Hook struct, then you should pass, function name, module name, and your function as parameters, note, that module (eg "abc.dll") should already be loaded in process space.

example

```c
// Override PR_Write function in nspr4.dll with our PR_Write_H, 
// Note nspr4.dll must already be
// loaded in process space
Firefox.Initialize("PR_Write", "nspr4.dll", PR_Write_H);
// Write jump instruction on orginal function address
Firefox.Start();
```


Then you should call `Hook::Start()`, this will compute jumps and will write jump instruction on orginal function address

After that you'll be ready, when the original function is called, your function will be called. If for example, you need to call the actual real function, you can first call `Hook::Reset()` this will remove the JUMP instructions and will replace it with original data. Then when you have finished calling it, you can again replace JUMP instructions by calling `Hook::Place_Hook()`

example

```c
Firefox.Reset();
// point prw(function) to original function
prw = (prWrite)Firefox.original_function;
// log the headers
write_log(log_file, (char*) buf);
// call the real PR_Write function
DWORD ret = prw(fd, buf, amount);
// again place the jump instruction on the original function
Firefox.Place_Hook();
```

That's it, Below is a complete source of a DLL, that hooks to PR_Write function of `nspr4.dll` that firefox uses to write to a file descriptor, this is used to log all the headers (POST, GET data) of HTTP/HTTPS requests. 

The example code is purely for demonstrative purpose. I can't be held responsible for whatever you may do with it.

Moved to github <a href="https://github.com/linuxexp/chrom-lib/blob/master/firefox_hook/chrom/chrom.cpp">https://github.com/linuxexp/chrom-lib/blob/master/firefox_hook/chrom/chrom.cpp</a>
