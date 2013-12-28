---
layout: post
title:  "Solving Project Euler problem 148"
date:   2013-12-29
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
$$ \rightarrow 233 $$
$$ \rightarrow 18 $$

$$ 18_{10} = 200_3 $$
$$ \rightarrow 311 $$
$$ \rightarrow 3 $$

This means row 17 of Pascal's triangle contains 18 numbers that are *not* divisible by 3, and row 18 has 3 numbers.

So, you could be terrible (like me), and write some code that counts from $0$ to $10^9$, converts to base 7, increments and multiplies, such as below:

{% highlight c linenos=table %}
#include <stdio.h>
#include <math.h>

void inc_m(int m[], int index) {
    m[0] += 1;
    int i = 0;
    while (m[i] == index) {
        m[i] = 0;
        i++;
        m[i] += 1;
    }
}

int get_t(int m[], int size) {
    int t = 1;
    int i;
    for (i = 0; i < size; i++)
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

    int size = ceil(log(limit)/log(index));
    int m[size], i;
    for (i = 0; i < size; i++)
        m[i] = 0;

    unsigned long int t = 0;
    for (i = 0; i < limit; i++) {
        t = t + get_t(m,size);
        inc_m(m,index);
    }

    printf("%lu %d\n",t,limit);
    return 0;
}
{% endhighlight %}

This solves 148 in about 10s on my i7-2960XM laptop. Impressive, given that we have to account for each of the $10^9$ rows.

However, as my friend[^2] points out, this could be simplified greatly. Take a look at $10^9$ in base 7:

$$10^9_{10} = 33531600616_7$$

If our base 7 number was actually $30000000000$, then all would need to do is calculate $T_3$[^4], then multiply by $28^n$, where $n$ is the number of digits after the digit in question (in this case, there are 10 following digits). The $28$ comes from $T_7$, which arises from each digit effectively contributing the sum of $1$ to $7$. Thus, if 148 posed the problem for $30000000000_7$ rows, the answer would be $T_3\times28^{10}=1777180600172544$.

However, our problem is slightly more complicated, as there are other digits. Move onto the next one:

$$33000000000_7$$

We add our previous result to this one ($T_3\times28^{9}$), but we need to incorporate the fact that we've "added onto" the most significant digit. This is done by simply multiplying this digit's contribution by all previous digits plus 1, like so:

$$ 30000000000_7 \rightarrow \text{count} = T_3\times 28^{10} $$

$$ \text{product} = (3+1) $$

$$ 33000000000_7 \rightarrow \text{count += } 4\times T_3\times28^{9} $$

$$ \text{product *= }(3+1) $$

$$ 33500000000_7 \rightarrow \text{count += } 16\times T_5\times28^{8} $$

$$ \text{product *= }(5+1) $$

For fun, I wrote this in Go:

{% highlight go linenos=table %}
package main

import (
	"fmt"
	"os"
	"strconv"
	"math"
)

func triangular(n uint) float64 {
    return float64((n+1)*n/2)
}

func main() {
	var limit, index float64
	switch len(os.Args) {
	// No command line arguments - defaults
	case 1:
		limit = math.Pow10(9)
		index = 7.

	// limit and index specified manually
	case 3:
		var err error
		limit, err = strconv.ParseFloat(os.Args[1],64)
		if err != nil {
			fmt.Println(err)
		}
		index, err = strconv.ParseFloat(os.Args[2],64)
		if err != nil {
			fmt.Println(err)
		}

	default:
		panic("Incorrect amount of arguments used.")
	}

	// Convert our "limit" to base "index", saving into "m"
	// The most significant bit is the last element
	var m []uint
	reduced := limit
	for i := 0; reduced > 0; i++ {
		m = append(m, uint(math.Mod(reduced,index)))
		reduced = math.Floor(reduced/index)
	}

	var count uint = 0
	var product uint = 1
	for i := range m {
		// "n" is the reverse index of the slice
		n := len(m)-i-1
		count += product * uint(triangular(m[n]) * math.Pow(triangular(uint(index)),float64(n)))
		product *= m[n]+1
	}

	fmt.Printf("%v %v\n", count, limit)
}
{% endhighlight %}

This solves 148 virtually instantly. Satisfying my functional craving, Haskell:

{% highlight haskell linenos=table %}
import System.Environment
import Data.Digits

convertBase :: Integral a => a -> a -> [a] -> [a]
convertBase from to = digits to . unDigits from

triangular :: Integral a => a -> a
triangular n = (n+1)*n`div`2

main :: IO()
main = print $ sum $ zipWith3 (\x y z -> x*y*z) triangles products weights
    where baseConverted = convertBase 10 7 [1000000000]
          products = init $ [1] ++ scanl1 (*) (map (+1) baseConverted)
          triangles = map triangular baseConverted
          weights = reverse [ 28^x | x <- [0..n-1] ]
              where n = length baseConverted
{% endhighlight %}

Again, runs virtually instantly.

* * *
[^1]: I usually compile Haskell code with something like: `ghc -O2 --llvm <code.hs>`
[^2]: Thanks Dave Horsley!
[^3]: Lucas' theorem - [Paper](http://copper.math.buffalo.edu/urgewiki/uploads/Literature2010Carbonara/Fine47.pdf) and [Wikipedia article](https://en.wikipedia.org/wiki/Lucas%27_theorem).
[^4]: $T_n$ is the $n^{th}$ [triangular number](http://en.wikipedia.org/wiki/Triangular_number).
