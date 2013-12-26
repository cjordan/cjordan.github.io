---
layout: post
title:  "Solving Project Euler problem 148"
date:   2013-12-24
---

For the uninitiated, [Project Euler](projecteuler.net) is a fantastic source of brain food. The website contains hundreds of typically mathematically-based problems, and are usually best solved by programming solutions. For this reason, PE can be a great way to learn a programming language - if you're familiar enough with algorithms, Google is usually sufficient to fill the gaps in your code.

Because PE problems can be quite enjoyable to work out on your own, I hereby warn the reader that the following material may spoil their experience for [problem 148](http://projecteuler.net/problem=148).

This blog post is mostly an excuse to populate and test my website - however, I found this experience enlightening, and thought it was worth writing about. This PE problem was one that I had naively attempted to solve years ago with C, and revisited at the start of the year with Haskell. 148 is easy to understand, as so many PE problems are. The devil is in the details, and investigation reveals many curiousities. Rather prominently, if you transform Pascal's triangle element-by-element into a 1 or 0 for "not divisible" or "divisible", you will get a [Sierpinski triangle](http://en.wikipedia.org/wiki/Sierpinski_triangle). The following code illustrates this nicely:

{% highlight haskell linenos=table %}
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

{% highlight haskell linenos=table linenostart=0 style=emacs %}
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

The pattern is reasonably obvious from here. The powers of your prime (in this case, 3) are important; that's where the fractal pattern repeats (is that the right terminology?). So, back to the problem - we develop a formula involving triangular numbers that easily describes all numbers divisible by 7 up to the highest power of 7 less than $10^9$. But what do you do for those rows between $7^{\lfloor\log_{7}10^{9}\rfloor}$ and $10^9$? If you tried to brute force it, you would have $10^9$ mod operations on *just the last row*. There must be a better way.

If you sum each of these rows and print the result (such as with the following line):

{% highlight haskell %}
  mapM print $ take limit $ map (\x -> sum $ map (numNotDivisible index) x) pascal
{% endhighlight %}

... you will get:

{% highlight haskell linenos=table linenostart=0 %}
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

It turns out my mathematician friend[^2] knew of a very relevant theorem[^3]. It basically boils down to taking the row number of Pascal's triangle and converting it to base p (prime), adding 1 to each digit, and multiplying all digits. A couple of examples, returning to our prime 3; look at row 17 and 18:

$$ 17_{10} = 122_3 $$
$$ \gg 233 $$
$$ \gg 18 $$

$$ 18_{10} = 200_3 $$
$$ \gg 311 $$
$$ \gg 3 $$

This means row 17 of Pascal's triangle contains 18 numbers that are *not* divisible by 3, and row 18 has 3 numbers.

So, you could be terrible (like me), and write some code that counts from $0$ to $10^9$, converts to base 7, increments and multiplies, such as below:

{% highlight c linenos=table %}
#include <stdio.h>
#include <math.h>

void inc_m(int m[], int size, int index) {
    m[0] += 1;
    int i;
    for (i = 0; i < size; i++) {
        if (m[i] == 7) {
            m[i] = 0;
            m[i+1] += 1;
        }
    }
}

int get_t(int m[], int size) {
    int t = 1;
    int i;
    for (i = 0; i <= size; i++)
        t = t*(m[i]+1);
    return t;
}


int main(int argc, const char* argv[]) {
    int limit;
    int index;
    if (argc == 1) {
        limit = 1000000000;
        index = 7;
        printf("Limit defaulting to: %d\n",limit);
        printf("Index defaulting to: %d\n",index);
    }
    else if (argc == 2) {
        printf("Please specify upper bound and prime to be used.\n");
        return 1;
    }
    else {
        limit = atoi(argv[1]);
        index = atoi(argv[2]);
    }

    int size = ceil(log(limit)/log(7));
    int m[size];
    int i;
    for (i = 0; i <= size; i++)
        m[i] = 0;

    unsigned long int t = 0;
    for (i = 0; i < limit; i++) {
        t = t + get_t(m,size);
        inc_m(m,size,index);
    }

    printf("%lu %d\n",t,i);
    return 0;
}
{% endhighlight %}

This solves 148 in about 20s on my i7-2960XM laptop. Impressive, given the crude nature of this solution.

However, as my friend[^2] points out, this could be simplified greatly.

* * *
[^1]: I usually compile Haskell code with something like: `ghc -O2 --llvm <code.hs>`
[^2]: Thanks Dave Horsley!
[^3]: Lucas' theorem - [Paper](http://copper.math.buffalo.edu/urgewiki/uploads/Literature2010Carbonara/Fine47.pdf) and [Wikipedia article](https://en.wikipedia.org/wiki/Lucas%27_theorem).
