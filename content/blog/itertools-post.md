---
author: "Alper Yildirim"
date: 2018-03-12
title: Itertools or no itertools
---

Itertools, in a nutshell, is a beautiful python library that allows out-of-the-box computations and functions using lazy evaluation and iterators. What I like about them is that they could be adapted to work with 
large quantities of data in batch-processing and python ecosystem.

I learned about itertools in one of [PyData] talks and I was puzzled by what the speaker said at that time. He said: 

> "When I first saw itertools, I was thinking why anyone would ever need that? After using it, why the hack do I want to use anything else?"
> - Joel Grus (the name of the speaker)

This was exactly how I felt about `itertools` too. As a data engineer back then, I abused `pandas` library a lot, but then as the data-sets grew bigger on batch processing, I knew I had to use something else.
And this was the time I started introducing itertools to my data pipelines.


You can check about the other methods in this [link].

There will be a follow-up with examples and cool things to do with `itertools`


[PyData]: https://www.youtube.com/watch?v=ThS4juptJjQ
[link]: https://docs.python.org/3/library/itertools.html 
 








