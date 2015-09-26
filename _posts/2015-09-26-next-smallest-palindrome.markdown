---
layout: post
title:  "Next smallest palindrome"
tags: algorithms
---

> Find the next smallest palindrome greater than an input number N

This problem doesn't exhibit substantial mathematical difficulties, however requires careful consideration for corner cases that might arise. Let's consider a simple case where \\( N = 7215 \\). The next smallest palindrome greater than \\( N \\) is \\( 7227 \\) since mirroring the left part to the right is sufficient to form a palindrome number greater than the input one.

The corner cases arise when the number formed by mirroring the left part to the right isn't greater than the input one, e.g. \\( N = 7235 \\). In this case the first digit of the left part (either the central element or not) must be incremented before the mirroring. This incidentally brings the fact that the number itself might increase in length of digits (i.e. carries might be propagated). The input number can also be a palindrome *already*: \\( N=12321 \\). Mirroring the left part on the right would not yield a greater palindrome either and increasing the left part once again proves to be necessary.

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <iterator>
using namespace std;

// ~ Utility operator overloads to handle vector of digits ~ //

bool operator==(const vector<int>& a, const vector<int>& b) {
  if (a.size() != b.size())
    return false;
  for (int i = 0; i < a.size(); ++i) {
    if (a[i] != b[i])
      return false;
  }
  return true;
}

bool operator>(const vector<int>& a, const vector<int>& b) {
  if (a.size() > b.size())
    return true;
  else if (a.size() < b.size())
    return false;
  int an = 0, bn = 0;
  for (int i = 0; i < a.size(); ++i) {
    an = (an * 10) + a[i];
    bn = (bn * 10) + b[i];
  }
  return an > bn;
}

// Increments the vector by 1 from pos and deals with carries
void plusOne(vector<int>& v, int pos) {
  bool carry = false;
  for (int i = static_cast<int>(v.size() - 1 - pos); 
                                        i >= 0; --i) {
    ++v[i];
    if (v[i] == 10) {
      v[i] = 0;
      carry = true;
    } else {
      carry = false;
      break;
    }
  }
  if (carry == true) {
    v.insert(v.begin(), 1);
  }
}

void mirrorLeftHalfToRight(vector<int>& v) {
  for (int i = 0; i < v.size() / 2; ++i) {
    v[v.size() - 1 - i] = v[i];
  }
}

vector<int> getNextPalindrome(vector<int>& num) {
  vector<int> temp = num;

  // First easy case: check if copying the left part to the right yields a palindrome
  mirrorLeftHalfToRight(temp);

  if (temp > num) // Found a palindrome greater than the original one
    return temp;

  // Harder case, cannot be formed by simply "mirroring" the right part

  // Increment the first of the left part (either the center or part of the left part)
  temp = num;
  plusOne(temp, static_cast<int>(temp.size() / 2));

  // Now re-mirror the right part
  mirrorLeftHalfToRight(temp);

  return temp; // This surely worked
}


int main() {
  auto printResult = [](vector<int>& n, vector<int>& r) {
    cout << "Next palindrome for ";
    copy(n.begin(), n.end(), ostream_iterator<int>(cout, " "));
    cout << "is ";
    copy(r.begin(), r.end(), ostream_iterator<int>(cout, " "));
    cout << endl;
  };

  vector<int> num = { 1,9,1 };
  auto res = getNextPalindrome(num);
  printResult(num, res); // 2 0 2
  
  num = { 9,9,9 };
  res = getNextPalindrome(num);
  printResult(num, res); // 1 0 0 1

  num = { 1,2,3,4,6 };
  res = getNextPalindrome(num);
  printResult(num, res); // 1 2 4 2 1

  num = { 1 };
  res = getNextPalindrome(num);
  printResult(num, res); // 2

  return 0;
}
{% endhighlight %}