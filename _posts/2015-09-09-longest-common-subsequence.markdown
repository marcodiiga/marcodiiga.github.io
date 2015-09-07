---
layout: post
title:  "Longest common subsequence"
tags: algorithms dynamic-programming
---

[Longest common subsequence](https://en.wikipedia.org/wiki/Longest_common_subsequence_problem) problem - **not to be confused** with the [longest common substring](https://en.wikipedia.org/wiki/Longest_common_substring_problem) problem - involves creating the longest common subsequence (i.e. a sequence of elements that aren't required to be laid contiguously in the input strings) between two or more input vectors.

An example is the longest common subsequence for the following vectors

    A C B A Y A B
    C B D Y B A M

The longest common subsequence is `CBYA` and has length 4. Not always the longest common subsequence is unique, sometimes there can be multiple longest common subsequences of equal length. The most common instance of the *LCS* problem usually requires to find only one in the maximum set.

Given the lengths of the input vectors as \\( m \\) and \\( n \\), solving the problem via a brute-force approach would take any subsequence of the first input vector (and there can be \\( 2^m \\)) and try to match it in all the subpositions of the second input vector in \\( O(n) \\). This gives us an exponential \\( O(n2^m) \\).

A better *dynamic programming* strategy tries to consider the prefixes of the \\( x,y \\) input vectors \\( c[i,j] \\). If there's a match at the \\( (i,j) \\) position the previous maximum match is increased, otherwise the best of the two prefixes is taken.

$$c[i,j] = \begin{cases}
c[i-1,j-1]  & \text{if x[i] = y[j]} \\[2ex]
max{c[i,j-1],c[i-1,j]} & \text{otherwise}
\end{cases}$$

The approach described is somewhat similar to the one used in the [edit distance]({% post_url 2015-08-21-edit-distance %}) problem. The code to find the lenght of the LCS follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

int lcs(const string& A, const string& B) {

  if (A.size() == 0 || B.size() == 0)
    return -1;

  // Base cases for first row and column are also included here
  vector<vector<int>> mat(A.size() + 1, vector<int>(B.size() + 1, 0));

  for (int j = 1; j < mat.size(); ++j) {
    for (int i = 1; i < mat[0].size(); ++i) {
      if (A[j - 1] == B[i - 1]) {
        mat[j][i] = mat[j - 1][i - 1] + 1;
      } else {
        mat[j][i] = max(mat[j - 1][i], mat[j][i - 1]);
      }
    }
  }

  return mat[A.size()][B.size()];
}

int main() {

  string A = "ACBAYAB";
  string B = "CBDYBAM";
  cout << "Longest common subsequence has length: " 
       << lcs(A, B) << endl; // 4

  return 0;
}
{% endhighlight %}

The code is \\( \Theta(nm) \\) and features space optimizations (only two rows are necessary at any time to compute the maximum length of the LCS). Tracking the LCS found can also be easily implemented.