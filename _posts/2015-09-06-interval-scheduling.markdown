---
layout: post
title:  "Interval scheduling"
tags: algorithms dynamic-programming
---

[Interval scheduling](https://en.wikipedia.org/wiki/Interval_scheduling) is a class of problems where given time intervals have to be arranged in order to maximize or minimize a target function with no two intervals overlapping.

A common instance of the interval scheduling problem asks to find the maximum sum of weights in a set of overlapping intervals. Let's take the following intervals as an input: each interval has an associate weight/value which renders it more or less preferable compared to another one

$$
\begin{array}{c|lcr}
n & \text{Interval} & \text{Value} \\
\hline
1 & [0;5] & 10 \\
2 & [3;8] & 11 \\
3 & [4;7] & 6 \\
4 & [9;15] & 20 \\
5 & [15;17] & 4 \\
\end{array}
$$

There's an interesting dynamic programming solution to finding the maximum sum set of non-overlapping intervals. The key observation is that each interval may be added to the previous optimal set or not. Ordering the intervals by their `stop` value will help visualize this solution

![image](/images/posts/intervalscheduling1.png)

Every time an interval is picked up for being evaluated (intervals which finish before are chosen first) it can be skipped (no changes in the previous optimal value) or can be chosen. In case it is chosen all the intervals which overlap with this latter one need to be dropped. The condition for which a previous interval has to be dropped is having its `stop` value greater than the `start` value for the interval being evaluated.

If \\( g(i) \\) is the maximum sum we can get with the first \\( i \\) intervals ordered by their `stop` values, the problem can be expressed as

$$ 
\begin{cases}
g(0) = 0 \\[2ex]
g(i) = max\{ g(i-1), \ g(j) + v_i \} & \text{with 1 < $i$ < $N$}
\end{cases}
$$

with

$$ j = max(i) \ s.t. \ j_{stop} \le i_{start} $$

{% highlight c++ %}
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

struct Interval {
  int start;
  int stop;
  int value;
};

int maximumNonOverlappingValue(vector<Interval>& intervals) {

  // Sort intervals by stop value in O(N log N)
  sort(intervals.begin(), intervals.end(), [](auto int1, auto int2) {
    if (int1.stop < int2.stop)
      return true;
    else
      return false;
  });

  vector<int> maxWithIntervals(intervals.size() + 1, 0); // 1-based
  for (int i = 0; i < intervals.size(); ++i) {
    int opt1 = maxWithIntervals[i]; // Do not take this i-th interval
    int lo = 0, hi = i;
    while (lo <= hi) { // Binary search for the greatest stop value <= [i].start
      int mid = static_cast<int>(ceil((lo + hi) / 2.0f));
      if (intervals[mid].stop <= intervals[i].start)
        lo = mid + 1;
      else
        hi = mid - 1;
    }
    // (lo - 1) is the last valid value
    // maxWithIntervals is 1-based
    int opt2 = maxWithIntervals[lo] + intervals[i].value;
    maxWithIntervals[i+1] = max(opt1, opt2);
  }

  return maxWithIntervals.back();
}

int main() {
  vector<Interval> intervals = {
    // Start    End    Value
    {  0,       5,     10    },
    {  3,       8,     11    },
    {  4,       7,      6    },
    {  9,      15,     20    },
    { 15,      17,      4    }
  };

  cout << maximumNonOverlappingValue(intervals);

  return 0;
}
{% endhighlight %}

The complexity is \\( O(N \log{N}) \\) since sorting the intervals takes \\( O(N \log{N}) \\) and each binary search for the \\( j \\) intervals is \\( O(\log{N}) \\).