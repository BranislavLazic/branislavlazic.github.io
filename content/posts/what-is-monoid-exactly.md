---
date: '2025-03-04T16:11:10+01:00'
title: 'What is a Monoid, Exactly?'
---

Let's talk about some math concepts in this context. Remember those lessons about _identity_, _commutativity_, _associativity_, and _distributivity_? These laws are quite important in the context of monoids. A monoid comes from mathematics. In abstract algebra, a monoid is a set equipped with an associative binary operation and an identity element.

## Semigroup

In mathematics, a semigroup is a structure with a binary operation that satisfies the law of associativity.

An example of an associative operation would be addition, multiplication, or even string concatenation:

_(a + b) + c = a + (b + c)_

In programming, we can represent a semigroup with an interface that combines two values of the same type:

```scala
trait Semigroup[A]:
  def combine(x: A, y: A): A
```

## Monoid

A monoid is a semigroup with an identity element. The identity element is a value that, when combined with any other value, returns the other value unchanged.

For example, in addition, the identity element is 0:

_a + 0 = a_

In string concatenation, the identity element is an empty string:

_"abc" + "" = "abc"_

In programming, we can extend the `Semigroup` trait to create a `Monoid` trait:

```scala
trait Monoid[A] extends Semigroup[A]:
  def empty: A
```

Here, `empty` represents the identity element, which is a value that does not change other elements when used in the binary operation.

## Monoid Applied

Why is this even important? It turns out that monoids play pretty well with lists.

For example, let's try to sum up all numbers within a list. Obviously, we can simply call the `sum` function, but we can also do it with `foldLeft` or `foldRight` methods.

```scala
List(1, 2, 3, 4).foldLeft(0)(_ + _)
```

Looks familiar? Exactly. `foldLeft` takes the monoid's identity as the first argument, and the `combine` function as the second argument.

With Scala 3, we could write the sum function as:

```scala
def sum[A: Monoid as m](list: List[A]) =
  list.foldLeft(m.empty)(m.combine)
```

In order to sum the list of integers, we need to provide "a proof". An instance of `Monoid[Int]`. For that purpose, we could also use the new Scala 3 syntax:

```scala
given Monoid[Int] with
  def empty = 0
  def combine(x: Int, y: Int) = x + y

sum(List(1, 2, 3, 4))
```

By understanding and utilizing monoids, we can write more generic and reusable code, especially when dealing with collections and aggregation operations.

## Bonus

But that's not even all. Let's take for instance the `Tree` data structure. A `Tree` is a collection where values are organized hierarchically as nodes, where each node must have a single _parent_ node but can have multiple _children_ nodes. As such, a tree is a perfect collection for parallelizing associative operations.

For example, if we want to sum all integers within a Tree, it doesn't matter which _leaves_ will be summed first, since the operation is associative. Rewritten in the context of parallel collections:

```scala
def sumPar[A: Monoid](parColl: ParCollection[A]) =
  parColl.foldLeft(summon[Monoid[A]].empty)(summon[Monoid[A]].combine)
```

Our function remains pretty much the same, but the monoid gets a new, higher meaning - it's a warranty or proof for parallelization.
