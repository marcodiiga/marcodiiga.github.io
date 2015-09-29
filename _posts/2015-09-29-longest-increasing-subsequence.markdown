---
layout: post
title:  "Longest increasing subsequence"
tags: algorithms dynamic-programming
---

> Given an input vector of numbers find the longest monotonically increasing subsequence

The [longest increasing subsequence](https://en.wikipedia.org/wiki/Longest_increasing_subsequence) problem is a common example of dynamic programming. Given an input vector \\( v \\) of \\( n \\) elements, the longest increasing subsequence can be found in exponential time by recursively evaluating all the subsequences for monotonicity and maximum length or in \\( O(n \log{n}) \\) time with a dynamic programming approach.

The key to the recursion expression lies in the following observations:

* An element greater than any previously encountered increases the length of all the previous subsequences by one
* An element which is not the greatest one encountered till this point can be used to reduce the final value of a previous subsequence

In case of an input vector \\( v = \{ 1,2,4,3 \} \\) the longest subsequence encountered till element 4 is \\( \{ 1,2,4 \} \\). When element 3 is encountered it cannot be appended since it's less than 4, but it can be used in place of 4 to increase the length of the subsequence \\( {1,2} \\) since ending with 3 instead of 4 ensures a better chance to encounter a highest value in the future (and thus increasing the length of the sequence).

Let \\( lis(i) \\) be the best-candidate terminal element for the subsequence with length \\( i \\), it follows that

$$
lis(i) = v(t)
    \begin{cases}
                v(t) > lis(i-1)\\
                v(t) = min \{ v(j) > lis(i-1), \ j < t \}
     \end{cases}
$$

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
#include <cassert>
using namespace std;

// Finds the length of the longest increasing subsequence in O(NlogN)
int longestIncreasingSubsequence(vector<int>& vec) {

  // Notice: this code assumes no duplicates
  assert(unique(vec.begin(), vec.end()) == vec.end() && "No duplicates allowed");

  // lis[i] is the last element of the sequence with size i
  vector<int> lis(vec.size(), -1);
  int lis_max = 0;
  for (int i = 0; i < vec.size(); ++i) {
    // Find the smallest value greater than vec[i] in lis
    int lo = 0;
    int hi = lis_max - 1;
    while (lo <= hi) {
      int mid = static_cast<int>(ceil((lo + hi) / 2.0f));
      if (vec[i] > lis[mid])
        ++lo;
      else
        --hi;
    }
    if (lo >= lis_max)
      ++lis_max;
    lis[lo] = vec[i];
  }
  return lis_max;
}

int main() {

  vector<int> vec = { 1,2,4,3 };
  cout << longestIncreasingSubsequence(vec); // 3

  return 0;
}
{% endhighlight %}


