---
layout: post
title:  "Dice throw problem"
tags: algorithms dynamic-programming
---

There are many common problems regarding dice throwing; one of these asks to find the number of times a number \\( X \\) is extracted from \\( n \\) dice each with numbered faces in the range \\( [1;m] \\).

E.g. given two dice (\\( n=2 \\)) each one with faces from 1 to 4 (\\( m=4 \\)) the number of times we can obtain the number 4 by throwing both of them is 3 i.e.

$$ \{ 1,3 \} \\ 
   \{ 2,2 \} \\
   \{ 3,1 \} \\
$$

A brute-force method would be to backtrack or generate all the possible permutations with repetition in \\( O(m^n) \\). This approach quickly becomes unfeasible. 

A faster approach is via dynamic programming. The key observation is that the number of ways a sum can be formed with \\( n \\) dice can be obtained as a function of \\( n-1 \\) dice. Let \\( sum(n,X) \\) be the number of ways a value \\( X \\) can be obtained by launching \\( n \\) dice each with \\( m \\) faces (let's keep \\( m \\) out of the expression for clarity's sake). It follows that

$$ 
sum(2,4) = sum(1,3) + sum(1,2) + sum(1,1) + sum(1,0)
$$

The first recursive occurrence considers the \\( n^{th} \\) die as having been extracted with value 1. The second considers it as having been extracted with value 2 and so on. Therefore

$$
\begin{align}
sum(2,4) & = sum(1,3) + sum(1,2) + sum(1,1) + sum(1,0) = \\
         & = 1 + 1 + 1 + 0 = 3
\end{align}
$$

and also

$$
\begin{align}
sum(3,4) & = sum(2,3) + sum(2,2) + sum(2,1) + sum(2,0) \\
sum(2,3) & = sum(1,2) + sum(1,1) + sum(1,0) \\
\cdots
\end{align}
$$

The bottom-up construction dynamic programming algorithm follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int countDiceWays(const int m, const int n, const int X) {
  vector<vector<int>> sums(n + 1, vector<int>(X + 1, 0));

  // Initialize base case with 1 dice. X might be more than
  // the maximum number reachable with the faces
  for (int x = 1; x <= min(m, X); ++x) {
    ++sums[1][x];
  }

  for (int d = 2; d <= n; ++d) {
    for (int to_X = 1; to_X <= X; ++to_X) {
      int nSums = 0;
      for (int x = 1; x <= m && to_X - x >= 0; ++x) {
        nSums += sums[d - 1][to_X - x];
      }
      sums[d][to_X] = nSums;
    }
  }

  return sums[n][X];
}


int main() {
  
  cout << countDiceWays(4, 2, 4) << endl;

  return 0;
}
{% endhighlight %}

The dynamic programming algorithm version is \\( O(nm^2) \\).