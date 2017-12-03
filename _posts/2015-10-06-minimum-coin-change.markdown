---
layout: post
title:  "Minimum coin change"
tags: algorithms dynamic-programming
---

[Minimum coin change](https://en.wikipedia.org/wiki/Change-making_problem), also referred as *Change making problem*, is a [knapsack type problem]({% post_url 2015-07-23-knapsack-problem %}) which states

> Given a set S of coins of given values and an amount of money N, find the minimum number of coins required to obtain N

E.g. given the input data

    S = {1, 5, 10, 50}
    N = 167

the minimum number of coins needed is 7 (50+50+50+10+5+1+1 = 167).

More formally given an amount \\( W \ge 0 \\), find the set of non-negative integer values {\\( x_1, x_2, \dots , x_n \\)} with each \\( x_j \\) representing how many times the coin with value \\( w_j \\) is used for which

$$ \sum^{n}_{j=1}{x_j} \\
\mbox{s.t.} \\
\sum^{n}_{j=1}{w_j x_j} = W$$

The problem exhibits *optimal substructure* and *overlapping subproblems* properties and is therefore a good candidate to be solved with dynamic programming

{% highlight c++ %}
#include <iostream>
#include <vector>
using namespace std;

int minCoinChange(const vector<int>& coins, const int N) {
  vector<int> nOfCoinsForSum(N + 1, numeric_limits<int>::max());
  nOfCoinsForSum[0] = 0; // Base case
  for (int n = 1; n <= N; ++n) {
    for (int i = 0; i < coins.size(); ++i) {
      if (coins[i] > n)
        continue;
      int newCoinsCount = nOfCoinsForSum[n - coins[i]] + 1;
      if (newCoinsCount < nOfCoinsForSum[n])
        nOfCoinsForSum[n] = newCoinsCount;
    }
  }
  return nOfCoinsForSum[N];
}

int main() {
  vector<int> coins = { 1, 5, 10, 50 };
  const int sum = 167;
  cout << minCoinChange(coins, sum);
  return 0;
}
{% endhighlight %}

The code runs in \\( O(NC) \\) where \\( C \\) is the number of coins in the set.
