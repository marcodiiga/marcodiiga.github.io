---
layout: post
title:  "Generating integer partitions"
tags: algorithms
---

> Given a number N generate all the integer partitions for that number

Generating integer partitions for a number \\( N \\) is an ancient problem already studied by many mathematicians. In particular, being \\( p(n) \\) the partition function representing the number of possible partitions of a natural number \\( n \\), Euler generalized to an arbitrary set \\( A \\) the following [generating function](https://en.wikipedia.org/wiki/Generating_function)

$$ \sum_{n=0}^\infty p(n)x^n = \prod_{k=1}^\infty \left(\frac {1}{1-x^k} \right) $$

If we expand the right-hand side of this expression as [geometric series](https://en.wikipedia.org/wiki/Geometric_series#Formula) we have

$$ \sum_{n=0}^\infty p(n)x^n = (1 + x + x^2 + x^3 + \dotsc) (1 + x^2 + x^4 + x^6 + \dotsc) (1 + x^3 + x^6 + x^9 + \dotsc)  $$

If we were to just consider the terms that make up \\( p(3) \\), we would get

$$ p(3)x^3 = x^3 + x^1 \cdot x^2 + x^3$$

A simple algorithm to generate all integer partitions for a given \\( N \\) (cfr. [Generating all partitions: a comparison of two encodings](http://arxiv.org/abs/0909.2331) by Jerome Kelleher, Barry O'Sullivan) is based on a two-phase iteration. The algorithm proceeds by accumulating towards the left side the digits of a partition. Expansion into multiple smaller values (possibly all 1 values) happen before an accumulation if an ascending sequence of two digits is found during this process.

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <iterator>
using namespace std;

void printIntegerPartitions(const int N) {
  vector<int> v(N + 1);
  v[1] = N; // Initialize - ready to be broken up into subparts
  int k = 1;
  while (k != 0) {
    // Accumulate one unit from right to left
    int x = v[k - 1] + 1;
    int y = v[k] - 1;
    --k;
    // If the sequence [k-1;k] is not descending
    // break down y into y/x units of length x
    while (x <= y) {
      v[k] = x;
      y -= x;
      ++k;
    }
    v[k] = x + y;
    copy(v.begin(), v.begin() + k + 1, ostream_iterator<int>(cout, " "));
    cout << endl;
  }
}

int main() {
  
  printIntegerPartitions(3);

  return 0;
}
{% endhighlight %}

Output

    1 1 1
    1 2
    3

The algorithm is well-suited to be implemented as a stateful [generator function](https://en.wikipedia.org/wiki/Generator_(computer_programming)) and in that case has an amortized \\( O(1) \\) time.