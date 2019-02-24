+++
title = "Note to self.self: List comprehensions in Python 2.x leak!"
author = ["Olafur Bogason"]
date = 2015-10-18
lastmod = 2019-02-24T13:07:37+00:00
tags = ["python"]
categories = ["python"]
draft = false
+++

After considerable amount of debugging on some code I've been writing in Python 2.x I finally found the bug. This time it didn't arise because I was sloppy, as it so often does, but because of faulty behaviour intrinsic to the language design of Python 2.x itself..

{{< figure src="/images/18-dear-liza.png" >}}

```python
'This is an example of list comprehension control variables being leaked to their outer scope'

for i in range(10):
    a = [i for i in range(43)]
    print(i) # will print 42 every time!
```

It's no surprise that [others](http://stackoverflow.com/questions/4575698/python-list-comprehension-overriding-value) have encountered this bug/feature before and it has supposedly been fixed in Python 3.x.

Maybe this is a good example of why I should be switching over to Python 3.x soon...ish...
