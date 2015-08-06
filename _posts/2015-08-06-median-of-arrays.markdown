---
layout: post
title:  "Median of arrays"
tags: algorithms
---

Finding the median of a sorted array is a trivial matter of either getting the
middle element or the average of the closest-to-midpoint elements.

If an array isn't sorted a [kth order statistic]({% post_url 2015-08-05-kth-order-statistic %}) method might be used. Of particular interest is the **median of medians** algorithm which allows finding the *kth* order statistic in \\( O(n) \\).

Median of two equal-sized sorted arrays
---------------------------------------

Things change if we need to find the median of two arrays. The naive solution is to just append an array to the other and apply one of the techniques cited above. Anyway if the two arrays are sorted and they have the same size one could do even better.

A simple merging of the two arrays (in a fashion similar to how mergesort works in the merging phase) would be \\( O(n) \\) and it would prove to be considerably easier to implement than many of the \\( O(n) \\) techniques for *kth* order statistic.

Anyway one could do even better by exploiting the following observation: the median element has the same number of elements at its right and left. If the arrays have the same size and are sorted, e.g.

$$ A = \{1,2,3\} \\
B = \{4,5,6\}$$

we can take the respective medians in \\( O(1) \\): 2 and 5. Now three things can happen

* If the medians are equal, it means that the mean of the same value twice will also be the median of the merged arrays. Thus we're done.

* If the first median is less than the second (as in the example above), the median of the entire sequence obtained by merging the arrays will lie in the interval `[2;3]` or `[4;5]`. Thus we start the process again with these two arrays as input.

* If the first median is greater than the second, the process starts again searching in the interval `[start1,median1]` and `[median2;end2]`.

The code to get this process right follows

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

using vIt = vector<int>::iterator;

int getMedian(vIt vstart, vIt vend) {
  size_t vecSize = distance(vstart, vend);
  if (vecSize <= 0)
    return -1;
  if (vecSize % 2 == 0)
    return *(vstart + (vecSize / 2 - 1)) + *(vstart + (vecSize / 2) / 2);
  else
    return *(vstart + (vecSize / 2));
}

int getMedian(vIt v1start, vIt v1end, vIt v2start, vIt v2end) {

  size_t vecSize = distance(v1start, v1end);

  if (vecSize == 0)
    return -1;

  if (vecSize == 1)
    return (*v1start + *v2start) / 2;

  if (vecSize == 2)
    return (max(*v1start, *v2start) + min(*(v1start + 1), *(v2start + 1))) / 2;

  int median1 = getMedian(v1start, v1end);
  int median2 = getMedian(v2start, v2end);

  if (median1 == median2)
    return median1;

  if (median2 > median1) { // Recur in [median1..v1end] [v2start..median2]
    if (vecSize % 2 == 0)
      return getMedian(v1start + (vecSize / 2) - 1, v1end, 
                       v2start, v2end - (vecSize / 2) + 1);
    else
      return getMedian(v1start + (vecSize / 2), v1end, 
                       v2start, v2end - (vecSize / 2) - 1);
  }

  if (median2 < median1) { // Recur in [v1start..median1] [median2..v2end]
    if (vecSize % 2 == 0)
      return getMedian(v1start, v1start + (vecSize / 2) + 1, 
                       v2start + (vecSize / 2) - 1, v2end);
    else
      return getMedian(v1start, v1start + (vecSize / 2) + 1, 
                       v2start + (vecSize / 2), v2end);
  }

  return -1; // Should never get here
}

int getMedian(vector<int>& vec1, vector<int>& vec2) {
  return getMedian(vec1.begin(), vec1.end(), vec2.begin(), vec2.end());
}

int main() {
  vector<int> vec1 = { 1, 2, 7, 8, 11 };
  vector<int> vec2 = { 4, 9, 10, 12, 22 };

  if (vec1.size() != vec2.size())
    cout << "Vectors have different sizes";
  else
    cout << "Median is " << getMedian(vec1, vec2); // 8

  return 0;
}
{% endhighlight %}


Thanks to the [divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer) paradigm we can exploit the fact that merging subproblems together with the three rules written above is easier than merging the entire arrays in one time. Complexity is \\( O(\log{n}) \\).


