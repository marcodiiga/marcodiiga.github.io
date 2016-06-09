---
layout: post
title:  "Count matrix paths from TopLeft to BottomRight"
tags: algorithms dynamic-programming
---

> Write an efficient algorithm to count all possible paths in a \\( (m;n) \\) matrix to move from the top left cell to the bottom right one. The only movements allowed are
right and down. Time complexity should be \\( O(mn) \\).

Solution I
==========

Trivial solution: backtracking in \\( O(2^{max(m,n)}) \\)

{% highlight c++ %}
int uniquePaths(const int M, const int N) {
  function<int(int,int)> countPaths = [&](int start_x, int start_y) {
    if (start_x >= N || start_y >= M)
      return 0;
    if (start_x == N - 1 && start_y == M - 1)
      return 1;
    return countPaths(start_x + 1, start_y) + countPaths(start_x, start_y + 1);
  };
  return countPaths(0, 0);
}
{% endhighlight %}

While this solution proves to be extremely easy to implement, it is also extremely inefficient.

Solution II
===========

Realizing that many subproblems are recomputed multiple times in the previous approach and that right/bottom deltas can be considered as valid path increments (*optimal substructure*),
leads us to a better approach with dynamic programming

{% highlight c++ %}
int uniquePaths(const int M, const int N) {
  vector<vector<int>> dp(M, vector<int>(N, 0));

  for(int i = 0; i < M; ++i)
    dp[i][0] = 1;
  for(int i = 0; i < N; ++i)
    dp[0][i] = 1;

  for(int i = 1; i < M; ++i) {
    for(int j = 1; j < N; ++j) {
      dp[i][j] = dp[i-1][j] + dp[i][j-1];
    }
  }

  return dp[M-1][N-1];
}
{% endhighlight %}

Complexity is \\( O(mn) \\).

Solution III
============

A third more complex but definitely more performant approach lies in the following observation. Let us consider the allowed paths that can reach a cell for a 3x3 matrix

$$ \left[
    \begin{array}{ccc}
      0&1&1\\
      1&2&3\\
      1&3&6
    \end{array}
\right] $$

The diagonals formed by bottom and left deltas for the \\( (m;n) \\) element are the same values in the \\( (m;n) = (n;k) \\) row of [Pascal's triangle](https://en.wikipedia.org/wiki/Pascal%27s_triangle), therefore
the following holds

$$
{n \choose k} = {n-1 \choose k-1} + {n-1 \choose k}
$$

for any non-negative integer \\( n \\) and any integer \\( k \\) between 0 and \\( n \\).

It can be trivially demonstrated that

$$
{n \choose k} = {n \choose k-1} \frac{n-k+1}{k}
$$

therefore a recursive relation holds for which the base conditions are the unit elements on the fringe

{% highlight c++ %}
int uniquePaths(const int M, const int N) {
  if (M == 0 || N == 0)
    return 0;
  if (M == 1 || N == 1)
    return 1;

  int Nc = (M - 1) + (N - 1); // (*)

  int C = 1;
  for (int k = 0; k < N - 1; ++k)
    C = C * (Nc - k) / (k + 1); // (*)
  return C;
}
{% endhighlight %}

The parts marked with `(*)` require careful consideration to get the indices right (since we loop in the range \\( [0;N-2] \\) by skipping the first base case 1), and by considering that matrices for which holds \\( m \neq n \\)
need the `Nc = (M - 1) + (N - 1)` row of the triangle and `2N` (to get the last \\( k \\)), but since we're interested in going halfway through the \\( k \\) arrangements reduces to `N - 1`.

Final complexity is \\( O(max(n,m)) \\) and due to the triangle symmetry it also works for \\( m < n \\).