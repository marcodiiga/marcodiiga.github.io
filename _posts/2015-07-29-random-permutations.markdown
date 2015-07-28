---
layout: post
title:  "Generating random permutations"
tags: algorithms
---

Generating permutations given a list of items is a simple problem which can be
solved with a few lines algorithm. Anyway it is also one of the problems
that people often get wrong.

Given a set of items

$$ S = \{ 1, 2, 3 \} $$

generating a random permutation of the elements mean giving each possible permutation
(i.e. each one of the \\( n! = 3! = 6 \\) possible permutations) the same probability
of being generated.

The simplest algorithm that might come to mind is the following

{% highlight c++ %}
for(int i=0; i<N; ++i) {
  perm[i] = arr[i];
}

for(int i=0; i<N; ++i) {
  int rand = random(0, N); // Random number in [0;N[
  swap(arr[i], arr[rand]);
}
{% endhighlight %}

The algorithm above is **incorrect** and fails to deliver a uniform distribution
for all the permutations of a given set. The reason why this algorithm is incorrect
lies in the way permutations are generated.

Let's take, for instance, the complete tree for the previous given set of elements
(*click the image to enlarge it*)

<a href="/images/posts/randompermtree.png" target="_blank">![randompermtree](/images/posts/randompermtree.png)</a>

The final nodes which yield the \\( 1,2,3 \\) permutation have been colored in red,
while the nodes for \\( 2,1,3 \\) have been colored in green.

Every edge has a probability of \\( \frac{1}{3} \\) and thus probabilities are

$$ \{1,2,3\} = 4 \cdot {\frac{1}{3} \cdot \frac{1}{3} \cdot \frac{1}{3}} \\
\{2,1,3\} = 5 \cdot {\frac{1}{3} \cdot \frac{1}{3} \cdot \frac{1}{3}} $$

Therefore this is not a uniform distribution of probabilities for random permutations.

A better algorithm is

{% highlight c++ %}
for(int i=0; i<N; ++i) {
  perm[i] = arr[i];
}

for(int i=0; i<N; ++i) {
  int rand = random(i, N); // Random number in [i;N[
  swap(arr[i], arr[rand]);
}
{% endhighlight %}

Let's calculate the probability of generating the permutation \\( \{1,2,3\} \\):
the probability for the first element is \\( \frac{1}{3} \\), while the probability
for the second number is \\( \frac{2}{3} \cdot \frac{1}{2} = \frac{1}{3}\\) i.e.
the probability of not being extracted as first number and the probability of being
extracted in the second position. The third element's probability is, similarly,
\\( \frac{2}{3} \cdot \frac{1}{2} = \frac{1}{3}\\). Every element has probability
\\( \frac{1}{3} \\) and thus this is a uniform distribution.
