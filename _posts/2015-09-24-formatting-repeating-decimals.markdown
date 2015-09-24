---
layout: post
title:  "Formatting repeating decimals"
tags: algorithms
---

> Write out the decimal number obtained by dividing two given integers A and B. The number might be a recurring decimal, e.g. 1 / 3 = 0.(3)

Rational numbers can be written as *ratio* of two integers while irrational numbers like \\( \pi \\) cannot.
In order to format the decimal number into a string one could write code to perform a simple long division and checking for the maximum repeating substring in its decimal digits. A smarter approach though would be to notice that while doing decimal long divisions between the remainder and the divisor, the operation is similar to a state machine: if \\( n = \frac{a}{b} \\), its recurring decimal cycle (if present at all) can be at most \\( b-1 \\) long. This follows logically from the definition of long division.

Therefore the simplest algorithm to identify such cycles would be to keep an hash table of the remainders calculated and identify a cycle when a remainder has been already encountered

{% highlight c++ %}
#include <iostream>
#include <vector>
#include <string>
#include <sstream>
#include <unordered_map>
using namespace std;

typedef unsigned long long ull;

string divisionToString(const ull A, const ull B) {

  stringstream ss;
  ss << A / B;

  ull remainder = A % B;
  if (remainder == 0)
    return ss.str();
  else
    ss << ",";

  unordered_map<ull, ull> remainders; // remainder and position in decimals
  vector<ull> decimals;
  do {    
    remainders.emplace(remainder, decimals.size());
    decimals.push_back((remainder * 10) / B);
    remainder = (remainder * 10) % B;
  } while (remainder != 0 && remainders.find(remainder) == remainders.end());
  if (remainder == 0) {
    for (auto& v : decimals)
      ss << v;
    return ss.str();
  }

  ptrdiff_t beginPeriod = remainders.find(remainder)->second;
  ptrdiff_t endPeriod = decimals.size() - 1;
  for (ull i = 0; i < decimals.size(); ++i) {
    if (i == beginPeriod)
      ss << "(";
    ss << decimals[i];
    if (i == endPeriod)
      ss << ")";
  }

  return ss.str();
}

int main() {
  
  const ull A = 3227;
  const ull B = 555;

  cout << divisionToString(A, B); // 5,8(144)

  return 0;
}
{% endhighlight %}


Calculating period's digits
---------------------------

We might be interested at times into precalculating the number of digits that come before the period or that form the period itself. Number theory distinguishes between three kind of rational numbers:

* Regular numbers whose decimal expansion is finite (e.g. \\( 1/2 = 0.5 \\)). Let \\( r \equiv p/q \\), after factoring common multiples it follows that these numbers are of the form

$$ r=\frac{p}{2^\alpha5^\beta} \quad p \not\equiv 0 \pmod {2,5}$$

