---
layout: post
title:  "Solutions for a linear equation"
tags: algorithms dynamic-programming
---

> Suppose we have a linear equation of \\( n \\) variables of the form \\
> $$ 3i + 5j = 16 $$ \\
> How many solutions are there?

Let us reduce the problem to a simpler one: how many solutions are there to \\( 3i = 6 \\)?
We can think of this problem as recursively building on smaller known terms

$$ \begin{align}
   & 3i = 0 \rightarrow 1 \\
   & 3i = 1 \rightarrow 0 \\
   & 3i = 2 \rightarrow 0 \\
   & 3i = 3 \rightarrow 1 \\
   & 3i = 4 \rightarrow 0 \\
   & 3i = 5 \rightarrow 0 \\
   & 3i = 6 \rightarrow 1 \\
   \end{align}
$$

i.e. we have one way of getting 0 from \\( 3i \\) with \\( i=0 \\) and so on. This approach works similarly to a single-variable sieve: 
\\( f(xn) \\) i.e. the number of ways \\( xn \\) can be obtained builds on \\( f((x-1)n) \\): \\( f(xn) = f((x-1)n) \\).

When multiple variables are present, e.g. \\( 3i + 3j = 6 \\), we should first solve the problem for one variable as shown then proceed
to consider the second variable by building on the results of the former one

$$ \begin{align}
   & 3j = 0 \rightarrow 1 \\
   & 3j = 1 \rightarrow 0 \\
   & 3j = 2 \rightarrow 0 \\
   & 3j = 3 \rightarrow 1 + 1 = 2 \\
   & 3j = 4 \rightarrow 0 \\
   & 3j = 5 \rightarrow 0 \\
   & 3j = 6 \rightarrow 1 + 2 = 3 \\
   \end{align}
$$

so we have a total of 3 solutions for \\( (i,j) \\): \\( (0,2),(2,0),(1,1) \\). The same reasoning holds for the equation presented at
the beginning of this article.

The approach is well-suited for a dynamic programming implementation. Code follows

{% highlight c++ %}
#include <iostream>
#include <vector>
using namespace std;

int numberOfSolutions(const vector<int>& coeffs, const int kTerm) {
  
  // waysToObtain(x) is the number of ways to obtain x
  vector<int> waysToObtain(kTerm + 1, 0);

  waysToObtain[0] = 1; // Base case

  for (auto c : coeffs) { // For each variable coefficient
    for (int i = c; i <= kTerm; ++i) {
      waysToObtain[i] += waysToObtain[i - c];
    }
  }

  return waysToObtain[kTerm];
}

int main() {

  vector<int> coefficients = { 3, 5 };
  int knownTerm = 16;

  cout << numberOfSolutions(coefficients, knownTerm) << endl; // 1

  return 0;
}
{% endhighlight %}

Complexity is \\( O(NK) \\) where \\( N \\) is the number of coefficients and \\( K \\) the known term.