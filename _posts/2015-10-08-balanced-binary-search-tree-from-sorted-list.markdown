---
layout: post
title:  "Balanced binary search tree from sorted list"
tags: algorithms
---

> Given a sorted list (or a sorted array) as input, generate a balanced binary search tree in O(n)

Iteratively extracting every single element from the list/array and inserting it into a balanced binary search tree implementation (e.g. [red/black trees](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)) won't work since it's \\( O(N \log{N}) \\). A better solution would be to recursively create the left and right subtrees from the bottom up and inserting the subtrees' roots each time by advancing a single pointer in the list.

Pseudocode and code follow

    recursiveCreateBST(pointer_to_first_element, number_of_nodes_in_the_tree)
      
      if(number_of_nodes_in_the_tree == 0)
        return NULL // no node was inserted for a degenerate tree
      
      leftSubtree = recursiveCreateBST(pointer_to_first_element, 
                                       number_of_nodes_in_the_tree / 2)
      root = createRootNode(pointer_to_first_element)
      root.left = leftSubtree
      pointer_to_second_element = advance_pointer(pointer_to_first_element)

      root.right = recursiveCreateBST(pointer_to_second_element, 
                                      number_of_nodes_in_the_tree - (leftSubtree.size) - 1)
      
      return root


{% highlight c++ %}
#include <iostream>
#include <list>
#include <memory>
#include <cassert>
#include <algorithm>
#include <functional>
#include <stack>
using namespace std;

struct Node {
  Node(int v) : val(v) {}
  int val;
  unique_ptr<Node> left;
  unique_ptr<Node> right;
};

unique_ptr<Node> sortedListToBST(list<int> ll) {
  assert(is_sorted(ll.begin(), ll.end()));

  function<unique_ptr<Node>(list<int>::iterator& node, size_t nOfNodes)>
    recursiveCreateBST = [&](list<int>::iterator& node, size_t nOfNodes) {
    if (nOfNodes == 0)
      return unique_ptr<Node>();
    // Recursively create the left subtree    
    unique_ptr<Node> left = recursiveCreateBST(node, nOfNodes / 2);
    // Create the root of this subtree
    unique_ptr<Node> root = make_unique<Node>(*node);
    root->left = std::move(left);
    ++node;
    // Recursively create the right subtree
    root->right = std::move(recursiveCreateBST(node, nOfNodes - (nOfNodes / 2) - 1));
    return std::move(root);
  };

  auto firstNode = ll.begin();
  return recursiveCreateBST(firstNode, ll.size());
}

int main() {

  list<int> ll = { 1, 2, 3, 4 };
  unique_ptr<Node> root = sortedListToBST(ll);
  // In-order print
  auto inOrderPrint = [](const unique_ptr<Node>& root) {
    stack<Node*> nodes;
    nodes.push(root.get());
    Node *topNode = nodes.top();
    while (topNode->left) {
      nodes.push(topNode->left.get());
      topNode = topNode->left.get();
    }
    while (nodes.empty() == false) {
      topNode = nodes.top();
      cout << topNode->val << " ";
      nodes.pop();
      if (topNode->right) {
        nodes.push(topNode->right.get());
        topNode = nodes.top();
        while (topNode->left) {
          nodes.push(topNode->left.get());
          topNode = topNode->left.get();
        }
      }
    }
  };
  inOrderPrint(root);

  return 0;
}
{% endhighlight %}

This code can safely be used to build from the bottom up a balanced BST from either a **sorted** list or a **sorted** array. The iterative in-order print is taken from [this article]({% post_url 2015-09-13-binary-tree-traversal-techniques %}).

The complexity of the algorithm in `sortedListToBST` can be calculated with [Master theorem](https://en.wikipedia.org/wiki/Master_theorem). The problem is in the form

$$ T(n) = a \; T\!\left(\frac{n}{b}\right) + f(n) $$

with the assumptions of \\( a = 2, \ b = 2 \\). Since

$$ \quad f(n) \in O\left( n^{\log_b a - \varepsilon} \right) $$

(\\( f(n) \in O(1) \\)), therefore it holds

$$ \quad T(n) \in \Theta\left( n^{\log_b a} \right) $$

and the algorithm is \\( O(n) \\) as expected.

