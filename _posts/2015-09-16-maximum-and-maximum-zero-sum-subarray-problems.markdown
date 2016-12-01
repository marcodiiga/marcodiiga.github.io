---
layout: post
title:  "Maximum and maximum zero-sum subarray problems"
tags: algorithms
---

Maximum subarray problem
------------------------

The [maximum subarray problem](https://en.wikipedia.org/wiki/Maximum_subarray_problem) consists in finding the maximum contiguous subarray into a longer sequence of numbers. Given the following sequence of values

$$ V = \{ 1, 2, -4, 3, 6, -4, -1, 0, 4, 2, -4, 6 \} $$

the maximum subarray is {\\( 3, 6, -4, -1, 0, 4, 2, -4, 6 \\)} since it grants a grand-total of 12.

Kadane's algorithm is \\( O(n) \\) and allows for a quick identification of the maximum subarray

{% highlight c++ %}
tuple<int, int, int> maximumSubarrayLength(vector<int>& vec) {
  int total_max = 0;
  int current_max = 0;
  int current_max_start = 0, current_max_end = 0;
  for (int i = 0; i < vec.size(); ++i) {
    current_max += vec[i];
    if (current_max < 0) {
      // Skip last value which made the sum negative
      current_max_start = i + 1;
      current_max = 0; // Change to vec[i] if negative values
                       // are allowed
    }
    if (current_max > total_max) {
      total_max = current_max;
      current_max_end = i;
    }
  }
  return make_tuple(total_max, current_max_start, current_max_end);
}
{% endhighlight %}

Maximum zero-sum subarray problem
---------------------------------

The maximum zero-sum subarray problem is a modified statement from the previous problem: it asks to find the largest subarray whose elements add up to zero. The maximum zero-sum subarray for the sequence

$$ V = \{ 1, 2, -4, 3, 6, -4, -1, 0, 4, 2, -4, 6 \} $$

is {\\( -4, 3, 6, -4, -1, 0 \\)}. As usual a naive \\( O(n^2) \\) algorithm is available by exploring all the subintervals \\( [i;j] \\), anyway a \\( O(n) \\) algorithm based on sum hashing would be a better match. The algorithm keeps a running sum and stores intermediate sums into a hash table. In the following cases, a potential maximum is found

* If no maximum zero-sum subarray has been identified yet and there's a 0 element, the maximum zero-sum array is trivially composed of the 0 number itself
* If the running sum reaches zero from the beginning of the array, the longest zero-sum subarray consists of the entire array till the index we reached
* If a previous intermediate sum is found, it means that between the previous sum \\( X \\) and the current sum \\( X \\) there must have been an interval of values which sum to zero. Check for the maximum between the current maximum length and the length of this intermediate interval

{% highlight c++ %}
int longestSubarrayZeroSum(vector<int>& vec) {
  unordered_map<int, int> hm;
  int sum = 0;
  int max_len = 0;
  for (int i = 0; i < vec.size(); ++i) {
    sum += vec[i];

    // explicit 0-sum cases: a 1-length 0 element or
    // a sum of zero
    if (vec[i] == 0 && max_len == 0)
      max_len = 1;
    if (sum == 0)
      max_len = i + 1;

    auto it = hm.find(sum);
    if (it != hm.end())
      max_len = max(max_len, i - it->second);
    else
      hm[sum] = i;
  }
  return max_len;
}
{% endhighlight %}
