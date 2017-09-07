---
layout: post
title:  "Building intuition on Value Categories"
tags: c++
---

*This post aims to be an intuitive (rather than a thorough) explanation of value categories. It assumes a certain degree of prior knowledge regarding [move semantics](https://stackoverflow.com/q/3106110/1938163) and the C++ language. You're encouraged to report errors / unclear parts / suggestions on the [issue tracker](https://github.com/marcodiiga/marcodiiga.github.io/issues).*

## Expressions

An expression in C++ is defined as

> *n4687 - §8/p1*
>
> a sequence of operators and their operands, that specifies a computation. An expression can result in a value and can cause side effects

Some examples:

```cpp
x = 42; // Assignment expression

42; // An expression as well, with no side effects

int a; // A declaration statement (not an expression)

int b = fun(42); // A declaration statement with an expression initializer
                 // fun(42) is an expression
```

A *side-effect* is defined as a change of the execution environment. If an expression modifies some state outside of its scope or has an *observable* interaction with the outside world besides returning a value, it has side effects.

```cpp
a = ++x; // ++x is an expression and has side effects (it doesn't just return a value)
```

An expression is characterized by two important properties:

* its type
* its *value category*

Understanding the value category of an expression is important because instructs the programmer on what kind of operations are allowed on it and possibly on the lifetime expectancy of the entities involved as well.

## Value categories

The three primary value categories of an expression are

* *lvalue*
* *xvalue*
* *prvalue*

these can in turn be grouped into *glvalues* and *rvalues*

<p align="center">
<img src="/images/posts/valuecategories1.png"/>
</p>

There were historically only two value categories: *lvalues* and *rvalues*, and they were simplistically referred to as the left-hand and right-hand of a variable assignment. This definition is no longer generally true.

*lvalue* expressions represent storage location values (they can often appear on the left-hand side of an assignment) while *rvalues* are usually associated with temporary or disposable objects.

The importance of distinguishing between a variable's type and the value categories of the expressions where it is used can't be stressed enough:

```cpp
int&& x = 42;
```

The variable `x` has type *reference to an rvalue*  but that doesn't mean the **expression** `x` is an `rvalue`:

```cpp
void foo(int&& x) {
    int *v = &x; // x is an lvalue here
    ...
}
```

The distinction between the variable type and its value category in expressions is important: `x` is an *lvalue* expression in the line where its address is taken.

Also notice that *lvalue* references and *rvalue* references are both reference types but they're different with respect to **what binds** to them. An *lvalue* expression can bind to *lvalue* references

```cpp
int a = 42;
int& lvr = a; // lvalue expression binding to lvalue reference
```

while *rvalue* expressions can't bind to them

```cpp
// int& lvr = 42; // Error
int&& rvr = 42;
```

The second line works just fine because the literal `42` is an *rvalue* expression and binding it to a reference [prolongs its lifetime](http://en.cppreference.com/w/cpp/language/reference_initialization#Lifetime_of_a_temporary) (*n4687 - §15.2/p6*). The language also allows temporaries to bind to `const` *lvalue* references

```cpp
const int& v = 42;
```

to prevent accidental modifications to temporaries (that will be destroyed anyway). *rvalue* references can't be bound to *lvalue* expressions though

```cpp
int a = 42;
// int&& rvr = a; // Not allowed
```

Takeaway is:

* there are three main value categories: *lvalue*, *xvalue*, *prvalue*
* `variable_type != value_category`, the latter is a property of expressions
* binding temporaries or disposables to references prolongs their lifetime

## Identity vs movability

Two particular traits that render value categories more understandable and might aid in explaining the need for the three aforementioned main categories are *having an identity* and *being movable* (cfr. *n4687 - [basic.lval]*, notice that we'll be simplifying "having an identity" as "having a name/being bound to a reference" for explanation's sake)

<p align="center">
<img src="/images/posts/valuecategories2.png"/>
</p>

This allows us to dive more into the definitions given in the standard:

* *lvalues* are *glvalues* that are not *xvalues*: they're expressions associated with storage areas whose address can be requested, they **have a name or are bound to a reference** and are **not** marked as movable from

  ```cpp
  int x = 42;
  int *addr_of_x = &x; // I can request x's address
  *addr_of_x; // lvalue expression referring to whatever the variable points to
  // int&& rvr = x; // Wrong: the expression x doesn't designate an rvalue
  ```

  A call to a function which returns an *lvalue* reference is an *lvalue* as well

  ```cpp
  int x;
  int& foo() { return x; } // decltype(foo()) = int&
  // I can take the address of a reference as well (it'll be the one of its referent)
  ```
  *lvalue* references are always *lvalue* expressions
  ```cpp
  int x = 42;

  int& ref = x;
  int& foo() {return x;}

  int expr1 = ref; // ref is an lvalue
  int expr2 = foo(); // foo() is also an lvalue
  ```
  The main point here is that *lvalues* used in an expression are not marked as *expiring* (their resources are not marked for moving) and they also have an identity (they have a name or are bound to a reference, so their lifetime doesn't need immediate attention in the expression to be prolonged).

  A *lvalue* reference is bound to an entity not marked as *expiring* while a *rvalue* reference is bound to an entity marked as *expiring*.

* *xvalues* are eXpiring values: *glvalues* (they do have an identity) whose resources are also marked as *movable*.

    A call to a function which returns an *rvalue reference* is an *xvalue*

  ```cpp
  int x;
  int&& foo() {
    return std::move(x); // Mark x as 'movable'
  } // decltype(foo()) = int&&

  class A {
  public:
    A(int&& v) { /* Cannibalize v */ }
  };

  A obj{foo()};
  ```
  Invoking `foo()` will yield an *xvalue*, i.e. an expression which has an identity (the function is returning a reference type bound to an entity marked as *expiring*, still a reference though - doesn't need immediate lifetime prolonging) and has been marked as movable (in the example above `obj`'s move constructor is called to potentially cannibalize `v`'s resources).
  Using `std::move` produces an *xvalue* as well since it marks an object as suitable for moving from and casts it to an *rvalue* reference.


* *prvalues* are *pure rvalues*: *rvalues* which are not *xvalues*. They're marked as *can be moved from* (temporaries are constructed if needed), but don't have an identity of their own: if one intends to retain/save them, they will need to bind them to a reference. Unnamed temporaries (`X()`) and literals (`42`) are *prvalues*.
  The result of calling a function whose return type is **not a reference** is a *prvalue*: something that can bind to *rvalue* references (or `const` lvalue references) but that does not refer to an identity-owning entity.

*prvalues* are expressions whose evaluations are temporary entities marked as *movable* and immediately expiring unless a life-prolonging reference binding happens. *xvalues* yield entities marked as *movable* but that do not require immediate binding to a reference to see their life prolonged (they will be destroyed but this is not going to happen automatically at the end of the full expression). *lvalues* are the most *"stable"* state.

The remaining two broader categories lie at the intersection of the ones outlined above

* *glvalues* are generalized *lvalues*: they're either *lvalues* or *xvalues*. The standard adds
  > A glvalue is an expression whose evaluation determines the identity of an object

* *rvalues* are either *xvalues* or *prvalues*

The distinction is often subtle but key to instruct programs to better manipulate objects/memory regions and understanding move semantics (and lengthy compiler errors as well).

## References

* [C++ Standard n4687](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4687.pdf)

* [What are rvalues, lvalues, xvalues, glvalues, and prvalues?](https://stackoverflow.com/questions/3601602/what-are-rvalues-lvalues-xvalues-glvalues-and-prvalues)

* [What expressions yield a reference type when decltype is applied to them?](https://stackoverflow.com/questions/17241614/what-expressions-yield-a-reference-type-when-decltype-is-applied-to-them)
