---
layout: post
title:  "Permutations and combinations"
tags: algorithms
---

A string of length \\( n \\) has \\( n! \\) permutations without repetitions. A
simple **incremental** algorithm to generate all permutations of a given string follows

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
as naive algorithms to generate combinations and permutations. It has to be noted that
other more efficient but more complex methods (e.g. [Heap](https://en.wikipedia.org/wiki/Heap%27s_algorithm))
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

Ranking and unranking permutations
==================================
A common problem when dealing with permutations is obtaining the **rank** of a permutation,
i.e. its index in the list of permutations sorted lexicographically.

A simple solution could be to just generate all permutations with the code seen some
paragraphs ago and check each time for a match with the given input permutation. Anyway
the complexity would remain exponential.

A smarter approach would be to [count the previous smaller permutations](http://www.geeksforgeeks.org/lexicographic-rank-of-a-string/)
that came before the input one.

The key to understanding both ranking and unranking for permutations lies in the
following lines so make sure to understand them.

As an example: suppose we were asked to rank the permutation \\( \{ 3,4,1,2 \} \\).
The first element is 3 and there are 2 elements in the set which are smaller than it: 1
and 2. This means there have to be other permutations of the form

$$ \text{1 x x x} \\
   \text{2 x x x} $$

that came **before** our permutation (since it starts with 3). How many permutations
are generated from the two lines above? Each one comprises \\( 3! \\) permutations
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

If we're numbering the permutations from 0, the permutation we were given has
rank **16**.

Thus the pseudocode for the ranking algorithm is

    rank = 0
    for each element i in the input permutation
       smallerElements = find the number of smaller elements from i onward

       rank += smallerElements * (number of elements from i+1 to the end)!


Unranking works similarly: we're given the input number 16 and we want to know
which permutation it corresponds to (in the lexicographic order list).

For this problem we're also given the input set (i.e. the elements to permute):
\\( \{ 1,2,3,4 \} \\). Notice that they can be unordered too.

[TODO]


The code to calculate ranking and unranking for permutations follows

{% highlight c++ %}
#include <algorithm>
#include <iostream>
#include <vector>
#include <iterator>
using namespace std;

int factorial(int i) { // Utility function
  if (i == 0)
    return 1;
  else
    return i*factorial(i - 1);
}

int getRankForPermutation(const vector<int>& v) {

  int rank = 0; // Start from 0

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
    rank += smallerChars * factorial(static_cast<int>(v.size()) - i - 1);
  }

  return rank;
}

vector<int> unrankPermutation(vector<int> vec, int rank) {

  sort(vec.begin(), vec.end()); // Sort the input set so that we have each index
                                // telling us how many elements are there less than it

  int remainingIndex = rank;
  vector<int> solution;
  while (vec.size() > 0) {

    int maximumFound = -1;
    int i = 0;
    for (; i < vec.size(); ++i) {
      // Calculate the previous permutations with this element as the leading element
      int previousPermutations = i * factorial(static_cast<int>(vec.size()) - 1);

      // The important check here is whether the previous permutations fit into the
      // rank we were given. Notice that the remaining index might even be zero. In that
      // case we just position the last elements of the permutation.
      if (previousPermutations > maximumFound && previousPermutations <= remainingIndex)
        maximumFound = previousPermutations;
      else if (previousPermutations > remainingIndex) { // We're not allowed to exceed the rank
        break;
      }
    }
    --i; // i was the solution either if we found a maximum or we terminated our variables

    solution.push_back(vec[i]);
    remainingIndex -= maximumFound;
    vec.erase(vec.begin() + i);
  }

  return solution;
}

int main(int argc, char * argv[]) {

  vector<int> vec = { 4, 3, 2, 1 };
  int rank;

  cout << "~-~-~-~ Indices are 0-based ~-~-~-~" << endl << endl;

  rank = getRankForPermutation(vec);
  cout << "Rank for permutation { ";
  copy(vec.begin(), vec.end(), ostream_iterator<int>(cout, " "));
  cout << "} is " << rank << endl << endl;


  vector<int> permutation = unrankPermutation(vec, rank);
  cout << "Rank " << rank << " permutation is { ";
  copy(permutation.begin(), permutation.end(), ostream_iterator<int>(cout, " "));
  cout << "}" << endl;

  return 0;
}
{% endhighlight %}

Complexity for the algorithms: \\( O(n^2) \\). Precomputing or using a table for the factorial
calculations would reduce the runtime. The code above could also work for duplicate
characters but it would need some adjustments, in particular:

1. The duplicate characters would have to be included as smaller elements

2. The total for an element would have to be divided by the number of duplicates
of that element. This is because duplicates will generate the same permutation
multiple times (in the same fashion as combinations without repetition need to get
  rid of the duplicates).
