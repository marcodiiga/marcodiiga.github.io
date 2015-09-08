---
layout: post
title:  "Minimum number of jumps"
tags: algorithms dynamic-programming
---

A classical question is the *minimum number of jumps* problem. Instances of this problem are usually of the following form

> Find the minimum number of jumps needed to get from a starting point X to an end point Y given a number of possible steps per each subposition

Easier instances have a fixed distance per each intermediate step. Harder instances use an array of distances available to move forward at each subposition.
E.g. given the following vector

$$ v = \{2,1,1,4\} $$

and let position (0-based) \\( X=0 \\) be the start position and \\( Y=3 \\) be the end position, find the minimum number of jumps to go from \\( X \\) to \\( Y \\).

There are two possible paths:

$$ 
   p_1 = \{ 2 \to 1 \to 1 \to 4 \} \\
   p_2 = \{ 2 \to 1 \to 4 \}
$$

Since \\( p_2 \\) only uses 2 jumps, it is the path using the minimum number of jumps we were searching for.

A recursive approach covering the entire search space (i.e. starting at \\( X \\) and recursively proceeding from every reachable new index) would be \\( O(n!) \\).

However a dynamic programming approach is available since the problem exhibits overlapping subproblems and optimal substructure: it is possible to determine an optimal number of jumps per each subset of the input indices.

{% highlight c++ %}
#include <iostream>
#include <algorithm>
#include <vector>
#include <limits>
using namespace std;

int minJumps(const vector<int>& vec) {
  vector<int> jumpsFrom0(vec.size(), 0);
  
  jumpsFrom0[0] = 0; // Base case

  for (int i = 1; i < vec.size(); ++i) {
    jumpsFrom0[i] = numeric_limits<int>::max(); // Unreachable
    for (int j = 0; j < i; ++j) {
      if (jumpsFrom0[j] != -1 && // Can be reached
          j + vec[j] >= i) {     // Can reach/surpass i
        jumpsFrom0[i] = min(jumpsFrom0[i], jumpsFrom0[j] + 1);
      }
    }
  }

  return jumpsFrom0.back();
}

int main() {

  vector<int> vec = { 2, 1, 1, 4 };
  cout << minJumps(vec); // 2

  return 0;
}
{% endhighlight %}

The dynamic programming solution is \\( O(N^2) \\). One should always check the previous indices, i.e. in the array

$$ v = \{ 1, 3, 2, 10, 9, 2, 4, 1, 6 \}$$

at position \\( i = 4\\), the subindex \\( j = 1 \\) could reach \\( i \\) in less jumps than any other index.