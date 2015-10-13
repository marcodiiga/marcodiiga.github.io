---
layout: post
title:  "Calculating binomial coefficients"
tags: algorithms dynamic-programming
---

A [binomial coefficient](https://en.wikipedia.org/wiki/Binomial_coefficient) \\( C(n,k) \\) can be calculated in many ways. One is through the recursive formula

$$ \binom nk = \binom{n-1}{k-1} + \binom{n-1}k $$

Since this recursion exhibits optimal substructure and overlapping subproblems it is a good candidate to be solved via dynamic programming

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int binomial(const int N, const int K) {
  vector<vector<int>> C(N + 1, vector<int>(K + 1, 0));

  for (int n = 1; n <= N; ++n) {
    for (int k = 0; k <= min(n,K); ++k) {
      if(k == 0 || n == k) // Base cases
        C[n][k] = 1;
      else
        C[n][k] = C[n - 1][k - 1] + C[n - 1][k];
    }
  }

  return C[N][K];
}


int main() {

  cout << binomial(3, 2) << endl; // 3

  return 0;
}
{% endhighlight %}

Time complexity is \\( O(nk) \\). Space complexity is also \\( O(nk) \\).

