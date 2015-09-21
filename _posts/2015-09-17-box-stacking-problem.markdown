---
layout: post
title:  "Box stacking problem"
tags: algorithms dynamic-programming
---

Instances of the *box stacking* problem are usually of the following form

> You're given a set of boxes \\( b_1 \cdots b_n \\), each one has an associated width, height and depth. Find the highest possible stack of boxes subject to the constraints that a box on top of another should have both dimensions of its base less than the box under it. Boxes can be rotated.

An example follows: given two boxes (\\( w,h,d \\))

$$ b_1 = \{5,5,1\} \\
b_2 = \{4,5,2\} $$

the maximum height is 7 since \\( b_2 \\) can be used twice. The boxes are rotated to have bases \\( (4 \times 5) \\) and \\( 2 \times 4 \\).

The mathematical formulation of the dynamic programming solution follows: let \\( H(j,R) \\) be the tallest stack of boxes with \\( j \\) on top with rotation \\( R \\).

$$  
  H(j,R) = \begin{cases}
           0 & \mbox{if $j=0$} \\[2ex]
           max_{j<i \ with \ w_i < w_j, d_i < d_j}(H(i,R)+h_j) & \mbox{if $j>0$}
           \end{cases}
$$

Particular attention has to be paid for the rotation code. If the problem allows for any number of rotations along any axis **and** no box can be used more than once (as in the instance above), the rotation has to keep a reference to its originating box and disallow the same box to appear more than once in the stack associated with every dynamic programming substate. This is not immediately obvious but has to be kept in mind nonetheless (*hint: the problem is better visualized with multiple instances of the same \\( \{ 1,2,3 \} \\) box*).

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Box {
  int width, height, depth;
};

int maximumHeightBoxStack(const vector<Box>& boxes) {
  // Generate all meaningful rotations for each box
  vector<Box> allRotations(6 * boxes.size());
  for (int i = 0; i < boxes.size(); ++i) {
    allRotations[6 * i] = boxes[i]; // Original
    allRotations[6 * i + 1] = boxes[i]; // 90 degrees original
    swap(allRotations[6 * i + 1].width, allRotations[6 * i + 1].depth);
    allRotations[6 * i + 2] = boxes[i]; // Rotation 1
    swap(allRotations[6 * i + 2].width, allRotations[6 * i + 2].height);
    allRotations[6 * i + 3] = allRotations[6 * i + 2]; // 90 degrees rot1
    swap(allRotations[6 * i + 3].width, allRotations[6 * i + 3].depth);
    allRotations[6 * i + 4] = boxes[i]; // Rotation 2
    swap(allRotations[6 * i + 4].height, allRotations[6 * i + 4].depth);
    allRotations[6 * i + 5] = allRotations[6 * i + 4]; // 90 degrees rot2
    swap(allRotations[6 * i + 5].width, allRotations[6 * i + 5].depth);
  }

  // Sort rotations by base (biggest goes first). This avoids checking
  // against previous boxes to achieve NlogN
  sort(allRotations.begin(), allRotations.end(), [](const auto& r1,
    const auto& r2) {
    if (r1.width * r1.depth > r2.width * r2.depth)
      return true;
    else
      return false;
  });

  // Initialize the maximum height with each box on top to its height 
  // (i.e. one box only)
  vector<int> topIndexForStackOfBoxes(allRotations.size() + 1, -1);
  vector<int> predecessorBox(allRotations.size() + 1, -1);
  // How much can a box width/depth increment and still fit within the chain
  vector<pair<int, int>> margin(allRotations.size() + 1, make_pair<int, int>(-1, -1)); 

  // Utility function
  //  -1 -> first rotation has a bigger base
  //   0 -> no winner for all dimensions
  //   1 -> second rotation has a bigger base
  auto compareBases = [](const Box& rot1, const Box& rot2) {
    if (rot1.width > rot2.width && rot1.depth > rot2.depth)
      return -1;
    else if (rot1.width < rot2.width && rot1.depth < rot2.depth)
      return 1;
    else
      return 0;
  };

  int maximumNumberOfBoxes = 0; // Maximum number of boxes found to pile
                                // for the maximum height

  // Bottom up calculation for the maximum height with i-th box on top
  for (int i = 0; i < allRotations.size(); ++i) {

    // Find the longest series of boxes with a box with a greater
    // base than this one's (where it could fit)
    int lo = 1;
    int hi = maximumNumberOfBoxes;
    bool discardElement = false;
    while (lo <= hi) {
      int mid = static_cast<int>(ceil((lo + hi) / 2.0f));
      const Box& rot1 = allRotations[topIndexForStackOfBoxes[mid]];
      const Box& rot2 = allRotations[i];
      int opt = compareBases(rot1, rot2);
      if (opt == -1)
        lo = mid + 1;
      else if (opt == 1)
        hi = mid - 1;
      else {
        lo = mid;
        // This code allows for box substitution in case a box fits in
        // the chain in the same position of another but has greater
        // dimensions (and it still fits the predecessor)
        if ((rot2.width == rot1.width && rot2.depth > rot1.depth 
          && (margin[mid].second == -1 || margin[mid].second >= 
                                          (rot2.depth - rot1.depth))) == false &&
          (rot2.depth == rot1.depth && rot2.width > rot1.width 
            && (margin[mid].second == -1 || margin[mid].first >= 
                                            (rot2.width - rot1.width))) == false)
          discardElement = true; // Not worth of consideration
        break; // In both cases break
      }
    }
    if (discardElement)
      continue;

    if (maximumNumberOfBoxes < lo)
      maximumNumberOfBoxes = lo;
    predecessorBox[lo] = topIndexForStackOfBoxes[lo - 1];
    topIndexForStackOfBoxes[lo] = i; // Store the ith element

    if (predecessorBox[lo] != -1) { // Update margins
      int parentWidth = allRotations[predecessorBox[lo]].width;
      int parentDepth = allRotations[predecessorBox[lo]].depth;
      int childWidth = allRotations[topIndexForStackOfBoxes[lo]].width;
      int childDepth = allRotations[topIndexForStackOfBoxes[lo]].depth;
      margin[lo].first = parentWidth - 1 - childWidth;
      margin[lo].second = parentDepth - 1 - childDepth;
    }
  }

  // Reconstruct the maximum height and return it
  int index = topIndexForStackOfBoxes[maximumNumberOfBoxes];
  int max = 0;
  while (index != -1) {
    max += allRotations[index].height;
    index = predecessorBox[maximumNumberOfBoxes];
    --maximumNumberOfBoxes;
  }

  return max;
}

int main() {
  vector<Box> boxes = { { 5,5,1 },{ 4,5,2 } };
  cout << maximumHeightBoxStack(boxes); // 6
  return 0;
}
{% endhighlight %}

Running time is \\( O(n \log n) \\), an easier \\( O(n^2) \\) is also available which doesn't require binary search nor the complex margin-checking code.

References
==========

* [Handouts fall10 @ csail.mit.edu](http://courses.csail.mit.edu/6.006/fall10/)