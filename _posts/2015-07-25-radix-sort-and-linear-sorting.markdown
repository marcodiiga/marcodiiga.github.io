---
layout: post
title:  "Radix sort and linear sorting"
tags: algorithms
---

[Radix sort](https://en.wikipedia.org/wiki/Radix_sort) is a stable sorting algorithm based
on iteratively *bucketing* elements by their individual digits. Radix sort requires
a positional notation and under some circumstances can be an efficient sorting algorithm
but due to its tradeoffs and efficiency considerations programmers often mark it as
a 'second-class' algorithm.

Given an array of integers

$$ A = \{ 0, 10, 102, 13, 193, 12, 7 \} $$

a classic LSD (least significant digit) radix sort will first order the elements
by their least significant digit. This is usually achieved through counting sort.

Counting sort
=============

[Counting sort](https://en.wikipedia.org/wiki/Counting_sort) is a sorting algorithm
that groups input integers into *buckets* in the same fashion as a hash table would
and, as its name suggests, counts how many times a value is encountered.

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <numeric>
using namespace std;

void countSort(vector<int>& vec) {

  vector<int> counts(200, 0);

  for (auto& v : vec)
    ++counts[v];

  partial_sum(counts.begin(), counts.end(), counts.begin());

  vector<int> output(vec.size());

  for (auto& v : vec) {
     output[counts[v] - 1] = v;
     --counts[v];
  }

  vec = std::move(output);
}

int main() {
  vector<int> arr = { 0, 10, 102, 13, 193, 12, 7 }; // All data in range 0 - 200

  countSort(arr);

  return 0;
}
{% endhighlight %}

It has to be noted that counting sort requires knowledge of the data range in advance,
i.e. of the range in which input elements span in their value. If the variation
between the keys and the number of items isn't huge, counting sort might offer
an advantageous \\( O(n + k) \\) complexity where \\( n \\) is the number of elements
and \\( k \\) is their value range. Space complexity is also \\( O(n+k) \\) since
it uses additional arrays to do sorting and counting.

Stable bucketing by digit
=========================
As said before, radix sort often resorts to counting sort to sort elements based
on their *i-th* digit since the range is fixed to the positional notation base
the elements are referring to.

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <numeric>
using namespace std;

void countSort(vector<int>& vec, int base, int digit);

void radixSort(vector<int>& vec, const int base, const int range) {
  const int digits = static_cast<int>(ceil(log10(range) / log10(base)));

  for (int i = 0; i < digits; ++i) {
    countSort(vec, base, i);
  }
}

void countSort(vector<int>& vec, int base, int digit) {

  vector<int> counts(base, 0);

  for (auto v : vec)
    ++counts[static_cast<int>(v / pow(base, digit)) % base];

  partial_sum(counts.begin(), counts.end(), counts.begin());

  vector<int> output(vec.size());

  // Nb. this has to be done in reverse to maintain the algorithm correct and stable
  for (int i = static_cast<int>(vec.size()) - 1; i >= 0; --i) {
    output[counts[static_cast<int>(vec[i] / pow(base, digit)) % base] - 1] = vec[i];
    --counts[static_cast<int>(vec[i] / pow(base, digit)) % base];
  }

  vec = std::move(output);
}

int main() {
  vector<int> arr = { 0, 10, 102, 13, 193, 12, 7 };
  const int range = 200; // All data in range 0 - 200

  radixSort(arr, 10, range);

  return 0;
}
{% endhighlight %}

A word of advice: the sorting performed by counting sort will always be correct
with the algorithm seen in the first part, anyway radix sort requires traversing
the input vector in reverse order when accounting for the position given by the
`counts` vector. This is necessary for both correctness and radix sort stability.

Radix sort complexity is \\( O(w \cdot N) \\) where \\( w \\) stands for the *word size*.
If all keys are distinct \\( w \\) has to be at least \\( \log_2{n} \\) and that
yields a complexity of \\( O(n \log_2{n}) \\) comparable to other sorting algorithms.

Tweaking the base for a faster sorting
--------------------------------------
Things get interesting when we're able to tweak the base of the input numbers and
perform radix sort on the digits of another base to save time.

Suppose we had \\( n \\) numbers in range \\( [0;n^2-1] \\). Using counting sort
on such a range would yield \\( O(n^2) \\) and any other standard sorting algorithm
would yield \\( O(n \log{n}) \\).

On the other hand, radix sort operates on digits of a positional notation.. and
those depend on the base representation of the numbers. The number of digits
required to store a number \\( n \\) in a given base \\( b \\) is

$$ d = \log_{b}{n} $$

and thus radix sort complexity in a given base is

$$ O(n \cdot \log_{b}{n}) $$

that means choosing a base \\( b = n \\) will be \\( O(n) \\). The counting sort
performed when bucketing digits will also be in range \\( O(n) \\).

We can modify the driver `main()` function of the previous program to account
for this change (n.b. make sure all data is in the range \\( [0;n^2-1] \\))

{% highlight c++ %}
int main() {
  vector<int> arr = { 0, 10, 44, 13, 2, 12, 7 };
  const int range = static_cast<int>(arr.size()) *
                    static_cast<int>(arr.size()) - 1; // All data in range 0 - n^2-1

  radixSort(arr, static_cast<int>(arr.size()), range);

  return 0;
}
{% endhighlight %}

For an interesting *exponentiation by squaring* algorithm take a look at [this other
post]({% post_url 2015-07-22-integer-arithmetic-using-bit-operations %}).

Also notice that this isn't a silver-bullet to linearize all kind of sort problems
since memory requirements and operations on large integers will probably take a
significant toll enough to justify a classic reliable approach or, with large data,
an external sorting algorithm.

References
==========

* [Wikipedia.org](https://wikipedia.org)
* [geeksforgeeks.org](http://www.geeksforgeeks.org/)
