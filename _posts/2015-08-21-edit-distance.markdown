---
layout: post
title:  "Edit distance problem"
tags: algorithms dynamic-programming
---

The so-called [Edit distance](https://en.wikipedia.org/wiki/Edit_distance)
problem consists in finding (and possibly identifying) the minimum number of operations needed to transform a string into another one. A given set of weighted operations is given. Common operations include

- Insertion of a character at a specific position
- Deletion of a character at a specific position
- Change of a character with another one

A recursive exponential-time solution would compare each operation at any given character of the input strings and choose the minimum cost one as the operation to be performed in that step.

The complexity of the recursive approach is bounded by \\( min(m,n) \\) where \\( m,n \\) are the sizes of the input strings. Complexity of the recursion

$$ f(m, n) = f(m-1, n-1) + f(m, n-1) + f(m-1, n) + k $$

where \\( k \\) is a constant, is the complexity of a call tree of branching factor 3 and maximum depth \\( min(m,n) \\). Therefore we have \\( \Theta(3^{min(m,n)}) \\) with worst case of \\( m = n \\) so actually \\( \Theta(3^n) \\). It has to be noted that the previous complexity doesn't take into account any amount of work that might be necessary for example to track the series of operations needed.

Wagner-Fischer algorithm
------------------------

The [Wagner-Fischer](https://en.wikipedia.org/wiki/Wagner%E2%80%93Fischer_algorithm) algorithm is a dynamic programming algorithm suited to calculate the edit distance between two vectors.

If we consider the lengths of the input vectors \\( a,b \\)as \\( m,n \\) and we construct a matrix storing the minimum number of operations needed to *morph* a subset of an input vector into another subset of the other vector

$$ \begin{bmatrix}
    x_{11} & x_{12} & x_{13} & \dots  & x_{1n} \\
    x_{21} & x_{22} & x_{23} & \dots  & x_{2n} \\
    \vdots & \vdots & \vdots & \ddots & \vdots \\
    x_{d1} & x_{m2} & x_{m3} & \dots  & x_{mn}
\end{bmatrix} $$

we can proceed by iteratively constructing the matrix considering at each step the cost of an insertion, deletion or modification of a character. During the calculation for cell \\( x_{22} \\) for instance, with respect to the abscissa being the *expected* string and the ordinate being the *received* string, the value of cell \\( x_{21} \\) is considered with the addition of a deletion operation, the value of cell \\( x_{12} \\) is considered with the addition of an insertion operation and the value of cell \\( x_{11} \\) is considered with the addition of a modification operation \\( \iff  a_1 \ne b_1 \\).

The general mathematical formulation follows: the edit distance between \\( a = a_1\ldots a_n \\) and \\( b = b_1\ldots b_m \\) is given by \\( d_{mn} \\), defined by the recurrence

$$ \begin{align}d_{i0} &= \sum_{k=1}^{i} w_\mathrm{del}(b_{k}), & & \quad  \text{for}\; 1 \leq i \leq m \\
d_{0j} &= \sum_{k=1}^{j} w_\mathrm{ins}(a_{k}), & & \quad \text{for}\; 1 \leq j \leq n \\
d_{ij} &= \begin{cases} d_{i-1, j-1} & \text{for}\; a_{j} = b_{i}\\ \min \begin{cases} d_{i-1, j} + w_\mathrm{del}(b_{i})\\ d_{i,j-1} + w_\mathrm{ins}(a_{j}) \\ d_{i-1,j-1} + w_\mathrm{sub}(a_{j}, b_{i}) \end{cases} & \text{for}\; a_{j} \neq b_{i}\end{cases} & & \quad  \text{for}\; 1 \leq i \leq m, 1 \leq j \leq n.\end{align} $$

The \\( O(mn) \\) dynamic programming solution follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <string>
using namespace std;

const int INSERTION_COST = 1;
const int DELETION_COST = 1;
const int CHANGE_COST = 1;

template<typename T>
T min(const T& e1, const T& e2, const T& e3) {
  if (e1 <= e2 && e1 <= e3)
    return e1;
  else if (e2 <= e1 && e2 <= e3)
    return e2;
  else
    return e3;
}

int minEditDistanceDP(const string& str1, const string& str2) {

  size_t M = str1.size();
  size_t N = str2.size();
  //              str2 - n
  //           |------------
  //  str1 - m |
  //           |
  // m[m][n]
  vector<vector<int>> m(M + 1, vector<int>(N + 1, 0));

  // Initialize base cases
  for (int i = 0; i < M + 1; ++i)
    m[i][0] = i;
  for (int i = 0; i < N + 1; ++i)
    m[0][i] = i;

  for (int y = 1; y < M + 1; ++y) {
    for (int x = 1; x < N + 1; ++x) {
      int left = m[y][x - 1];
      left += DELETION_COST;

      int top = m[y - 1][x];
      top += INSERTION_COST;

      int topLeft = m[y - 1][x - 1];
      if (str1[y - 1] != str2[x - 1])
        topLeft += CHANGE_COST;

      m[y][x] = min(left, top, topLeft);
    }
  }

  return m[M][N];
}

int main() {
  string str1 = "hectagon";
  string str2 = "etthagon";

  cout << minEditDistanceDP(str1, str2);

  return 0;
}
{% endhighlight %}
