---
layout: post
title:  "Permutations and combinations"
tags: algorithms
---

A string of length \\( n \\) has \\( n! \\) permutations without repetitions. A
simple algorithm to generate all permutations of a given string follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <iterator>
#include <algorithm>
using namespace std;

void permutations(vector<int>& v, size_t start, size_t end){
  if (start >= end) {
    copy(v.begin(), v.end(), ostream_iterator<int>(cout, " "));
    cout << endl;
    return;
  }
  for (size_t i = start; i < end; ++i) {
    swap(v[start], v[i]);
    permutations(v, start + 1, end);
    swap(v[start], v[i]);
  }
}

int main() {

  vector<int> vec = { 1, 2, 3 };
  permutations(vec, 0, vec.size());

  return 0;
}
{% endhighlight %}

A string also has

$$ \binom n k = \frac{n!}{k!(n-k)!} $$

combinations

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <iterator>
#include <algorithm>
using namespace std;

void combinations(vector<int>& v, vector<int>& temp, size_t start, size_t end, int k) {
  if (k == 0) {
    copy(temp.begin(), temp.end(), ostream_iterator<int>(cout, " "));
    cout << endl;
    return;
  }
  if (start >= end)
    return;
  temp.push_back(v[start]);
  combinations(v, temp, start + 1, end, k - 1);
  temp.pop_back();
  combinations(v, temp, start + 1, end, k);
}

void combinations(vector<int>& v, int k) {
  vector<int> temp;
  combinations(v, temp, 0, v.size(), k);
}

int main() {

  vector<int> vec = { 1, 2, 3 };
  combinations(vec, vec.size());

  return 0;
}
{% endhighlight %}

Since the code above can also be used to generate all possible subsets, it can be
used to generate k-permutations

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <iterator>
#include <algorithm>
using namespace std;

void permutations(vector<int>& v, size_t start, size_t end){
  if (start >= end) {
    copy(v.begin(), v.end(), ostream_iterator<int>(cout, " "));
    cout << endl;
    return;
  }
  for (size_t i = start; i < end; ++i) {
    swap(v[start], v[i]);
    permutations(v, start + 1, end);
    swap(v[start], v[i]);
  }
}

void combinations(vector<int>& v, vector<int>& temp, size_t start, size_t end, int k) {
  if (k == 0) {
    permutations(temp, 0, temp.size());
    return;
  }
  if (start >= end)
    return;
  temp.push_back(v[start]);
  combinations(v, temp, start + 1, end, k - 1);
  temp.pop_back();
  combinations(v, temp, start + 1, end, k);
}

void combinations(vector<int>& v, int k) {
  vector<int> temp;
  combinations(v, temp, 0, v.size(), k);
}

int main() {

  vector<int> vec = { 1, 2, 3 };
  combinations(vec, 2);

  return 0;
}
{% endhighlight %}

