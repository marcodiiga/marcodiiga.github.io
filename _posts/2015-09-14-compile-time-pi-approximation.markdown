---
layout: post
title:  "Compile-time pi approximation"
tags: algorithms
---

The *pi* constant is usually available in the `math.h` header

{% highlight c++ %}
#define _USE_MATH_DEFINES // Needed on MSVC
#include <cmath>
#define M_PI       3.14159265358979323846   // pi
{% endhighlight %}

Anyway besides being defined as the ratio between the circumference and diameter, the value of \\( \pi \\) can also be approximate with the help of [Leibniz's formula](https://en.wikipedia.org/wiki/Leibniz_formula_for_%CF%80)

$$ \sum_{n=0}^\infty \, \frac{(-1)^n}{2n+1} \;=\; \frac{\pi}{4} $$

and this can be computed in \\( O(n) \\) where \\( n \\) is the number of iterations at runtime or directly at compile-time moving the bulk of the work to the compiler itself with templates

{% highlight c++ %}
#include<iostream>

template <unsigned long long Iteration>
struct pi_calculator {
  static constexpr double value() {
    return (4.0 * ((Iteration % 2 == 0) ? 1.0 : -1.0) / (2.0 * Iteration + 1.0))
      + pi_calculator<Iteration - 1>::value();
  }
};

template <>
struct pi_calculator<0> {
  static constexpr double value() {
    return 4.0;
  }
};

template <unsigned long long Iterations>
struct compileTimePi {
  static constexpr double value() {
    return pi_calculator<Iterations>::value();
  }
};

double runtimeComputePi(const unsigned long long iterations) {
  double pi = 0;
  for (unsigned long long i = iterations; i > 0; --i) {
    pi += ((i % 2 == 0) ? 1.0 : -1.0) * 4.0 / (2.0*i + 1.0);
  }
  pi += 4.0;
  return pi;
}

int main() {
  const unsigned long long iterations = 1000u; // cfr. -ftemplate-depth
  double rtPi = runtimeComputePi(iterations);
  double ctPi = compileTimePi<iterations>::value();

  std::cout.precision(10);
  std::cout << "Runtime pi: \t\t" << rtPi << std::endl;
  std::cout << "Compile-time pi: \t" << ctPi << std::endl;

  return 0;
}
{% endhighlight %}
