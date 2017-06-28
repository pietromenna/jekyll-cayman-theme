---
layout: default
title:  "Left Hand Doesn't Know What the Right Hand is Doing"
date:   2017-06-28 12:00:00
categories: python
---

I do a lot of data analysis in python and more often than not this means reading, processing, analyzing (with many different metrics) millions and millions of records at a go. This means for loops with many replicated actions.

So I find myself focusing on performance because any one hang-up, even if small, is magnified when mulitplied by millions.

I also find myself making a lot of use of logging because one small error in logic is magnified with multiplied by millions. 

So, `debug(<some message>)` is liberally strewn throughout my code. With a command line or configuration change I can run a process with an log level of debug and get much better insight into what is going on. And this is great because when I'm not in debug mode I spend ~~no~~ negligible time on these debug statements. This satisfies both my requirements:

1. makes it easier to find errors
2. keeps my code performant

Right?

Wrong! At least on the "keeps my code performant" part. If you aren't careful debug log statements, even when not in a debug mode, can eat large chunks of time. Here is why:

A common case:

```python
logger.debug("Processed %s, resulting in %s" % (some_str, str(list_of_objects))
```

You could also do this, which guarantees a lazier processing of the string formatting:

```python
logger.debug("Processed %s, resulting in %s", some_str, str(list_of_objects))
```

But there is a problem. For clarity, let's break these log messages down into two parts:

1. Left hand: the string to format
2. Right hand: the arguments

In the above examples, the string is not formatted until the 