A very similar way to search for the median in \\( O(\log{n}) \\) time works with the same observations. Here's the simplified pseudocode

	findMedian(vector1, vector2)
	  if(vector1 is exhausted)
	    findMedian(vector2, vector1); // Switch parts
	  i = median index of vector1
	  j = j - i - 1 // Corresponding index to i in vector2
	  if(vector1[i] >= vector2[j] && vector1[i] <= vector2[j])
	    return (vector[i] + vector2[j]) / 2;
	  else if(vector1[i] < vector2[j])
	    findMedian(upper_half(vector1), vector2);
	  else
	    findMedian(lower_half(vector1), vector2);


The key observations are the same as the code above:

* If the arrays have the same median, return it

* If median1 is less than median2 subsearch into array1's upper half and, if not there, subsearch into array2's lower half

* If median1 is greater than median2 subsearch into array1's lower half and, if not there, subsearch into array2's upper half

Both methods have a divide and conquer approach with binary searches throughout the potentially valid subarrays.

Median of two sorted arrays with different sizes
------------------------------------------------

An extension of the previous methods can be applied if the arrays are sorted but have different sizes. Considerations for the general case withstand, therefore a linear merging is another available option in \\( O(n) \\) time, plus all the other techniques. Anyway even with different sizes we can achieve logarithmic time \\( O(\log{n} + \log{m}) \\) where \\( n \\) and \\( m \\) are the sizes of the arrays.

There are many base cases to keep in mind this time and it can be confusing to list them all and get them right.

First an assumption is made: if it is possible to identify a smaller and a bigger vector, so be it. If the two arrays are equal it doesn't matter which one is marked as bigger. This helps reduce code complexity since every assumption that will follow assumes two arrays *A* and *B* where *A* is the shorter one.

1. If the two arrays have 1 element each, the mean of their elements is taken as median

2. If the two arrays have 2 elements each, the maximum of the first indices and the minimum of the last indices are averaged to get the median

3. If one of the two (*A*) has size 1, if *B* has an even number of elements we're in the following situation

	$$ A = \{1\} \\
	B = \{2,3,4,5\}$$

	if *A*'s element is less than central element 3 (as it is in the example above), the median is exactly 3. If *A*'s element had been between 3 and 4, the median would have been *A*'s element. Otherwise 4 is the median.

4. If one of the two (*A*) has size 1 and *B* has an odd number of elements

	$$ A = \{1\} \\
	B = \{2,3,4\}$$

	if 1 is smaller than 2 then the median is the average between 2 and 3. Had it been between 2 and 4, the median would have been the average between 1 and 3. Otherwise the median is the average between 1 and 4.

5. If one of the two (*A*) has size 2, if *B* has an even size

	$$ A = \{1,2\} \\
	B = \{3,4,5,6\}$$

	then the median is the median of the following numbers: 4, 5, the maximum between 1 and 3 (3) and the minimum between 2 and 6 (2).

6. If one of the two (*A*) has size 2 and *B* has an odd size

	$$ A = \{1,2\} \\
	B = \{3,4,5\}$$

	then the median is the median of the following numbers: 4, the maximum between 1 and 3 (3) and the minimum between 2 and 5 (2).

The recursive normal cases are very similar to the case with equal sized arrays: medians of the two vectors are found and compared. If *A*'s median is less than *B*'s median then the search continues in `[median1...Aend]` and `[Bstart...median2]`. The same number of elements dropped from *A*'s left side `median1 - start` are also dropped from *B*'s right side. The opposite happens if *A*'s median is greater than *B*'s median. The recursion ends in one of the base cases or if the medians are equal.

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

using vIt = vector<int>::iterator;

int getMedian(vIt vstart, vIt vend) {
  size_t vecSize = distance(vstart, vend);
  if (vecSize <= 0)
    return -1;
  if (vecSize % 2 == 0)
    return *(vstart + (vecSize / 2 - 1)) + *(vstart + (vecSize / 2) / 2);
  else
    return *(vstart + (vecSize / 2));
}

int medianOf4(int a, int b, int c, int d) {
  vector<int> v = { a, b, c, d };
  sort(v.begin(), v.end());
  return (v[1] + v[2]) / 2;
}

