---
layout: post
title:  "Vector rotation"
tags: algorithms
---

> Write an efficient algorithm to perform arbitrary vector rotations. Time complexity should be \\( O(N) \\)

Solution I
==========

Trivial solution: use an auxiliary vector to rotate the vector by \\( k \\) positions

{% highlight c++ %}
template <typename T>
vector<T> rotate(vector<T>& v, int K) {
  assert(K >= 0);
  while (K > v.size()) // Error checking
    K %= v.size();
  vector<T> ret; 
  for (auto i = 0; i < v.size(); i++)
    ret.push_back(v[(i + K) % v.size()]);
  return ret; 
}
{% endhighlight %}

Time \\( O(N) \\), space \\( O(N) \\); it doesn't work with negative \\( k \\) values.

Solution II
===========
A better space-efficient algorithm to perform a vector rotation relies on three subvector inversions. Let's suppose we have a vector \\( v \\), `begin` and `end` positions \\( [i,j[ \\) and a split point \\( s \\).
We can construct the output vector \\( v' \\) as follows

$$ v' = \phi( \phi(v_{i,s}), \phi(v_{s,j}) ) $$

where \\( \phi(x) \\) is the reverse function.

{% highlight c++ %}
template <class BidirectionalIt>
void rotate(BidirectionalIt start, BidirectionalIt n_start, BidirectionalIt end) {
  static_assert(std::is_same<typename std::iterator_traits<BidirectionalIt>::iterator_category, 
                             std::bidirectional_iterator_tag>::value
             || std::is_same<typename std::iterator_traits<BidirectionalIt>::iterator_category, 
                             std::random_access_iterator_tag>::value,
                "Wrong iterator category");
  // vec_reverse(start, n_start);
  BidirectionalIt first = start;
  BidirectionalIt last = n_start - 1;
  while (first != last && last + 1 != first)
    std::iter_swap(first++, last--);
  // vec_reverse(n_start, end);
  first = n_start;
  last = end - 1;
  while (first != last && last + 1 != first)
    std::iter_swap(first++, last--);
  // vec_reverse(start, end);
  first = start;
  last = end - 1;
  while (first != last && last + 1 != first)
    std::iter_swap(first++, last--);
}
{% endhighlight %}

The algorithm runs in \\( O(N) \\) and \\( O(1) \\) space.

Solution III
============

A third more complex approach which often exhibits better data locality is now presented. The key to understanding it is considering three separate cases:

1. \\( s = \frac{j - i}{2} \\)

        begin = 0
        n_begin = split
        for i = 0 to split
            swap(begin, n_begin)
            begin++
            n_begin++

2. \\( s > \frac{j - i}{2} \\)

        begin = 0
        n_begin = split
        for i = 0 to dist(sj)
            swap(begin, n_begin)
            begin++
            n_begin++
        n_begin = split // Reset exhausted list
        // Repeat the process until case 3 applies

3. \\( s < \frac{j - i}{2} \\)

        begin = 0
        n_begin = split
        for i = 0 to dist(is)
            swap(begin, n_begin)
            begin++
            n_begin++
        split = n_begin
        // We now have a subinstance of the original problem


{% highlight c++ %}
template <class ForwardIt>
void rotate(ForwardIt start, ForwardIt n_start, ForwardIt end) {
  static_assert(std::is_same<typename std::iterator_traits<ForwardIt>::iterator_category, 
                             std::forward_iterator_tag>::value
             || std::is_same<typename std::iterator_traits<ForwardIt>::iterator_category, 
                             std::random_access_iterator_tag>::value,
                "Wrong iterator category");
  ForwardIt split = n_start;
  while (start != n_start) {
    std::iter_swap(start++, n_start++);
    if (n_start == end)
      n_start = split;
    else if (start == split)
      split = n_start;
  }
}
{% endhighlight %}

Algorithm is \\( O(N) \\).

A driver stub is also provided for completeness' sake

{% highlight c++ %}
int main() {
  std::vector<int> v{ 1, 2, 3, 4, 5 };
  rotate(v.begin(), v.begin() + 2, v.end());
  assert((v == decltype(v){ 3, 4, 5, 1, 2 }));

  v = { 1, 2, 3, 4, 5 };
  rotate(v.begin(), v.begin() + 3, v.end());
  assert((v == decltype(v){ 4, 5, 1, 2, 3 }));

  // Insertion sort (~ O(N^2))
  v = { 3, 1, 2, 6, 8 };
  for (auto i = v.begin(); i != v.end(); ++i)
    rotate(std::upper_bound(v.begin(), i, *i), i, i + 1);

  assert(std::is_sorted(v.begin(), v.end()));
}
{% endhighlight %}