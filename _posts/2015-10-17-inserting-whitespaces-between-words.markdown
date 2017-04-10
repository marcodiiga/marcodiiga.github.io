---
layout: post
title:  "Inserting whitespaces between words"
tags: algorithms dynamic-programming
---

> Given a whitespace-less string like
>
> "*thereverseisnottrue*"
>
> and a dictionary of words ordered by probability of appearance in a text (first means more likely to appear), insert the whitespaces
> in the string in the appropriate positions.

In probability [Zipf's law](http://www.britannica.com/topic/Zipfs-law) asserts that frequencies \\( f \\) of certain events are inversely
proportional to their rank \\( r \\). For example the most ranked English word *the* has a high frequency in a generic text. Suppose we were
given the following dictionary of words ordered by descending frequency

    the
    not
    is
    there
    reverse
    verse
    true

we can use a simplification of Zipf's law to estimate the cost of insertion of each word: \\( c(w) = \log(r+1) \\) i.e. the cost of a word \\( w \\)
is the logarithm of its rank + 1.

Now let's take into account the example string we were given

    thereverseisnottrue

with 0 characters we obviously have a cost of 0 since we haven't inserted anything yet. If we proceed adding a character each time and checking for a
correspondance in the dictionary we have that the first key found is at position 3

     thereverseisnottrue
    ^^^^
       | 'the' exists in the dictionary and has weight log(1+0)

The cost for the previous positions is `inf` since there is no match for `t` or `th`. By keeping a history equal to the longest string in the dictionary we
can recursively evaluate each previous position for a better match, i.e. for a solution which either

1. Covers all the characters encountered
2. Is more cost-effective than the previous one

e.g.

     thereverseisnottrue
       ^ ^
       | |*
       | | cost log(1+3)
       |
       | cost log(1+0)

when the position `*` has been reached we can now evaluate which one of the following splits is a better match

    ther + e   = inf + inf
    the + re   = log(1+0) + inf
    th + ere   = inf + inf
    t + here   = inf + inf
    '' + there = 0 + log(1+3)

therefore the best match is `there` with a cost of \\( 0 + \log(1+3) \\). This means that if we were to stop our search at this point, `there` would cost more than `the` but would provide a better match since it covers all the encountered characters. It is an *optimal solution for this subproblem* since all the other solutions are not viable.

When the input text is covered for the length of `thereverse` the process is restarted and this time `the` + `reverse` has a minimum cost rather than `there` + `verse`. Of course heuristics and more advanced text-recognition techniques would be used if this were a semantic-aware engine trying to reconstruct a text. In this simple instance it suffices to find the minimum cost to split the words.

Since this recursion exhibits both *optimal substructure* and *overlapping subproblems* during the dictionary checks, it is a good candidate to be solved with a dynamic programming approach

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
#include <sstream>
using namespace std;

// Words sorted according to their probability of appearing
vector<string> dictionary = {
  "the",
  "not",
  "is",
  "there",
  "reverse",
  "verse",
  "true"
};

string insertWhitespaces(const string& text) {

  // Preprocessing step on the dictionary: use a simplified Zipf's law to compute
  // the inverse of the probability of a word to occur in a text
  unordered_map<string, float> wordCosts;
  int maximumWordLength = -1;
  for (int i = 0; i < dictionary.size(); ++i) {
    wordCosts.emplace(dictionary[i], logf(static_cast<float>(i+1)));
    maximumWordLength = max(static_cast<int>(dictionary[i].size()), maximumWordLength);
  }  

  // Cost of all best matches at i-th character
  vector<float> cost(text.size() + 1, 0);
  vector<int> predecessorCharacters(text.size() + 1, -1);
  cost[0] = 0; // The cost for 0 characters is 0

  for (int i = 1; i <= text.size(); ++i) {
    // Explore all previous positions in the range maximumWordLength
    float minimumCost = numeric_limits<float>::max();
    int predecessorLength = -1;
    for (int j = 1; j <= min(i, maximumWordLength); ++j) {
      // Try to use j as a split-point and choose the most cost-effective split
      int startPos = i - j;
      auto it = wordCosts.find(text.substr(startPos, i - startPos));
      if (it != wordCosts.end() && minimumCost > cost[i - j] + it->second) {
        minimumCost = cost[i - j] + it->second;
        predecessorLength = i - startPos;
      }
    }
    cost[i] = minimumCost;
    predecessorCharacters[i] = predecessorLength;
  }

  // String reconstruction via backtracking
  int index = static_cast<int>(text.size());
  vector<string> resultReversed;
  while (predecessorCharacters[index] != -1) {
    resultReversed.push_back(text.substr(index -
                   predecessorCharacters[index], predecessorCharacters[index]));
    index -= predecessorCharacters[index];
  }
  stringstream result;
  for (int i = static_cast<int>(resultReversed.size()) - 1; i >= 0; --i) {
    result << resultReversed[i];
    if (i != 0)
      result << " ";
  }

  return result.str();
}

int main() {

  string text = "thereverseisnottrue";

  cout << insertWhitespaces(text); // the reverse is not true

  return 0;
}
{% endhighlight %}

The code runs in \\( O(N^2) \\) and requires \\( O(N) \\) space.
