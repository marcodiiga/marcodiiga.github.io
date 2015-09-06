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
    for (int j = i + 1; j < static_cast<int>(v.size()); ++j) {
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
\\( \{ 2,1,4,3 \} \\). Notice that they can be unordered.

We start by sorting the input set, this allows us to use each index to automatically
know how many smaller elements are there for a given element: \\( \{ 1,2,3,4 \} \\).
Then per each element we calculate the number of permutations less than the one
with that element as leading one, e.g. for the first element 1 we calculate
\\( 0 \cdot 3! = 0 \\) permutations before it (1 is the smallest element). 0 permutations
surely fits into the asked 16 rank thus we store it as a potential candidate. We then
try to have 2 as leading element and calculate that there are \\( 1 \cdot 3! = 6 \\)
permutations, this fits even better in the 16 rank and thus we store it as new candidate.
We then try 3 as discover that it has 12 permutations less than the first permutation
with it as leading element - even better. 4 doesn't work (it yields 18 and that's too
much to fit into 16) and thus we stop searching for a first element.

The first element is therefore 3 and since there are 12 permutations starting with
element less than 3, we still have \\( 16-12 \\) permutations to cover. The process
then starts again to find the next digit (3 is eliminated by the set of potential
  candidates).

    sort input set

    while(there are elements in the input set) {
      maximumFound = -1;
      for each element i in the input set
         smallerElements = find the number of smaller permutations before the one
                           starting with i
         maximumFound = max(maximumFound, smallerElements) - make sure this fits into
                        the given rank

      solution += add i corresponding to the maximumFound
      delete i from the input set
    }

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
                                // telling us how many elements are there less
                                // than it

  int remainingIndex = rank;
  vector<int> solution;
  while (vec.size() > 0) {

    int maximumFound = -1;
    int i = 0;
    for (; i < vec.size(); ++i) {
      // Calculate the previous permutations with this element as the leading element
      int previousPermutations = i * factorial(static_cast<int>(vec.size()) - 1);

      // The important check here is whether the previous permutations fit into the
      // rank we were given. Notice that the remaining index might even be zero.
      // In that case we just position the last elements of the permutation.
      if (previousPermutations > maximumFound &&
          previousPermutations <= remainingIndex)
        maximumFound = previousPermutations;
      else if (previousPermutations > remainingIndex) { // We're not allowed to
                                                        // exceed the rank
        break;
      }
    }
    // i was the solution either if we found a maximum or we terminated our variables
    --i;

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

Precomputing or using a table for the factorial calculations would reduce the runtime.
The code above could also work for duplicate characters but it would need some adjustments, in particular:

1. The duplicate characters would have to be included as smaller elements

2. The total for an element would have to be divided by the number of duplicates
of that element. This is because duplicates will generate the same permutation
multiple times (in the same fashion as combinations without repetition need to get
  rid of the duplicates).

Lexicographically sorted combinations
=====================================
Generating lexicographically sorted combinations proves to be easier than with
permutations since for \\( n = 4 \\) and \\( k = 2 \\) we have

$$ 1 2 \\
1 3 \\
1 4 \\
2 3 \\
2 4 \\
3 4 $$

and we can think of a recursive algorithm

    generate_combination(set, k, start)
      for each element i from start to the end of the set
        store_element i
        generate_combination(set, k-1, start+1)
        drop_element i

Code follows

{% highlight c++ %}
#include <algorithm>
#include <iostream>
#include <vector>
#include <iterator>
using namespace std;

void combinationsLexicographically(vector<int>& v, int index, vector<int>& c, int k) {
  if (k == 0) {
    copy(c.begin(), c.end(), ostream_iterator<int>(cout, " "));
    cout << endl;
    return;
  }

  for (int i = index; i < static_cast<int>(v.size()); ++i) {
    c.push_back(v[i]);
    combinationsLexicographically(v, i + 1, c, k - 1);
    c.pop_back();
  }

}

void combinationsLexicographically(vector<int>& v, int k) {
  sort(v.begin(), v.end());

  vector<int> combination;
  for (int i = 0; i < static_cast<int>(v.size()); ++i) {
    combination.push_back(v[i]);
    combinationsLexicographically(v, i + 1, combination, k - 1);
    combination.pop_back();
  }
}

int main(int argc, char * argv[]) {

  vector<int> v = { 1, 2, 3, 4 };
  int k = 2;

  combinationsLexicographically(v, k);

  return 0;
}
{% endhighlight %}

Ranking and unranking combinations
==================================

While generating lexicographically sorted combination was easier than its permutation
counterpart, ranking and unranking combinations proves to be considerably more challenging
since the reasoning we did can no longer be applied.

Before diving into the ranking/unranking algorithms two reminders are needed: the first
one being the [multiplicative formula](https://en.wikipedia.org/wiki/Binomial_coefficient#Multiplicative_formula)
for the binomial coefficient calculation. The formula assures that

$$ \binom nk = \prod_{i=1}^k \frac{n+1-i}{i} $$

or, put another way,

$$ \binom {4}{2} = \frac{4 +1-1}{1} \cdot \frac{4 +1-2}{2} = 6$$

In code it becomes

{% highlight c++ %}
int binomial(int n, int k) {
  if (n < 0 || k < 0 || k > n) // Exceptional/nonsense cases handling
    return 0;
  int b = 1;
  for (int i = 0; i < k; ++i) {
    b = b * (n - i) / (i + 1); // Simplified multiplicative formula
  }
  return b;
}
{% endhighlight %}

A second reminder is needed: the [recursive formula](https://en.wikipedia.org/wiki/Binomial_coefficient#Recursive_formula)

$$ \binom nk = \binom{n-1}{k-1} + \binom{n-1}k $$

In our previous example with \\( n = 4 \\) and \\( k = 2 \\) we had

$$ 1 2 \\
1 3 \\
1 4 \\
2 3 \\
2 4 \\
3 4 $$

The first term of the recursive formula corresponds to the first interval of combinations
starting with 1 and featuring 2,3 and 4 *shuffling*. The rest corresponds to three elements
(every other element except 1 which has been already done) *shuffling* with \\( k=2 \\).
The process restarts recursively. Another consideration to keep in mind is that
the first lexicographic element is always the first \\( k \\) elements of the set.
At the end of an interval the first combination is exactly the previous one with every
element increased by one: \\( {1,2} \dashrightarrow {2,3} \\)

Understanding this reasoning is the key to understanding how ranking works for
combinations (and also how *k-subsets* are generated). The pseudocode is

    rankThisCombination(combination, n)
      infer k from the combination's size

      handle special k cases (e.g. n == k)

      if k == 1 just return combination[0]

      for every element in combination
        --element; // Back out one interval
      // this is only needed for the final adjustments

      if we reached the first interval (i.e. no other intervals before this one)
        return rankThisCombination(combination, n-1) // Recurse without the interval

      return binomial(n-1,k-1) /* Skip one interval and continue */
             + rankThisCombination(combination, n-1)


Unraking works similarly although the code is a bit more involved:

      unrankGetCombination(n, k, rank, start = 1)

        comb = generate first combination from start with k elements

        s = find the biggest smaller interval where our rank fits

        if comb has more than one element
          // recurse without the first element
          comb = unrankGetCombination(n - 1, k - 1, rank - s, start = comb[1])

        // If we finished our recursions or comb has one element, either case
        // just return it. This assumes a [1;n] input array
        return comb

The previous considerations assume a \\( [1;n] \\) interval input set although the
same considerations can be applied to any input set.

The complete code for ranking and unranking combinations is the following

{% highlight c++ %}
#include <algorithm>
#include <iostream>
#include <vector>
#include <iterator>
using namespace std;

int binomial(int n, int k) {
  if (n < 0 || k < 0 || k > n)
    return 0;
  int b = 1;
  for (int i = 0; i < k; ++i) {
    b = b * (n - i) / (i + 1); // Multiplicative formula
  }
  return b;
}

int getRankForCombination(int n, vector<int> comb) {

  int k = static_cast<int>(comb.size());
  if (k == 0 || k == n)
    return 0;

  for (auto& c : comb)
    --c;

  int j = comb[0];
  if (k == 1) // This assumes an input set from [1;n]
    return j;

  if (j == 0) {
    vector<int> sp(comb.size() - 1);
    copy(comb.begin() + 1, comb.end(), sp.begin());
    return getRankForCombination(n - 1, sp);
  }

  return binomial(n - 1, k - 1) + getRankForCombination(n - 1, comb);
}

vector<int> unrankGetCombination(int n, int k, int rank, int start = 1) {

  vector<int> comb(k);
  for (int i = 0; i < k; ++i){
    comb[i] = i + start;
  }

  int s = 0;
  int currentN = n - 1;
  int newS = binomial(currentN, k - 1);
  while (newS <= rank) {
    s = newS;
    for (auto& c : comb)
      ++c;
    newS += binomial(--currentN, k - 1);
  }

  if (comb.size() > 1) {
    vector<int> rest = unrankGetCombination(n - 1, k - 1, rank - s, comb[1]);
    for (int i = 0; i < rest.size(); ++i) {
      comb[i + 1] = rest[i];
    }
  }

  return comb;
}

int main(int argc, char * argv[]) {

  const int N = 4; // {1,2,3,4}
  const int K = 2;
  vector<int> combination = { 3, 4 };
  int rank;

  cout << "~-~-~-~ Indices are 0-based ~-~-~-~" << endl << endl;

  rank = getRankForCombination(N, combination /*K is deduced from the combination*/);
  cout << "Rank for combination { ";
  copy(combination.begin(), combination.end(), ostream_iterator<int>(cout, " "));
  cout << "} is " << rank << endl << endl; // Rank for combination { 3 4 } is 5

  combination = unrankGetCombination(N, K, rank);
  cout << "Rank " << rank << " combination is { ";
  copy(combination.begin(), combination.end(), ostream_iterator<int>(cout, " "));
  cout << "}" << endl; // Rank 5 combination is { 3 4 }

  return 0;
}
{% endhighlight %}