* Nonregular numbers whose decimal expansion is formed by a recurring period only. These are formed by having prime factors [coprime](https://en.wikipedia.org/wiki/Coprime_integers) to 10

* Nonregular numbers with non-cycling digits before the period (also called *antiperiod*) and then having a recurring period. These have both coprime factors and factors of the form in the first bullet point.

Therefore to know the length of the period one should first check if the denominator has 10-coprime factors. If such factors are present the length of the period is given by the [multiplicative order](https://en.wikipedia.org/wiki/Multiplicative_order) of the denominator, i.e. to the maximum \\( k \\) given by the [discrete logarithm](https://en.wikipedia.org/wiki/Discrete_logarithm)

$$ 10^k \equiv 1 \pmod f $$

where \\( f \\) is a 10-coprime factor and \\( k \\) is the minimum positive integer value for which the equivalence is verified.
The length of the antiperiod is given by the maximum power factor between 2 and 5 factors of the denominator **after** factoring common ones out with the numerator.

The following code formats a rational number given in the same form as before by applying the concepts explained above and using a trial-factoring algorithm together with a simple discrete logarithm implementation and a fast (but quite limited due to the huge numbers that might be involved with bigger inputs) [exponentiation by squaring]({% post_url 2015-07-22-integer-arithmetic-using-bit-operations %})

{% highlight c++ %}
typedef unsigned long long ull;

// Exponentiation by squaring
ull ebs(ull x, ull y) {

  ull result = 1;
  while (y != 0) {

    if (y % 2 != 0)
      result *= x;

    x *= x;
    y >>= 1;
  }

  return result;
}

string divisionToStringCycleLength(const ull A, const ull B) {

  stringstream ss;
 
  auto computePrimeFactors = [](ull N, ull limit = -1) {
    map<ull, ull> factors;
    while (N % 2 == 0) {
      ++factors[2];
      N /= 2;
    }
    if (limit == 2)
      return factors;
    ull sqroot = static_cast<ull>(sqrt(N));
    for (ull i = 3; i <= sqroot; ++i) {
      while (N % i == 0) {
        ++factors[i];
        N /= i;
      }
      if (limit == i)
        return factors;
    }
    if (N > 1)
      ++factors[N];
    return std::move(factors);
  };
  // Compute prime factors for the denominator
  auto primeFactors = computePrimeFactors(B);

  // If there are no 10-coprimes factors there's no cycle
  function<ull(ull, ull)> gcd = [&](ull A, ull B) {
    ull remainder = A % B;
    if (remainder == 0)
      return B;
    else
      return gcd(B, remainder);
  };
  bool isCyclePresent = false;
  set<ull> coprimes;
  for (auto& v : primeFactors) {
    if (gcd(v.first, 10ull) == 1) {
      isCyclePresent = true;
      coprimes.insert(v.first);
    }
  }

  if (isCyclePresent == false) {
    // Just perform the normal division
    double result = static_cast<double>(A) / static_cast<double>(B);
    ss << result;
    return ss.str();
  }

  // There's a cycle, calculate its length
  auto discreteLog = [](ull coprime, ull limit = -1) {
    ull i = 1;
    while ((ebs(10ull, i) % coprime) != 1ull 
      && (limit != -1 ? (i < limit - 1) : true)) ++i;
    return i;
  };
  ull cycleLength = 0;
  for (auto cp : coprimes) {
    ull k = discreteLog(cp, B);    
    if (k > cycleLength)
      cycleLength = k;
  }

  ss << A / B;

  ull remainder = A % B;
  if (remainder == 0)
    return ss.str();

  ss << ",";

  if (cycleLength == 0) {
    while (remainder != 0) {
      ss << remainder;
      remainder = (remainder * 10) % B;
    }
    return ss.str();
  }

  // Calculate the number of digits before the period
  auto factors25Numerator = computePrimeFactors(A, 5ull);
  ull digitsBeforePeriod = 0;
  auto it = primeFactors.find(2ull);
  if (it != primeFactors.end()) { // 2
    auto itN = factors25Numerator.find(2);
    if(itN == factors25Numerator.end())
      digitsBeforePeriod = max<long long>(digitsBeforePeriod, 
                            static_cast<long long>(it->second));
    else
      digitsBeforePeriod = max<long long>(digitsBeforePeriod, 
                            static_cast<long long>(it->second - itN->second));
  }
  it = primeFactors.find(5);
  if (it != primeFactors.end()) { // 5
    auto itN = factors25Numerator.find(5);
    if (itN == factors25Numerator.end())
      digitsBeforePeriod = max<long long>(digitsBeforePeriod, 
                            static_cast<long long>(it->second));
    else
      digitsBeforePeriod = max<long long>(digitsBeforePeriod, 
                            static_cast<long long>(it->second - itN->second));
  }

  for (ull i = 0; i < digitsBeforePeriod; ++i) {
    ss << (remainder * 10) / B;
    remainder = (remainder * 10) % B;
  }
  ss << "(";
  for (ull i = 0; i < cycleLength; ++i) {
    ss << (remainder * 10) / B;
    remainder = (remainder * 10) % B;
  }
  ss << ")";

  return ss.str();
}
{% endhighlight %}

It has to be noted that the formatting problem isn't particularly suited to be computed with this latter period-estimate method since discrete logarithm is a [notoriously hard problem](http://cs.stackexchange.com/q/2658) so the second approach would easily be outperformed by the first one given huge numbers. Anyway the code above might be useful to practically show how to approach problems requiring calculating the size of a rational number (anti)period.

References
==========
* [Length of digits before the period in decimal expansion for rational numbers](http://math.stackexchange.com/q/1449660)
* [Decimal expansion](http://mathworld.wolfram.com/DecimalExpansion.html) - Wolfram MathWorld