---
layout: post
title: Kotlin Basics - Operators and Variables
updated: 2019-04-05 19:30
image: kotlin_header_1.png
tags: [kotlin, jvm, code, howto]
---

## Kotlin Basics

Taking a look at this language, I thought it would be a good idea to show
you guys what I'm learning. I'll go over it from scratch, hopefully this helps both you and me.
Note that these are my notes(no pun intended), cheatsheet style. This post is going to go over basics, namely variables.
Before you go any further I highly recommend you setup and IDE like Intellij inorder to follow along.
You can follow [this](https://kotlinlang.org/docs/tutorials/getting-started.html) guide.

### Trying out the snippets.

You don't need to keep creating new files for each code snippet. I use the interpreter. Yup, you read that right.
Kotlin has an interpreter for IntelliJ. You can access it by going to Tools > Kotlin > KotlinREPL. You should
see a little REPL pop up below in IntelliJ. This is all you require to follow along, so let's get started.

### Operators

Kotlin has all the basic math operators. You can check them out. All this is done in the REPL. 
Once you're done typing in, use Ctrl + Enter or Command + Enter to execute the code.

{% highlight kotlin %}
1 + 2       // 3
22 / 7      // An integer divided by an integer will yield an integer
22.0 / 7    // Will yield a double
{% endhighlight %}

### Operators but not exactly

Kotlin actually translates the above operator to something much readable under the hood.
It's much readable imo.

{% highlight kotlin %}
1.add(2)    // same as 1 + 2
22.div(2)   // same as 22 / 7
22.rem(7)   // same as 22 % 7
{% endhighlight %}

### Variables

Any decent language has variables(except esoteric languages maybe). Kotlin has 2 keywords for declaring variables: val & var.
The main difference is that val is used for declaring constants whereas var is used for declaring...(drum roll please) variables.
This means I can't change the value of val at runtime, but I can do so for var. So something like the following would blow up.

{% highlight kotlin %}
val x = 32
x = 4
error: why tf would you do that.
{% endhighlight %}

So how do we fix this? Simple we use var.

{% highlight kotlin %}
var x = 32
x = 4
{% endhighlight %}

There is a catch however, note that kotlin is not really a dynamic language, but a statically typed language.
Wait, wut? But we didn't declare any types above? Yes you're right. Kotlin does the magic by doing something known as type inference.
It infers the type of the variable from the right hand value. Now, here's the catch. I can only reassign variables with the same type.
That's why this will blow up...

{% highlight kotlin %}
var intentionalBomb = 3
intentionalBomb = 3.0
error: this really isn't my type...
{% endhighlight %}

One cool thing to notice is that, kotling doesn't enforce the use of semicolons(great for python kinda programmers).

### Types

Now that I've told you about types, how do we _explicitly_ declare a variable with a type. Kotlin uses a funny syntax(funny to me because of my background in 'old' languages).
The syntax is kinda like the following

{% highlight kotlin %}
    val/var <some_variable_name> : <type> = <value>
{% endhighlight %}

Let's declare a string type variable and print it.

{% highlight kotlin %}
var greet: String = "Can you not...force me to talk to people."
println(greet)
{% endhighlight %}

The other types are Char, Int, Float, Double and a few more which you can find in the kotlin documentation.

### Template Strings

I've faced this problem a lot, and I think that many of you have faced the same problem too. String concatenation in some languages is just a nightmare.
Mostly end up many times with this pile of junk...

{% highlight kotlin %}
var a = 32.0
var b = "Dylan"
var t = "Why tf should " + b + " keep track of " + a + " concats and quotes. He demands an answer."
println(t)
{% endhighlight %}

Kotlin has the solution to this problem. It deals with this nightmare using template strings. So the above piece of junk becomes more concise and readable.

{% highlight kotlin %}
var a = 32.0
var b = "Dylan"
var t = "$b don't need to keep track of $a junk. This didn't make sense, meh."
println(t)
{% endhighlight %}

### Null Type Safety

A huge ass thanks to the creators of this language for dealing with this pain in the behind of Java.
No more "NullPointerException". Kotlin just won't let you write shitty code. Doing something like this would blow up.
(what is this? a Micheal Bay movie??).

{% highlight kotlin %}
var x: Int = null
error: oh no you don't
{% endhighlight %}

What if I have those "situations" in which I really really need to use a null type. Kotlin isn't some control freak bruh. It let's you do that too.
Like so...

{% highlight kotlin %}
var x: Int? = null
println(x)
{% endhighlight %}

Notice the '?' at the end of the type. This tells kotlin that this variable may or may not be null.
Kotlin howerver forces me to use '?.' and '!!.' calls on such types.

{% highlight kotlin %}
var x: Int? = null
x?.add(3)   // Will not throw an exception if x is null
x!!.add(3)  // Will certainly throw an error if x is null
{% endhighlight %}

So that's that for this post. Moving on we'll dive deeper into functions and arrays.

