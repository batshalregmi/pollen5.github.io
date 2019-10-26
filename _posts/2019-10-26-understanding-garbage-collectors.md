---
layout: post
title: "Understanding Garbage Collectors"
categories: language-design
---

**What is a garbage collector?** Garbage collectors (also known as GC, for the rest of this post i'll refer to it by GC) reclaims memory for objects that go out of scope.

Consider this simple python program:
{% highlight python %}
def hello():
  x = "hello"
  y = "world"

hello()
{% endhighlight %}

When inside the `hello` function, two strings are allocated in `x` and `y` now they are in scope, after leaving the function we lost `x` and `y` and we didn't delete the memory used for them, we have leaked memory!!! Fear not, fortunately python comes with a GC so when we lost the references to `x` and `y`, Python itself did not and it knew that they are garbage now and freed their memory for us!

GCs might look like a black box of magic and no one knows what's inside, but in reality it's pretty simple and we will go through some algorithms used for GCs in this post.

### Reference Counting
One of GC algorithms is reference counting, this is the way python does it. It means when an object is made it's ref count is 1 as it gets referenced more that count gets incremented, likewise when it goes out scope the count gets decremented, when it becomes 0 destroy that object, this is good but cyclic references become an issue and we will need additional care, Python does not have this issue.

### Mark and Sweep
Mark and sweep is another one of popular algorithms so much that often just the term GC refers to them, how it works is that each object stores an `isMarked` attribute, and when GC kicks in there is 2 phases, first the mark phase traverses all alive objects and marks them, then the second phase is the sweep phase, it traverses all objects and those that did not get marked is sweeped or freed.

This is great and the algorithm avoids cyclic references as is, but introduces one issue, pause times, this GC is an STW (Stop-the-world) it means while it's doing it's job the whole program stops until GC is done, this is bad especially if the language will be used for games because it would result in some fps drops here and there, to solve this there's another solution:

### Tri-color incremental Mark and Sweep GC
First let me introduce you to the term "incremental": it means the GC performs it's job incrementally, in other words each sweep phase is done seperately not all at once, this is still stop the world, kind of but the pauses are less, the overall time to complete a GC is longer than a regular mark and sweep but each phase is shorter and pauses are less noticible.

Tri-color means each object has one of the 3 colors, white, black or gray, the rule of thumb for each of them is as follows:

- **White:** White objects are targets to be sweeped.
- **Gray:** Marked but child references is not marked.
- **Black:** Like gray but child references is also marked. (Black objects cannot reference any whites.)

The algorithm goes as follows: In the mark phase objects are placed in a *gray stack* then the gray stack is processed to mark them as black, once there is no gray objects we are done, then the sweep phase scans for remaining whites and frees them.

<!-- TODO: explain generational GCs -->
