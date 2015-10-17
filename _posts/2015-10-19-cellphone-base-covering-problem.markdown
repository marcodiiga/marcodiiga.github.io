---
layout: post
title:  "Cellphone base covering problem"
tags: algorithms dynamic-programming
---

> Given the following data:
>
> * An array representing the positions of N houses along a line
>
> * An array representing the position of M suitable places along that line where a cellphone base transmitter could be placed 
>
> * An array representing the respective M costs to build up and place a base transmitter
>
> * The radius R of coverage for a base
>
> Determine the minimum cost to cover all the houses with cellphone network or report if no solution could be found

Let us take some example data

    houses = {  0,   2,   4  }
    bases  = {  1,   2,   3  }
    costs  = {  10,  100, 20 }
    R = 1

The data above can be represented with the following 1-dimensional graph

![image](/images/posts/cellphonebasecovering1.png)

In the image above there are three houses at positions 0, 2 and 4. There are also three bases at positions 1, 2 and 3. Each one has a radius of `R = 1` to cover other places and an associated cost to build and set up.

Working towards the optimal solution can be done recursively by considering a new house each time. A precondition to the algorithm is that both the houses and the bases should be ordered (and their costs along with them).

Finding the minimum cost for the first house is easy: there is no previous solution with 0 houses so we need to choose the minimum-cost base which also covers the first house.
Adding the second house means either

1. The previous solution has this new house in range, therefore it's the best match we could find
2. The previous solution cannot be used (insufficient range) - find a new minimum-cost base which can cover this house
3. The previous solution cannot be used and we found no bases with the house in range - *problem is unsolvable with the given data*

The algorithm follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

pair<bool, int> setRadioBases(vector<int>& houses,
                              vector<int>& bases,
                              vector<int>& costs,
                              const int R) 
{
  {
    // Sort the data in ascending order
    sort(houses.begin(), houses.end());
    // Sort bases and costs based on bases
    vector<pair<int, int>> baseDataAndIndex(bases.size());
    for (int i = 0; i < bases.size(); ++i)
      baseDataAndIndex[i] = make_pair(bases[i], i);
    sort(baseDataAndIndex.begin(), baseDataAndIndex.end());
    vector<int> sortedCosts(costs.size());
    for (int i = 0; i < baseDataAndIndex.size(); ++i) {
      bases[i] = baseDataAndIndex[i].first;
      sortedCosts[i] = costs[baseDataAndIndex[i].second];
    }
    costs = std::move(sortedCosts);
  }

  // dp[i] = minimum cost to cover i houses
  vector<int> dp(houses.size() + 1, numeric_limits<int>::max());
  // lastBase[i] = lastBase to cover i houses
  vector<int> lastBase(houses.size() + 1, -1);
  // Set up base cases
  dp[0] = 0;
  lastBase[0] = -1;

  for (int i = 1; i <= houses.size(); ++i) {
    int optCost = dp[i - 1];
    int optLastBase = lastBase[i - 1];

    // Check if previous solution has house in range
    if (optLastBase != -1 && // Previous solution is valid
      (houses[i - 1] <= bases[optLastBase] + R && 
       houses[i - 1] >= bases[optLastBase] - R)) {
      dp[i] = optCost; // Previous solution is preferable
      lastBase[i] = optLastBase;
    }
    else {
      // Previous solution not feasible, try to add a new base which
      // minimizes the cost to cover this house
      optCost = numeric_limits<int>::max();
      optLastBase = -1;
      for (int b = optLastBase + 1; b < bases.size(); ++b) {
        if (bases[b] - R > houses[i - 1])
          break; // No longer in range
        if (bases[b] - R >= houses[i - 1] || bases[b] + R >= houses[i - 1]) {
          if (costs[b] < optCost) {
            optCost = costs[b];
            optLastBase = b;
          }
        }
      }
      if (optCost == numeric_limits<int>::max()) {
        // No previous/new solution can cover this house
        return make_pair(false, -1);
      }
      dp[i] = dp[i - 1] + optCost; // Add new solution
      lastBase[i] = optLastBase;
    }
  }
  return make_pair(true, dp[houses.size()]);
}

int main() {

  vector<int> houses = { 0, 2, 4 };
  vector<int> bases = { 1, 2, 3 };
  vector<int> costs = { 10, 100, 20 };
  int R = 1;

  auto res = setRadioBases(houses, bases, costs, R);
  if (res.first == false)
    cout << "No solution found" << endl;
  else
    cout << "Minimum cost: " << res.second << endl; // 30

  return 0;
}
{% endhighlight %}

The algorithm runs in \\( O(NM) \\) and has a preprocessing step of respectively \\( O(N\log{N}) \\) and \\( O(M\log{M}) \\).