Both the permutations and combinations algorithms presented make use of the backtracking
paradigm and have complexity of \\( O(n!) \\) and \\( O(2^n) \\). They're intended
as naive algorithms to generate combinations and permutations. Other more efficient
but more complex methods (e.g. [Heap](https://en.wikipedia.org/wiki/Heap%27s_algorithm))
are available.


Lexicographically sorted permutations
=====================================

Generating [lexicographically sorted](http://mathworld.wolfram.com/LexicographicOrder.html)
permutations means generating permutations in increasing numerical order, e.g.

$$ 123, 132, 213, 231, 312, 321 $$

The algorithm is as follows (read the comments to understand the necessary steps)

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
using namespace std;

void sortedPermutations(vector<int>& v) {

  // Sort in increasing order the vector. This is also
  // the first permutation
  sort(v.begin(), v.end());

  auto printPermutation = [&]() {
    copy(v.begin(), v.end(), ostream_iterator<int>(cout, " "));
    cout << endl;
  };
  printPermutation();

  bool noMorePermutations = false;
  while (noMorePermutations == false) {

    // Grab the rightmost position less than its successor (e.g. 3 4)
    //                                                           ^
    int i = -1;
    for (i = static_cast<int>(v.size()) - 2; i >= 0; --i) {
      if (v[i] < v[i + 1])
        break;
    }

    if (i == -1) {
      // No more permutations available
      noMorePermutations = true;
      continue;
    }

    // Find the smallest successor of the found i at its right
    int min = -1;
    for (int j = i + 1; j < v.size(); ++j) {
      if (v[j] > v[i]) {
        if (min == -1)
          min = j;
        else if (v[j] < v[min])
          min = j;
      }
    }

    // Swap i and j
    swap(v[i], v[min]);

    // Sort the rest at the right of i in increasing order - (*)
    sort(v.begin() + i + 1, v.end());

    printPermutation();
  }
}

int main() {
  vector<int> vec = { 1, 2, 3, 4 };
  sortedPermutations(vec);
  return 0;
}
{% endhighlight %}

Regarding the *(\*)* asterisk-marked line, a possible optimization could be to
just reverse the array instead of sorting it since it will always be in decreasing
order. The complexity remains kind of daunting though: \\( O(n \cdot n!) \\).

The algorithm above also allows us to quickly find the next duplicate of a given
permutation (it is trivial to substitute the input permutation to the first result
  of the `std::sort` line).

Ranking
=======
A common problem when dealing with permutations is obtaining the **rank** of a permutation,
i.e. its index in the list of permutations sorted lexicographically.

A simple solution could be to just generate all permutations with the code seen some
paragraphs ago and check each time for a match with the given input permutation. Anyway
the complexity would remain exponential.

A smarter approach would be to [count the previous smaller permutations](http://www.geeksforgeeks.org/lexicographic-rank-of-a-string/)
that came before the input one.

As an example: suppose we were asked to rank the permutation \\( \{ 3,4,1,2 \} \\).
The first element is 3 and there are 2 elements which are smaller than it: 1
and 2. This means there have to be other permutations of the form

$$ \text{1 x x x} \\
   \text{2 x x x} $$

that came **before** our permutation (since it starts with 3). How many permutations
are included in the two lines above? Each one comprises \\( 3! \\) permutations
since there are three elements 'shuffling', thus they yield a grand-total of
\\( 2 \cdot 3! \\) permutations before ours.

Now let's keep 3 *fixed* and consider the element 4: there are three elements less
than it, but since we're not considering 3 anymore (it is fixed) only 2 remain. They
generate permutations that come before ours of the following form

$$ \text{3 1 x x} \\
   \text{3 2 x x} $$

and this time the number of total smaller permutations is \\( 2 \cdot 2! \\).
Let's repeat this process for the last two elements and we have a total of

$$ 2 \cdot 3! + 2 \cdot 2! + 0 + 0 = 16 $$

Since we're numbering the permutations from 1, the permutation we were given has
rank **17**. The code to calculate the rank of a permutation follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

unsigned int factorial(unsigned int n) {
  if (n == 0)
    return 1;
  return n * factorial(n - 1);
}

int getRankForPermutation(vector<int>& v) {

  int rank = 1; // Start from 1

  // Per each character in the input permutation
  for (int i = 0; i < v.size(); ++i) {

    // Find the number of smaller characters in the string from i onward
    int smallerChars = 0;
    int ch = v[i];
    for_each(v.begin() + i, v.end(), [&smallerChars, ch](const int& element) {
      if (element < ch)
        ++smallerChars;
    });

    // Calculate the previous smaller permutations by keeping i fixed
    rank += smallerChars * factorial(v.size() - i - 1);
  }

  return rank;
}

int main() {
  vector<int> vec = { 3, 4, 1, 2 };
  cout << "Rank is " << getRankForPermutation(vec); // 17
  return 0;
}
{% endhighlight %}

Complexity: \\( O(n^2) \\). Precomputing or using a table for the factorial
calculations would reduce the runtime. The code above could also work for duplicate
characters but

1. The duplicate characters would have to be included as smaller elements

2. The total for an element would have to be divided by the number of duplicates
of that element. This is because duplicates will generate the same permutation
multiple times (in the same fashion as combinations without repetition need to get
  rid of the duplicates).
