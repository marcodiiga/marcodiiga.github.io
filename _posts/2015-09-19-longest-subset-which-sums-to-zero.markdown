---
layout: post
title:  "Longest subset which sums to zero"
tags: algorithms dynamic-programming
---

> Given a sequence of numbers determine the longest subset which sums to zero

This problem should not be confused with the [maximum zero-sum subarray problem]({% post_url 2015-09-16-maximum-and-maximum-zero-sum-subarray-problems %}): a **subset** might include non-contiguous elements. A simple linear hash-scan would not suffice here.

It has to be noted that this problem is a variation on the classic [subset sum problem]({% post_url 2015-07-23-knapsack-problem %}) where the maximum-length subset has to be found.

The general idea is to set up a typical dynamic programming recursion exploring the substates of sums that can be obtained by repeatedly adding one more element of the sequence

Let \\( d(x) \\) be the maximum length of the subset that sums to \\( x \\) and \\( V \\) be the input sequence of values

$$  
  d(x+v) = max\{ d(x+v), d(x)+1\} \forall \ v \in V
$$

In pseudocode

    sums_that_can_be_formed = (0 with 0 elements)
    for each element EL of the sequence
      for each sum S in sums_that_can_be_formed
        int newSum = (S + EL with the elements that formed S + 1)
        if newSum is not in the array OR
           newSum is formed by more elements
           add_or_update(sums_that_can_be_formed, (S+EL, elements_of_S+1)


{% highlight c++ %}
#include <algorithm>
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

int maximumZeroSubsequenceLength(const vector<int>& vec) {
  unordered_map<int, int> d1, d2;
  d1.emplace(0, 0); // 0 values sum to 0 at the beginning
  d2 = d1;
  for (auto& element : vec) {
    for (auto& subsum : d1) {
      int opt1 = 0;
      auto it = d1.find(subsum.first + element);
      if (it != d1.end())
        opt1 = it->second;
      d2[subsum.first + element] = max(opt1, d1[subsum.first] + 1);
    }
    d1 = d2;
  }
  return d1[0];
}

int main(int argc, char * argv[]) {

  vector<int> vec = { 4, -2, 2, -4 };
  cout << maximumZeroSubsequenceLength(vec); // 4

  return 0;
}
{% endhighlight %}

This algorithm runs in \\( O(nS) \\) where \\( S \\) is the sum of all numbers in the input sequence. The same algorithm can be used to calculate the maximum or minimum length of subsets which sum to a number \\( i \\) with small variations.