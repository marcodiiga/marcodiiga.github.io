---
layout: post
title:  "Length of longest valid substring"
tags: algorithms
---

> Given an input string consisting of opening and closing parenthesis find the length of the longest valid contiguous subexpression

An example is worth a thousand words:

    input:   "()"
    output:  2

    input:   ")())())"
    output:  2

    input:   ")(()())("
    output:  6

This problem exhibit similarities with the [the maximum subarray problem]({% post_url 2015-09-16-maximum-and-maximum-zero-sum-subarray-problems %}) although it **does** require a history to check for the continuation of the current valid subexpression. A stack is the natural data structure to be used to track the previous expression openings. The breakpoint of a contiguous subexpression is a closing parenthesis `)` with no pending `(` on the stack.

The algorithm is quite simple and runs in \\( O(N) \\)

{% highlight c++ %}
#include <iostream>
#include <algorithm>
#include <stack>
using namespace std;

int longestValidParenthesisExpression(const string& input) {
  stack<char> st;
  int currentMax = 0;
  int globalMax = 0;
  for (int i = 0; i < input.size(); ++i) {
    if (input[i] == '(')
      st.push(input[i]);
    if (input[i] == ')') {
      // Either pop out a valid sequence or reset the current max
      if (st.size() > 0 && st.top() == '(') {
        st.pop();
        currentMax += 2;
        globalMax = max(globalMax, currentMax);
      } else {
        currentMax = 0; // Invalid, reset the sequence
      }
    }
  }
  return globalMax;
}

int main() {

  string input = ")(()())(";
  cout << longestValidParenthesisExpression(input); // 6

  return 0;
}
{% endhighlight %}
