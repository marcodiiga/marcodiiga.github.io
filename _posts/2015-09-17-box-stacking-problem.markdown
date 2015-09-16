---
layout: post
title:  "Box stacking problem"
tags: algorithms dynamic-programming
---

Instances of the *box stacking* problem are usually of the following form

> You're given a set of boxes \\( b_1 \cdots b_n \\), each one has an associated width, height and depth. Find the highest possible stack of boxes subject to the constraints that a box on top of another should have both dimensions of its base less than the box under it and no box can be used more than once. Boxes can be rotated.

An example follows: given two boxes (\\( w,h,d \\))

$$ b_1 = \{5,5,1\} \\
b_2 = \{4,5,2\} $$

the maximum height is 6 since \\( b_1 \\) can be put on the bottom of \\( b_2 \\). The boxes are rotated to have bases \\( (5 \times 5) \\) and \\( 4 \times 2 \\).

The mathematical formulation of the dynamic programming solution follows: let \\( H(j,R) \\) be the tallest stack of boxes with \\( j \\) on top with rotation \\( R \\).

$$  
  H(j,R) = \begin{cases}
           0 & \mbox{if $j=0$} \\[2ex]
           max_{j<i \ with \ w_i < w_j, d_i < d_j}(H(i,R)+h_j) & \mbox{if $j>0$}
           \end{cases}
$$

Particular attention has to be paid for the rotation code. If the problem allows for any number of rotations along any axis **and** no box can be used more than once (as in the instance above), the rotation has to keep a reference to its originating box. This is not immediately obvious but has to be kept in mind nonetheless (*hint: the problem is better visualized with multiple instances of the same \\( \{ 1,2,3 \} \\) box*).

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Box {
  Box() = default;
  Box(int w, int h, int d) :
    width(w), height(h), depth(d)
  {
    boxId = ++boxIncrementalId; // Uniquely identifies this box
  }
  int width, height, depth;
  int boxId;
  static int boxIncrementalId;
};

int Box::boxIncrementalId = -1;

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

  // Sort rotations by base (biggest goes first)
  sort(allRotations.begin(), allRotations.end(), [](const auto& r1,
    const auto& r2) {
    if (r1.width * r1.depth > r2.width * r2.depth)
      return true;
    else
      return false;
  });

  // Initialize the maximum height with each box on top to its height 
  // (i.e. one box only)
  vector<int> maximumHeightForTopBox;
  for_each(allRotations.begin(), allRotations.end(), [&](auto& rot) {
    maximumHeightForTopBox.push_back(rot.height);
  });

  auto hasABiggerBase = [](const Box& rot1, const Box& rot2) { // Utility function
    if (rot1.width > rot2.width && rot1.depth > rot2.depth)
      return true;
    else
      return false;
  };

  // Bottom up calculation for the maximum height with i-th box on top
  for (int i = 1; i < allRotations.size(); ++i) {
    // Check previous (bigger) boxes and try to pile them up
    for (int j = 0; j < i; ++j) {

      // Three conditions to pile i on top of j:
      //  - j has a bigger base than i
      //  - it is actually convenient to pile i on j than leaving i alone
      //  - i and j are NOT two rotations of the same box ~ this condition can
      //    be commented out if the problem were to be allowed multiple instances
      //    of the same box
      if (hasABiggerBase(allRotations[j], allRotations[i]) == true &&
        maximumHeightForTopBox[j] + allRotations[i].height > maximumHeightForTopBox[i]
        && allRotations[j].boxId != allRotations[i].boxId) {
        maximumHeightForTopBox[i] = maximumHeightForTopBox[j] +
          allRotations[i].height;
      }
    }
  }

  // Find the maximum height and return it
  int max = -1;
  for_each(maximumHeightForTopBox.begin(), maximumHeightForTopBox.end(),
    [&](auto v) {
    if (max < v)
      max = v;
  });

  return max;
}

int main() {
  vector<Box> boxes = { { 5,5,1 },{ 4,5,2 } };
  cout << maximumHeightBoxStack(boxes); // 6
  return 0;
}
{% endhighlight %}

Running time is \\( O((n \mid R \mid)^2) \\) but since we kepth \\( R=3 \\), \\( O(n^2) \\).

References
==========

* [Handouts fall10 @ csail.mit.edu](http://courses.csail.mit.edu/6.006/fall10/)