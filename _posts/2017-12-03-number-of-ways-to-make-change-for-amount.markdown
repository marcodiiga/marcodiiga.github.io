---
layout: post
title:  "Number of ways to make change for an amount (coin change)"
tags: c++ dynamic-programming algorithms
---

> Count the number of ways one can make change for an amount `N` from an infinite supply of coins of given values.
>
> E.g.
>
> ```
> coins = {1, 2, 3}
>
> N = 5
> ```
>
> The result is 5 since `N` can be obtained from the following set of solutions
>
> * 1 + 1 + 1 + 1 + 1
> * 1 + 1 + 1 + 2
> * 1 + 1 + 3
> * 1 + 2 + 2
> * 3 + 2

The problem is different from the [minimum coin change problem]({% post_url 2015-10-06-minimum-coin-change %}) since this time we're not interested in one particular solution (the one that minimizes the number of coins used) but rather in the count of the solutions itself.

The problem can be solved via dynamic programming - notice the order of the `for` loops: each time a new coin is considered, all of the subsums are updated with the new possibilities opened by it

{% highlight c++ %}
int changePossibilities(const int N, std::vector<int> coins) {
  vector<int> dp(N + 1, 0);
  dp[0] = 1; // Initialize - only one way to make 0 (with no coins)

  for (const auto c : coins) { // For each coin
    for (int i = c; i <= N; ++i) // For each subsum
      dp[i] += dp[i - c]; // Add new ways to make i
  }

  return dp[N];
}
{% endhighlight %}

The code has a \\( O(NC) \\) pseudo-polynomial complexity where \\( N \\) is the amount and \\( C \\) the number of input coins.
