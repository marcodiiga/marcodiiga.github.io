---
layout: post
title:  "Linear recurrence relations"
tags: algorithms
---

A linear recurrence relation is a function or sequence of terms where each term is a [linear combination](https://en.wikipedia.org/wiki/Linear_combination) of previous terms. As such, only multiplications by constants are allowed per each term before adding them up.

When prompted with the following task

> Given f, a function defined as a linear recurrence relation, compute f(N) where N might get very large

the right way to approach the problem isn't by brute-forcing the solution (e.g. by sieving as in [primes generation]({% post_url 2015-09-10-primes-generation %})) but rather to find a **transformation matrix** for the recurrence and repeatedly multiplying it by itself in order to find the result (i.e. applying it multiple times to an input vector).

Constructing a  **transformation matrix** such that

$$ TF_i = F_{i+1} $$

is the fundamental step. In the Fibonacci sequence

$$ T = \begin{bmatrix} 0 & 1 \\ 1 & 1 \end{bmatrix} $$

To efficiently compute \\( T^{N-1} \\) the most popular method is to use [exponentiation by squaring]({% post_url 2015-07-22-integer-arithmetic-using-bit-operations %}) that does it in \\( O(\log{N}) \\) time. A brief reminder follows

$$
x^y=
    \begin{cases}
                x \, ( x^{2})^{\frac{y - 1}{2}}, & \mbox{if } y \mbox{ is odd} \\
                (x^{2})^{\frac{y}{2}} , & \mbox{if } y \mbox{ is even}.
     \end{cases}
$$

Multiplying two matrices together takes \\( O(K^3) \\) time (with \\( K \\) being any of the transformation matrix dimensions) with the standard method so overall is \\( O(K^3 \log{N}) \\).

Putting everything together
======

The following code calculates the \\( N^{th} \\) Fibonacci number in \\( O(K^3 \log{N}) \\)

{% highlight c++ %}
#include <iostream>
#include <vector>
using namespace std;

typedef vector<vector<int>> matrix;

// Multiplication for square matrices
matrix mul(matrix A, matrix B) {
  // Assuming matrices are non-zero, squared and of the
  // right dimensions
  matrix result(A.size(), vector<int>(A[0].size()));
  for (int i = 0; i < A.size(); ++i) {
    for (int j = 0; j < A[0].size(); ++j) {
      int temp = 0;
      for (int k = 0; k < A[0].size(); ++k) {
        temp += A[i][k] * B[k][j];
      }
      result[i][j] = temp;
    }
  }
  return result;
}

matrix pow(matrix T, int N) {
  if (N == 1)
    return T;
  if (N % 2 != 0)
    return mul(T, pow(T, N - 1));
  matrix result = pow(T, N / 2);
  return mul(result, result);
}

// Use linear recurrence relation to calculate the Nth Fibonacci number
int calculateNthFib(const int N) {
  if (N == 1)
    return 1; // Handle base case
  matrix T = {
    { 0, 1 },
    { 1, 1 }
  };
  vector<int> F1 = { 1, 1 };
  // Calculate T^N-1 via exponentiation by squaring
  matrix TN = pow(T, N-1);
  // Get Fn's first row
  int Fn = 0;
  for (int i = 0; i < TN.size(); ++i) {
    Fn += F1[i] * TN[i][0];
  }
  return Fn;
}

int main() {

  // Starts with 1th == 1
  cout << calculateNthFib(21); // 10946 = 2 x 13 x 421 (prime factor)
  
  return 0;
}
{% endhighlight %}

References
======
* [Solving linear recurrence for programming contests](http://fusharblog.com/solving-linear-recurrence-for-programming-contest/).