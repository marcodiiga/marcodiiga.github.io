---
layout: post
title:  "Linear recurrence relations"
tags: algorithms
---

A linear recurrence relation is a function or sequence of terms where each term is a [linear combination](https://en.wikipedia.org/wiki/Linear_combination) of previous terms. As such, only multiplications by constants are allowed per each term before adding them up: the following is not a linear recurrence

$$ f(i) = f(i-1) \cdot f(i-2) $$

while the following Fibonacci relation is

$$ f(i) = f(i-1) + f(i-2) $$

When prompted with the following task

> Given f, a function defined as a linear recurrence relation, compute f(N) where N might get very large

the right way to approach the problem isn't by brute-forcing the solution (e.g. by sieving as in [primes generation]({% post_url 2015-09-10-primes-generation %})) but rather to find a **transformation matrix** for the recurrence and repeatedly multiplying it by itself in order to find the result (i.e. applying it multiple times to an input vector).

Determining K - the number of terms on which f(i) depends
======


The first step is determining \\( K \\) which is the minimum integer such that \\( f(i) \\) doesn't depend on \\( f(i-M) \\) for all \\( M > K \\), e.g.

$$ f(i) = f(i-1)+f(i-2) \qquad \mbox{ $K = 2$ }$$

but in this sequence with missing terms

$$ f(i) = 2f(i-2) + f(i-4) \qquad \mbox{ $K=4$ } $$

since there are terms multiplied with 0-coefficients.

Determining the initial input vector F1
======

For Fibonacci sequence (\\( K=2 \\)) the initial values are

$$ f(1) = 1 \\ f(2) = 1 $$

More accurately the input vector \\( F_i \\) is a \\( K \times 1 \\) matrix whose first row is \\( f(i) \\), second row is \\( f(i+1) \\) and so on

$$ F_1 =  \begin{bmatrix} f(1) \\ f(2) \\ \vdots \\ f(K) \end{bmatrix} $$

Determining T - the transformation matrix
======

This is the most important step in the process: construct a \\( K \times K \\) matrix \\( T \\) called **transformation matrix** such that

$$ TF_i = F_{i+1} $$

If \\( f(i) = \sum_{j=1}^K c_j f(i-j) \\), the transformation matrix has the following form

$$ T =  \begin{bmatrix}
        0 & 1 & 0 & 0 & \cdots & 0 \\
        0 & 0 & 1 & 0 & \cdots & 0 \\
        0 & 0 & 0 & 1 & \cdots & 0 \\
        \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\
        c_K & c_{K-1} & c_{K-2} & c_{K-3} & \cdots & c_{1}
        \end{bmatrix} $$

where last row is a vector \\( \begin{bmatrix} c_K & c_{K-1} & c_{K-2} & c_{K-3} & \cdots & & c_{1} \end{bmatrix} \\).

In the Fibonacci sequence

$$ T = \begin{bmatrix} 0 & 1 \\ 1 & 1 \end{bmatrix} $$

Determining \\( F_N \\)
======

The last step is to repeatedly apply the transformation matrix \\( T \\) to \\( F_1 \\) to get \\( F_N \\)

$$ F_N = T^{N-1} F_1 $$

which in the Fibonacci case becomes

$$ \begin{bmatrix} 0 & 1 \\ 1 & 1 \end{bmatrix}^{N-1} \begin{bmatrix} 1 & 1 \end{bmatrix} $$

To efficiently compute \\( T^{N-1} \\) the most popular method is to use [exponentiation by squaring]({% post_url 2015-07-22-integer-arithmetic-using-bit-operations %}) that does it in \\( O(\log{N}) \\) time. A brief reminder follows

$$
x^y=
    \begin{cases}
                x \, ( x^{2})^{\frac{y - 1}{2}}, & \mbox{if } y \mbox{ is odd} \\
                (x^{2})^{\frac{y}{2}} , & \mbox{if } y \mbox{ is even}.
     \end{cases}
$$

Multiplying two matrices together takes \\( O(K^3) \\) time with the standard method so overall is \\( O(K^3 \log{N}) \\).

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

Handling variants in the recurrence relation
======
Some variants in the recurrence relation might occur, the most common ones being

* Constants in the relation

If the function is of the form \\( f(i) = \sum_{j=1}^K c_j f(i-j) +d \\) in this variant the starting vector \\( F_1 \\) is enhanced to remember the value of \\( d \\) and ends up being of size \\( (K+1) \times 1 \\)

$$ F_i = \begin{bmatrix} f(i) \\ f(i+1) \\ \vdots \\ f(i+K-1) \\ d \end{bmatrix} $$

the recurrence relation becomes

$$ \begin{bmatrix} T \end{bmatrix} \begin{bmatrix} f(i) \\ f(i+1) \\ \vdots \\ f(i+K-1) \\ d \end{bmatrix} = \begin{bmatrix} f(i+1) \\ f(i+2) \\ \vdots \\ f(i+K) \\ d \end{bmatrix}$$

and the matrix is therefore

$$ T =  \begin{bmatrix}
        0 & 1 & 0 & 0 & \cdots & 0 & 0 \\
        0 & 0 & 1 & 0 & \cdots & 0 & 0 \\
        0 & 0 & 0 & 1 & \cdots & 0 & 0 \\
        \vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\
        c_K & c_{K-1} & c_{K-2} & c_{K-3} & \cdots & c_{1} & 1 \\
        0 & 0 & 0 & 0 & cdots & 0 & 1
        \end{bmatrix} $$

E.g. with \\( f(i) = 2f(i-1) + 3f(i-2) + 5 \\) the matrix is

$$ \begin{bmatrix}0 & 1 & 0 \\ 3 & 2 & 1 \\ 0 & 0 & 1 \end{bmatrix}
\begin{bmatrix} f(i) & f(i+1) & 5 \end{bmatrix} = 
\begin{bmatrix} f(i+1) & f(i+2) & 5 \end{bmatrix}$$

* Odd/even conditional function

If the function behaves differently according to the parity of the argument, e.g. if \\( i \\) is even then \\( f(i) = f(i-1) / 2 \\) otherwise \\( f(i) = 3f(i-1) + 1 \\), it is possible to split the equation into two parts and construct \\( T_{even} \\) and \\( T_{odd} \\)

$$ \begin{cases} T_{even} F_i = F_{i+1} && \mbox{$i$ is even} \\
T_{odd} F_i = F_{i+1} && \mbox{$i$ is odd} \end{cases} $$

And \\( T = T_{even} \cdot T_{odd} \\). Therefore we can compute \\( F_N \\) with a single formula

$$ \begin{cases} F_N = T^{N/2} F_1 && \mbox{if $N$ is odd} \\
F_N = T_{odd} T^{(N-1)/2} F_1 && \mbox{otherwise} \end{cases}$$

Note that \\( T_{even} \\) and \\( T_{odd} \\) must be of the same size so in the example provided \\( f(i) = f(i-1) / 2 \\) must be converted into \\( f(i) = \frac{1}{2}f(i-1) + 0 \\) to look like the odd part. The standard method can then be applied.

References
======
This article is mostly based on the excellent post provided by Ashar Fuadi in his [solving linear recurrence for programming contests](http://fusharblog.com/solving-linear-recurrence-for-programming-contest/).