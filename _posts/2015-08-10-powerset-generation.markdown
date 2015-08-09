---
layout: post
title:  "Powerset generation"
tags: algorithms
---

In mathematics a [power set](https://en.wikipedia.org/wiki/Power_set) of any set \\( S \\), written \\( \mathcal{P}{(S)} \\) is the set of all subsets of \\( S \\) including the empty set and \\( S \\) itself. The size of the power set for a set \\( S \\) of 3 elements is \\( 2^{3} \\).

The simplest and most intuitive algorithm to generate the power set given a set of elements iteratively loops from 0 to \\( 2^{N}-1 \\) and considers the high bits of the index variable as the elements of a subset.

{% highlight c++ %}
vector<set<int>> findAllSubsetsBit(set<int>& s) {
  int N = pow(2, s.size());
  int mask = -1;
  vector<set<int>> result;
  for (int i = 0; i < N; ++i) {
    mask += 1;
    set<int> sub;
    int index = 0;
    int bits = mask;
    while (bits > 0) {
      if (bits & 0x1 == 0x1) {
        auto it = s.begin();
        advance(it, index);
        sub.insert(*it);
      }
      ++index;
      bits >>= 1;
    }
    result.insert(result.end(), sub);
  }
  return result;
}
{% endhighlight %}

Complexity of the given code is \\( O(N 2^{N}) \\). Another \\( O(N 2^{N}) \\) approach works by exploiting the recursive formula for the binomial coefficients

$$ \binom nk = \binom{n-1}{k-1} + \binom{n-1}k $$

Anyway implementing this latter algorithm proves to be considerably more complicated and thus the method above is preferable.

A faster approach at generating a powerset is through [gray codes]({% post_url 2015-07-30-gray-code-subsets %}).