---
layout: post
title:  "Binary tree to singly and doubly linked list"
tags: c++ algorithms
---

> Given a binary tree, flatten it in-place to a singly linked list in preorder fashion.
> e.g.
>
>         1                    1
>        / \                    \
>       2   5                    2
>      / \   \                    \
>     3   4   6                    3
>                                   \
>                                    4
>                                     \
>                                      5
>                                       \
>                                        6

A single traversal of the binary tree is sufficient if paired with a `next` pointer to remember the next element that will have to become the right one

{% highlight c++ %}
void bt_to_sll(Node *root) {

  if (!root)
    return;

  static Node *next = nullptr;

  if (root->right)
    bt_to_sll(root->right);

  if (root->left)
    bt_to_sll(root->left);
  root->left = nullptr;

  if (next)
    root->right = next;
  next = root;
}
{% endhighlight %}

> Given a binary tree, convert it to a doubly linked list in-place in an in-order fashion.
> e.g.
>
>          10                 
>        /    \      
>       12     15    
>      / \     /      
>     25  30  36      
>            
>     25 <-> 12 <-> 30 <-> 10 <-> 36 <-> 15

The algorithm is similar to the previous one: it is traversed `left` first and a `prev` pointer is kept. An additional way to return the new list head is needed since it no longer matches the root of the tree.

{% highlight c++ %}
void bt_to_dll(Node *root, Node **list_head) {

  if (!root)
    return;

  static Node *prev = nullptr;

  if (root->left)
    bt_to_dll(root->left, list_head);

  if (prev) {
    root->left = prev;
    prev->right = root;
  } else
    *list_head = root; // I'm the root of the dll

  prev = root;

  if (root->right)
    bt_to_dll(root->right, list_head);
}
{% endhighlight %}

Both algorithms are \\( O(N) \\).
