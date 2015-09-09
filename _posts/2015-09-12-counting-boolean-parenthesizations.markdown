---
layout: post
title:  "Counting boolean parenthesizations"
tags: algorithms dynamic-programming
---

The *parenthesization* or *counting boolean parenthesization* problem is somewhat similar to [optimal binary search tree]({% post_url 2015-07-26-optimal-binary-search-tree %}) finding. Given a boolean expression like

$$ true \lor true \land false \oplus true $$

the task is to determine the number of possible parenthesizations which render the expression \\( true \\). In the example above we have four solutions

$$ 
  true \lor ((true \land false) \oplus true) \\
  true \lor (true \land (false \oplus true)) \\
  (true \lor true) \land (false \oplus true) \\
  ((true \lor true) \land false) \oplus true \\
$$

and only one way to render it \\( false \\)

$$ (true \lor (true \land false)) \oplus true $$

Let \\( T(i,j) \\) represent the number of ways to parenthesize the operands in the range \\( [i;j] \\) such that the subexpression evaluates to \\( true \\) and let \\( F(i,j) \\) be the number of ways it evaluates to \\( false \\). It follows that

$$  
  T(i,j) = \begin{cases}
           \mbox{if operator is $\land$} \\[1pt]
           T(i,j) = T(i,k) \cdot T(k + 1,j) \\[2ex]

           \mbox{if operator is $\lor$} \\[1pt]
           T(i,j) = (T(i,k) + F(i,k)) \cdot (T(k + 1,j) + F(k + 1,j)) - (F(i,k) \cdot F(k + 1,j)) \\[2ex]

           \mbox{if operator is $\oplus $} \\[1pt]
           T(i,j) = (T(i,k) \cdot F(k + 1,j)) + (F(i,k) \cdot T(k + 1,j)) \\[2ex]
           \end{cases} \\
$$


$$
  F(i,j) = \begin{cases}
           \mbox{if operator is $\land$} \\[1pt]
           F(i,j) = (T(i,k) + F(i,k)) \cdot (T(k + 1, j) + F(k + 1, j)) - (T(i,k) \cdot T(k + 1, j)) \\[2ex]

           \mbox{if operator is $\lor$} \\[1pt]
           F(i,j) = F(i,k) \cdot F(k + 1, j) \\[2ex]

           \mbox{if operator is $\oplus $} \\[1pt]
           F(i,j) = (T(i,k) \cdot T(k + 1, j)) + (F(i,k) \cdot F(k + 1, j)) \\[2ex]
           \end{cases}
$$

The problem can be solved recursively or with a dynamic programming solution in \\( O(N^3) \\) as follows

{% highlight c++ %}
#include<iostream>
#include<vector>
using namespace std;

enum BOOLEAN_OPERATOR { AND, OR, XOR };

int booleanTrueParenthesization(const vector<bool>& operands, 
                                const vector<BOOLEAN_OPERATOR>& operators) {

  vector<vector<int>> T(operands.size(), vector<int>(operands.size(), 0));
  vector<vector<int>> F = T;

  // Initialize base cases (i.e. single-operand subexpressions)
  for (int i = 0; i < operands.size(); ++i) {
    if (operands[i] == true)
      ++T[i][i];
    else
      ++F[i][i];
  }

  for (int L = 2; L <= operands.size(); ++L) {
    for (int i = 0; i <= operands.size() - L; ++i) {
      int j = i + L - 1;
      for (int k = i; k < j; ++k) { // Break points for subexpressions
        switch (operators[k]) {
          case AND: {
            T[i][j] += T[i][k] * T[k+1][j];
            F[i][j] += (T[i][k] + F[i][k]) * (T[k + 1][j] + F[k + 1][j]) 
                       - (T[i][k] * T[k + 1][j]);
          } break;
          case OR: {            
            T[i][j] += (T[i][k] + F[i][k]) * (T[k + 1][j] + F[k + 1][j]) 
                       - (F[i][k] * F[k + 1][j]);
            F[i][j] += F[i][k] * F[k + 1][j];
          } break;
          case XOR: {
            T[i][j] += (T[i][k] * F[k + 1][j]) + (F[i][k] * T[k + 1][j]);
            F[i][j] += (T[i][k] * T[k + 1][j]) + (F[i][k] * F[k + 1][j]);
          } break;
        }
      }
    }
  }
  return T[0][operands.size() - 1];
}

int main() {  
  vector<bool> operands = { true, true, false, true };
  vector<BOOLEAN_OPERATOR> operators = { OR, AND, XOR };
  
  cout << booleanTrueParenthesization(operands, operators); // 4

  return 0;
}
{% endhighlight %}
