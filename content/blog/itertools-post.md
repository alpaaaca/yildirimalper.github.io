---
author: "Alper Yildirim"
date: 2018-03-12
title: Itertools or no itertools
---

When I first saw itertools, I was thinking why anyone would ever need that? 
However, after using itertools, I can't really think why we should use something else in Python when it comes to data.

I learned about itertools from Joel Grus in one of his PyData Talks. I found it very interesting that he said something like:
"When I first saw itertools, I was thinking why anyone would ever need that? After using it, why the hack do I want to use anything else?"
if you haven't checked it out yet, I strongly encourage you to check it, it was a very fun presentation.
https://www.youtube.com/watch?v=ThS4juptJjQ

This was also the case for me. As a data engineer back then, I was all about `pandas` and was a bit desperate when I had to write code that would not eat up memory
and would allow me to write .. 

Itertools is a beautiful library that allows lazy computation / evaluation in python and it has got methods such as:

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


[link]: https://docs.python.org/3/library/itertools.html 
 








