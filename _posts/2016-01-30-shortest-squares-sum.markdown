---
layout: post
title:  "Shortest squares sum"
tags: algorithms dynamic-programming
---

> Find and output the given number's shortest square sum, i.e. the shortest sequence of numbers whose squares summed together
> give back the input number

An example case: given \\( N = 12 \\), the *SSS* (short for shortest squares sum) is

$$ SSS = \{ 2, 2, 2 \} $$

since

$$ 2^2 + 2^2 + 2^2 = N $$

This is a [minimum coin change problem]({% post_url 2015-10-06-minimum-coin-change %}) in disguise and can be therefore formulated as follows:
let \\( D(N, S) \\) be the minimum number of coins needed among \\( N \\) ones to get \\( S \\) by squaring and adding them up. The recursion
can be expressed as

$$ D(n,s) = min\{ D(n, s - n^2) + 1, \ D(n-1,s) \} $$

and base cases

$$
   \begin{align}
   D(n, 0) & = 0 \\
   D(0, s) & = \infty 
   \end{align}
$$

Code to solve the problem in a dynamic programming fashion and backtrack the solution path follows

{% highlight c++ %}
// -------------------------------------------------------------------
// Find and output the given number's shortest square sum.
// e.g. for sum == 12, the shortest square sum is made of 3 values only
// i.e. 2^2+2^2+2^2 (not 3^2+1^2+1^2+1^2) so the output is: {2 2 2}  
// -------------------------------------------------------------------

#include <iostream>
#include <vector>
#include <limits>
#include <tuple>
#include <cmath>
#include <algorithm>

// A predecessor node. Identifies a D(n,s) cell and stores
// whether the n-th coin was used in a solution path
struct PredecessorNode {
  PredecessorNode(bool b, int n, int s) :
    taken(b),
    predecessorN(n),
    predecessorS(s)
  {}
  bool taken;
  int predecessorN;
  int predecessorS;
};

int pow(int x, int y) { // Exponentiation by squaring
  if (y == 0)
    return 1;
  else
    return pow(x*x, y >> 1) * ((y % 2 != 0) ? x : 1);
}

bool findSSS(int sum, std::vector<int>& res) {

  // By taking the N squares of only the numbers that will NOT be greater than the sum,
  // i.e. [1;ceil(sqrt(sum))], it's a min-coin-change problem. 
  // This can be solved via dynamic programming. Let D(n, s) be the MINIMUM number
  // of coins needed among n ones we have to make sum s. The recursion is
  //
  //  D(n,s) = min{ D(n, s - i^2) + 1 , D(n-1, s) }
  //                  ^             ^   ^ we don't use the coin here
  //                  |             |
  //                  |             |one coin more
  //                  |
  //                  - not n-1 since once used a coin can be reused
  // 
  // Base cases, or borders to the giant 2D matrix, are
  // D(0, s) = infinity (i.e. can't reach s with no coins, so we provide an infinity
  //                     value which is not desirable)
  // D(n, 0) = 0 (no coins needed to have a 0 sum)

  //               s
  //   -------------------------
  //   |
  // N |
  //   |  
  const int INF = std::numeric_limits<int>::max();
  const int N = static_cast<int>(ceil(sqrt(sum)));
  std::vector<std::vector<int>> dp(N + 1 /* To have it 1-index based */, 
                                   std::vector<int>(sum + 1 /* ditto */, INF));
  std::vector<std::vector<PredecessorNode>> predecessors(N + 1, 
              std::vector<PredecessorNode>(sum + 1, PredecessorNode{ false, 0, -1 }));

  // Base cases
  for (auto& v : dp)
    v[0] = 0;
  // D(0, s) already in place due to initialization


  // D(i,j)
  for (int i = 1; i <= N; ++i) {
    for (int j = 1; j <= sum; ++j) {

      int squared_element = static_cast<int>(pow(i, 2)); // This could be precomputed
      int previous_N = j - squared_element;

      if (previous_N < 0) {
        // This value cannot be used, set the predecessor to D(n-1, s)
        int rv = dp[i - 1][j];
        predecessors[i][j] = PredecessorNode{ false, i - 1, j };
        dp[i][j] = rv;
        continue;
      }

      int lv = dp[i][previous_N] + 1;
      int rv = dp[i - 1][j];

      if (lv < rv) {
        predecessors[i][j] = PredecessorNode{ true, i, previous_N };
        dp[i][j] = lv;
      }
      else {
        predecessors[i][j] = PredecessorNode{ false, i - 1, j };
        dp[i][j] = rv;
      }

    } // j
  } // i

  // After everything has been calculated, check if D(n,s) is not INF (i.e. there's
  // a min sequence of squares)
  if (dp[N][sum] >= INF)
    return false;

  // Reconstruct the shortest squares sequence
  int i = N;
  int j = sum;
  std::vector<int> result;
  while (true) {

    if (predecessors[i][j].taken == true)
      result.push_back(i);

    std::tie(i, j) = std::make_tuple(predecessors[i][j].predecessorN, 
                                     predecessors[i][j].predecessorS);

    if (i <= 0 || j < 0)
      break;
  }

  std::reverse(result.begin(), result.end());
  res = result;
  return true;
}


int main() {

  int sum = 45;
  std::vector<int> result;
  bool found = findSSS(sum, result); // {3, 6}

  if (found) {
    for (auto& v : result)
      std::cout << v << " ";
  }
  std::cout << std::endl;

  return 0;
}
{% endhighlight %}

The solution is \\( O(N^{\frac{3}{2}}) \\) since it features the same square optimization seen in the [primes generation]({% post_url 2015-09-10-primes-generation %}) article.