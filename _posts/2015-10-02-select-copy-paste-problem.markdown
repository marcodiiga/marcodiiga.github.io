---
layout: post
title:  "Select/Copy/Paste problem"
tags: algorithms dynamic-programming
---

> There are four actions available in a notepad edit box:
>  
> 1. Type the 'A' character
> 2. Select all the text in the edit box
> 3. Copy it into the clipboard buffer
> 4. Paste it
>
> Given a number of actions N return the maximum number of 'A' characters that can be printed.

This problem asks to find the maximum number of characters that can be typed in an edit box given a number of actions (or a number of key presses). Let \\( N \\) be 4 for instance. One could use all the 4 actions to press the 'A' key

    AAAA

or he could use one to type an 'A' character and the three left to select/copy/paste

    AA
     ^
     |
     typed through select/copy/paste

Since there are no other actions that can produce 'A' characters, the maximum number of characters can be obtained by typing all 'A's as in the first example: 4 'A's is the maximum that can be obtained with \\( N=4 \\).

Things begin to change when more actions are added as input. Until the number of actions hit \\( N=6 \\) typing characters manually outperforms (or performs equally as) the select/copy/paste operation. As soon as \\( N=7 \\) is hit, this is no longer true.

A good approach at solving this problem is through dynamic programming. Let \\( M(i) \\) be the maximum number of 'A's that can be obtained with \\( i \\) actions. It follows that

$$
M(i) = 
    \begin{cases}
                i & \mbox{if $i \le 6$} \\
                max\{M[i-1]+1, f(i)\} & \mbox{if $i > 6$} \\
     \end{cases} \\
\mbox{w/} \ f(i) = \max_{ 2 \le j \le i-2} \{ M[j](i - j) \} \\
$$

The code for this recursion follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int maximumNumberOfAPrinted(const int nOfKeys) {
  if (nOfKeys <= 6)
    return nOfKeys;

  vector<int> maximumWithKeys(nOfKeys + 1, 0);
  for (int i = 0; i <= 6; ++i) {
    maximumWithKeys[i] = i;
  }

  for (int i = 7; i <= nOfKeys; ++i) {
    int opt1 = maximumWithKeys[i - 1] + 1; // Just type another A
    int opt2 = 0;
    for (int j = 2; j <= i + 1 - 3; ++j) {
      opt2 = max(opt2, maximumWithKeys[j-1] * ((i + 1) - (j + 2) + 1));
    }
    maximumWithKeys[i] = max(opt1, opt2);
  }

  return maximumWithKeys[nOfKeys];
}


int main() {

  const int nOfKeys = 11;
  cout << "Maximum of " << maximumNumberOfAPrinted(nOfKeys) << endl; // 27

  return 0;
}
{% endhighlight %}

The code runs in \\( O(N^2) \\).