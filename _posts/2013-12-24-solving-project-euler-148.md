---
layout: post
title:  "Solving Project Euler problem 148"
date:   2013-12-24
---

For the uninitiated, [Project Euler](projecteuler.net) is a fantastic source of brain food. The website contains hundreds of typically mathematically-based problems, and are usually best solved by programming solutions. For this reason, PE can be a great way to learn a programming language - if you're familiar enough with algorithms, Google is usually sufficient to fill the gaps in your code.

Because PE problems can be quite enjoyable to work out on your own, I hereby warn the reader that the following material may spoil their experience for [problem 148](http://projecteuler.net/problem=148).

This blog post is mostly an excuse to populate and test my website - however, I found this experience enlightening, and thought it was worth writing about. This PE problem was one that I had naively attempted to solve years ago with C, and revisited at the start of the year with Haskell. 148 is easy to understand, as so many PE problems are. As always, the devil is in the details, and investigation reveals many curiousities. Rather prominently, if you transform Pascal's triangle element-by-element into a 1 or 0 for "not divisible" or "divisible", you will get a [Sierpinski triangle](http://en.wikipedia.org/wiki/Sierpinski_triangle). The following code illustrates this nicely:

{% highlight haskell %}
import System.Environment

pascal = iterate (\row -> zipWith (+) ([0] ++ row) (row ++ [0])) [1]

numDivisible n = (\x -> if (x`mod`n==0) then 1 else 0)
numNotDivisible n = (\x -> if (x`mod`n==0) then 0 else 1)

main = do
  (arg0:arg1:_) <- getArgs
  let index = read arg0
      limit = read arg1

  mapM print $ take limit $ map (\x -> map (numNotDivisible index) x) pascal
{% endhighlight %}

Compile[^1] and run that as `./pascal_divbyn 3 27`, for example:

{% highlight haskell linenos %}
[1]
[1,1]
[1,1,1]
[1,0,0,1]
[1,1,0,1,1]
[1,1,1,1,1,1]
[1,0,0,1,0,0,1]
[1,1,0,1,1,0,1,1]
[1,1,1,1,1,1,1,1,1]
[1,0,0,0,0,0,0,0,0,1]
[1,1,0,0,0,0,0,0,0,1,1]
[1,1,1,0,0,0,0,0,0,1,1,1]
[1,0,0,1,0,0,0,0,0,1,0,0,1]
[1,1,0,1,1,0,0,0,0,1,1,0,1,1]
[1,1,1,1,1,1,0,0,0,1,1,1,1,1,1]
[1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1]
[1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1]
[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
[1,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,1]
[1,1,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,1,1]
[1,1,1,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,1,1,1]
[1,0,0,1,0,0,0,0,0,1,0,0,1,0,0,0,0,0,1,0,0,1]
[1,1,0,1,1,0,0,0,0,1,1,0,1,1,0,0,0,0,1,1,0,1,1]
[1,1,1,1,1,1,0,0,0,1,1,1,1,1,1,0,0,0,1,1,1,1,1,1]
[1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1]
[1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1]
[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
{% endhighlight %}

This shows you all the elements of the first 27 rows of Pascal's triangle that are not divisible by 3. Very cute.

The pattern is reasonably obvious from here. The powers of your prime (in this case, 3) are important; that's where the fractal pattern repeats (is that the right way to describe it?). So, back to the problem - we develop a formula involving triangular numbers that easily describes all numbers divisible by 7 up to the highest power of 7 less than $10^9$. But what do you do for those rows between $7^{\log_{7}10^{9}}$ and $10^9$? If you tried to brute force it, you would have $10^9$ mod operations on *just the last row*. There must be a better way.

If you sum each of these rows and print the result (such as with the following line):

{% highlight haskell %}
  mapM print $ take limit $ map (\x -> sum $ map (numNotDivisible index) x) pascal
{% endhighlight %}

... you will get:

{% highlight haskell linenos %}
1
2
3
2
4
6
3
6
9
2
4
6
4
8
12
6
12
18
3
6
9
6
12
18
9
18
27
{% endhighlight %}

There's definitely a pattern here! If you could generalise this up to an arbitrary number (say, $10^9$), then we're done.

It turns out my mathematician friend[^2] knew of a very relevant identity.

* * *
[^1]: I usually compile Haskell code with something like: `ghc -O2 --llvm <code.hs>`
[^2]: Thanks Dave Horsley!
