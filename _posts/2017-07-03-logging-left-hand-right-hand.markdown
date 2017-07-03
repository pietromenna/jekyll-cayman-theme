---
layout: default
title:  "Right Hand Doesn't Know What the Left Hand is Doing"
date:   2017-07-03 11:21:00
categories: python performance
abstract: in which I point out a source of latency most people never think about when it comes to logging
---

I do a lot of data analysis in python processing and analyzing (with many different metrics) million of records at a time.

A performance hit, even if small, is magnified when mulitplied by millions.

Similarly, a logic error, even if minor, is magnified with multiplied by millions. 

So, `debug(<some message>)` is liberally strewn throughout my code. This is great because when I'm not in debug mode I spend ~~no~~ negligible time on these debug statements. This satisfies both my requirements:

1. makes it easier to find errors
2. keeps my code performant

Right?

Wrong! Debug log statements, even when not in a debug mode (ie. no handlers concerned with debug level), can eat large chunks of time. Here is why:

Take this common case:

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

In the above examples, the left hand side does not format unless one of the log handlers is handling debug statements. So, unless you are in debug there is no cost for the string format. This is great!

The right side, though, is a problem.

Whether or not the left side gets processed, the right side does. The arguments for the formatting get processed. In the above examples, `some_str` isn't a big deal, but `str(list_of_objects)` could be. 

Basic primatives are not a problem:

```python
logger.debug("I'm logging some dumb stuff like %d and %s", 1, 'a')
```

But it is a problem if the right hand side is more complex, such as:

```python
logger.debug("I'm logging a really big list comprehension as a string: %s" % str(x for x in range(1000000)))
```

But who the hell would ever log a list comprehension like that? It's illustrative. Let's take a more realistic example.

```python
logger.debug("I'm logging a custom object with its own __str__ method: %s" % str(custom_object_instance))
```

And now let's think about what the `__str__` function does. What if it does the following:
1. create a dictionary of 30 key/values pairs where key is some string and value is a locally held variable.
2. creates a string of that dictionary with `json.dumps` (json formatting is ungodly slow in python)

This is a pretty realistic example that you could easily run across. If that object is logged 6 times in a data processing pipeline and it is done for 1 million objects, you now are calling json.dumps 6 million times.

And if you aren't in debug that is 6 million calls to a slow function (`json.dumps`) that are being made with absolutely zero benefit!

The different levels of handlers for logging are great. They can save you processing time with their selectiveness and make your code easier to debug, but always remember: **the right hand does not know what the left hand is doing**.

