---
layout: post
title:  "Maximize gain by choosing numbers from array ends"
tags: algorithms dynamic-programming
---

> Find the maximum gain in a two-players game where an array is given and each player in turn is allowed to pick one element from either end of the array. The goal of each player is to score more than his opponent. Does the player who plays first always win?

Let us take an example array

    { 1, 3, 100, 2 }

A greedy approach would not work since picking up 2 would leave the 100 element at one of the ends to be picked up by our opponent. A better strategy is to think in terms of subproblems. Suppose we have a 1-element array (e.g. we reached the last step of the game)

    { 15 }

two things can happen

1. It is our turn: we score 15 and the opponent 0
2. It is his turn: we score 0 and the opponent 15

Therefore we need to keep track of both these chances. Now let's step back by one in the game

    { 15, 2 }

again two things can happen

1. It is our turn: we grab 15 and the opponent 2
2. It is his turn: the opponent grabs 15 and we score 2

Since each player has one goal: to score more than his opponent, it can't happen that a player grabs a value randomly or inefficiently.

Let us denote with \\( f_{max}(i,j) \\) the maximum we can score if it's our turn in the subinterval \\( [i,j] \\) of the array and with \\( g_{max}(i,j) \\) the maximum we can score if it's not our turn in the same interval. Let us also denote with \\( v_i \\) the \\( i^{th} \\) element in the array. It follows that

$$ f_{max}(i,j) = \max\{ v_i + g_{max}(i+1,j), v_j + g_{max}(i,j-1) \} \\ 
   g_{max}(i,j) = \min\{ f_{max}(i+1, j), f_{max}(i, j-1) \} $$

In both cases the goal is to maximize his own score and minimize the opponent's score. This recursion is well-suited to be implemented via dynamic programming.

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
using namespace std;

// Returns the maximum gain if we start with the first move or 
// if the opponent starts with the first move
pair<int,int> findOptimalSolution(const vector<int> nums) {
  vector<vector<int>> maximumGainMyTurn(nums.size(), vector<int>(nums.size(), 0));
  vector<vector<int>> maximumGainHisTurn(nums.size(), vector<int>(nums.size(), 0));

  // If one-element subinterval is in my turn, gain is the element itself
  for (int i = 0; i < nums.size(); ++i)
    maximumGainMyTurn[i][i] = nums[i];
  // No gain for me if one-element subinterval is in his turn
  
  for (int l = 2; l <= nums.size(); ++l) { // For each sublength

    for (int beg = 0; beg <= nums.size() - l; ++beg) {
      int end = beg + l - 1;

      // Calculate my maximum gain if this is my turn
      int maxGainMyTurn = max(nums[beg] + maximumGainHisTurn[beg + 1][end], 
      	                      nums[end] + maximumGainHisTurn[beg][end - 1]);
      // Calculate my maximum gain if this is his turn - Notice that the opponent's
      // objective is to MINIMIZE my gain
      int maxGainHisTurn = min(/* nums[beg] +*/ maximumGainMyTurn[beg + 1][end], 
      	                       /* nums[end] +*/ maximumGainMyTurn[beg][end - 1]);

      maximumGainMyTurn[beg][end] = maxGainMyTurn;
      maximumGainHisTurn[beg][end] = maxGainHisTurn;
    }
  }

  return make_pair(maximumGainMyTurn[0][nums.size() - 1], 
                   maximumGainHisTurn[0][nums.size() - 1]);
}

int main() {

  vector<int> numbers = { 1, 30, 100, 15, 2 };

  auto res = findOptimalSolution(numbers);

  cout << "{ ";
  copy(numbers.begin(), numbers.end(), ostream_iterator<int>(cout, " "));
  cout << "}\n";
  cout << "If we start with the first move I can score a maximum of " << res.first;
  cout << "\nIf the opponent starts with the first move "
       << "we can score a maximum of " << res.second;

  return 0;
}
{% endhighlight %}

The code computes the optimal maximum score in \\( O(N\log{N}) \\). The answer to the question

> Does the player who plays first always win?

is **no** since in the example provided two scores are returned: `47` and `101`. The first one is the score that we can reach if we start the game while the second is the score we can reach if our opponent starts. In the case above the second who plays with an optimal-seeking strategy always wins. Anyway to disprove the statement it would have sufficed to think of the case when an even-sized array of same elements is given: the players will score the same.