int medianOf3(int a, int b, int c) {
  vector<int> v = { a, b, c };
  sort(v.begin(), v.end());
  return v[1];
}

int medianOfVector(vector<int>& v) {
  if (v.size() % 2 == 0)
    return (v[v.size() / 2 - 1] + v[v.size() / 2]) / 2;
  else
    return v[v.size() / 2];
}

int getMedian(vIt v1start, vIt v1end, vIt v2start, vIt v2end) {

  // For clarity's sake
  size_t size1 = distance(v1start, v1end);
  size_t size2 = distance(v2start, v2end);

  vector<int> small, big;
  size_t smallSize, bigSize;
  if (size1 < size2) {
    small.insert(small.begin(), v1start, v1end);
    big.insert(big.begin(), v2start, v2end);
    smallSize = size1;
    bigSize = size2;
  } else {
    small.insert(small.begin(), v2start, v2end);
    big.insert(big.begin(), v1start, v1end);
    smallSize = size2;
    bigSize = size1;
  }

  // Handle base cases
  if (smallSize == 1 && bigSize == 1) { // 1
    return (small[0] + big[0]) / 2;
  }
  if (smallSize == 2 && bigSize == 2) { // 2
    return (max(small[0], big[0]) + min(small[1], big[1])) / 2;
  }
  if (smallSize == 1 || bigSize == 1) {
    
    if (bigSize % 2 == 0) { // 3
      if (small[0] < big[bigSize / 2 - 1])
        return big[bigSize / 2 - 1];
      else if (small[0] >= big[bigSize / 2 - 1] && small[0] <= big[bigSize / 2])
        return small[0];
      else
        return big[bigSize / 2];
    } else { // 4
      if (small[0] < big[bigSize / 2 - 1])
        return (big[bigSize / 2 - 1] + big[bigSize / 2]) / 2;
      else if (small[0] >= big[bigSize / 2 - 1] && small[0] <= big[bigSize / 2 + 1])
        return (small[0] + big[bigSize / 2]) / 2;
      else 
        return (big[bigSize / 2] + big[bigSize / 2 + 1]) / 2;
    }
  }
  if (smallSize == 2 || bigSize == 2) {
    vector<int> small, big;
    size_t smallSize, bigSize;
    if (size1 == 2) {
      copy(v1start, v1end, small.begin());
      copy(v2start, v2end, big.begin());
      smallSize = size1;
      bigSize = size2;
    }
    else {
      copy(v2start, v2end, small.begin());
      copy(v1start, v1end, big.begin());
      smallSize = size2;
      bigSize = size1;
    }
    if (bigSize % 2 == 0) { // 5
      return medianOf4(max(small[0], big[bigSize / 2 - 2]), min(small[1], 
        big[bigSize / 2 + 1]), big[bigSize / 2 - 1], big[bigSize / 2]);
    }
    else { // 6
      return medianOf3(max(small[0], big[bigSize / 2 - 1]), min(small[1],
        big[bigSize / 2 + 1]), big[bigSize / 2]);
    }
  }

  // Normal cases
  int median1 = medianOfVector(small);
  int median2 = medianOfVector(big);
  
  if (median1 == median2)
    return median1;

  if (median2 > median1) { // Recur in [median1..v1end] [v2start..median2]
    int dropOffset = small.size() / 2;
    return getMedian(small.begin() + dropOffset, small.end(), 
                     big.begin(), big.end() - dropOffset);
  }

  if (median2 < median1) { // Recur in [v1start..median1] [median2..v2end]
    int dropOffset = small.size() / 2;
    return getMedian(small.begin(), small.end() - dropOffset, 
                     big.begin() + dropOffset, big.end());
  }

  return -1; // Should never get here
}

int getMedian(vector<int>& vec1, vector<int>& vec2) {
  return getMedian(vec1.begin(), vec1.end(), vec2.begin(), vec2.end());
}

int main() {
  vector<int> vec1 = { 1, 2, 7, 8, 11 };
  vector<int> vec2 = { 4, 9, 10, 12, 22 };

  cout << getMedian(vec1, vec2);

  return 0;
}
{% endhighlight %}

Complexity is \\( O(\log{m} + \log{n}) \\) where \\( m,n \\) are the lengths of the vectors.