---
author: "Alper Yildirim"
date: 2018-03-12
title: Itertools or no itertools
---

Itertools, in a nutshell, is a beautiful python library that allows lazy computation or way to create iterators. What I like about them is that they could be adapted to work with large quantities of data.

I learned about itertools in one of [PyData] videos on the internet and the speaker said at the time. 

"When I first saw itertools, I was thinking why anyone would ever need that? After using it, why the hack do I want to use anything else?"

I did not understand this at the time because I was also like "Why would I need such a thing?" and then the more I learned about it and used it in data engineering, the more I ca

This was also the case for me. As a data engineer back then, I was all about `pandas` and was a bit desperate when I had to write code that would not eat up memory
and would allow me to write .. 

```python
count()
cycle()
repeat()
accumulate()
chain()
chain.from_iterable()
compress()
```

One may think: why would I ever need that? In python, there is already great deal of things.
Well, the answer is yes and no. 

You can check about the other methods in this [link], the examples there are quite nice too.

Please, check it here if you have not yet. I find the examples there already very useful, but in this article 
I will show more real use case examples.

You can really do pretty much all the cool stuff with this library and it really gives 

Instead of repeating the examples, I would like to go over a real-case scenario example, which I think might give a better
gist of it.


[PyData]: https://www.youtube.com/watch?v=ThS4juptJjQ
[link]: https://docs.python.org/3/library/itertools.html 
 








