# Chapter 10 - Collections - III

## 10.15 Flattening a List of Lists with flatten
The `flatten` method isn't limited to a `List`; it works with other sequences (`Array, ArrayBuffer, Vector`, etc) as well.

You can chain methods after calling flatten too:
```
scala> val people = couples.flatten.map(_.capitalize).sorted
people: List[String] = List(Al, Julia, Kim, Terry)
```

## 10.16 Combining map and flatten with flatMap
You can use `flatMap` in situations where you run `map` followed by `flatten`.
- You're using `map` (or a `for/yield` expression) to create a new collection from an existing collection
- The resulting collection is a list of lists
- You call `flatten` immediately after `map` (or a `for/yield` expression)

This next example shows how to use `flatMap` with an `Option`. Here, you are supposed to calculate the sum of the numbers in a list, but the numbers are all strings and some of them won't convert properly to integers.

```
val bag = List("1", "2", "three", "4", "one hundred seventy five")
```
You begin by creating a "String to integer" conversion method that returns either `Some[Int]` or `None`:
```
def toInt(in: String): Option[Int] = {
  try {
    Some(Integer.parseInt(in.trim))
  } catch {
    case e: Exception => None
  }
}
```
Now, you can do this:
```
scala> bag.flatMap(toInt).sum
res0: Int = 7
```

Let's break the problem down into smaller steps. First, here's what happens when you use `map` on the initial collection of strings:
```
scala> bag.map(toInt)
res0: List[Option[Int]] = List(Some(1), Some(2), None, Some(4), None)
```
The `map` method applies the `toInt` function to each element in the collection, and returns a list of `Some[Int]` and `None` values. But the sum method needs a `List[Int]`; how do you get there from here?

As shown in the previous recipe, `flatten` works very well with a list of `Some` and `None` elements. It extracts the values from the `Some` elements while discarding the `None` elements:
```
scala> bag.map(toInt).flatten
res1: List[Int] = List(1, 2, 4)
```
This makes finding the sum easy.

As you can imagine, once you get the original list down to a `List[Int]`, you can call any of the powerful collections methods to get what you want:
```
scala> bag.flatMap(toInt).filter(_ > 1)
res4: List[Int] = List(2, 4)

scala> bag.flatMap(toInt).takeWhile(_ < 4)
res5: List[Int] = List(1, 2)

scala> bag.flatMap(toInt).partition(_ > 3)
res6: (List[Int], List[Int]) = (List(4),List(1, 2))
```

## 10.17 Using filter to Filter a Collection
`filter` is one of the methods that can be used to filter the elements of an input collection to produce an output collection.

To use `filter` on your collection, give it a predicate to filter the collection of elements as desired. Your predicate should accept a parameter of the same type that the collection holds, evaluate that element, and return `true` to keep the element in the new collection, or `false` to filter it out.

Unique characteristics of `filter`:
- `filter` walks through all of the elements in the collection; some of the other methods stop before reaching the end of the collection
- `filter` lets you supply a predicate to filter the elements

You can also put your algorithm in a separate method or function and then pass it into `filter` as a predicate.

The two keys to using `filter` are:
- Your algorithm should return true for the elements you want to keep and false for the other elements
- Remember to assign the results of the filter method to a new variable; filter doesn’t modify the collection it’s invoked on

## 10.18 Extracting a Sequence of Elements from a Collection
Let's say you want to extract a sequence of contiguous elements from a collection, either by specifying a starting position and length, or a function.

There are a few collection methods you can use to extract a contiguous list of elements from a sequence, including: `drop, dropWhile, head, headOption, init, last, lastOption, slice, tail, take, takeWhile`.

- `drop` method drops the number of elements you specify from the beginning of the sequence
- `dropWhile` method drops elements as long as the predicate you supply returns `true`
- `dropRight` method works like `drop`, but starts from the end of the sequence
- `take` extracts the first N elements from a sequence
- `takeWhile` returns elements as long as the predicate you supply returns `true`
- `takeRight` works like `take`, but starts from the end of the sequence
- `slice(from, until)` returns a sequence beginning at the index `from` until the index `until`, not including `until`.

## 10.19 Splitting Sequences into Subsets (groupBy, partition, etc)
Use `groupBy, partition, span`, or `splitAt` methods to partition a sequence into subsequences. The `sliding` and `unzip` methods can also be used to split sequences into subsequences, though `sliding` can generate many subsequences, and `unzip` primarily works on a sequence of `Tuple2` elements.

The `groupBy, partition,` and `span` methods let you split a sequence into subsets according to a function, whereas `splitAt` lets you split a collection into two sequences by providing an index number.

The `groupBy` method partitions the collection into a Map of subcollections based on your function - a `true` map containing the elements for which your predicate returned true, and a `false` map.

The `partition`, `span`, and `splitAt` methods create a `Tuple2` of sequences that are of the same type as the original collection. The partition method creates two lists, one containing values for which your predicate returned `true`, and the other containing the elements that returned `False`.

The `span` method returns a `Tuple2` based on your predicate `p`, consisting of “the longest prefix of this list whose elements all satisfy `p`, and the rest of this list.” The `splitAt` method splits the original list according to the element index value you supplied.

When a `Tuple2` of sequences is returned, its two sequences can be accessed like this:
```
scala> val (a,b) = x.partition(_ > 10)
a: List[Int] = List(15, 20, 12)
b: List[Int] = List(10, 5, 8)
```

## 10.20 Walking Through a Collection with the reduce and fold Methods
Let's say you want to walk through all of the elements in a sequence, comparing two neighboring elements as you walk through the collection.

You can use `reduceLeft, foldLeft, reduceRight`, and `foldRight` methods to walk through elements in a sequence, applying your function to neighboring elements to yield a new result, which is then compared to the next element in the sequence to yield a new result. For example:

```
scala> val a = Array(12, 6, 15, 2, 20, 9)
scala> a.reduceLeft(_ max _)

--> compare 12 to 6, 12 is larger (12, 15, 2, 20, 9)
--> compare 12 to 15, 15 is larger (15, 2, 20, 9)
... (15, 20, 9)
... (20, 9)
... (20)
```

You can write your own function and provide it into `reduceLeft`. One subtle but important note about reduceLeft: the function (or method) you supply must return the same data type that’s stored in the collection. This is necessary so `reduceLeft` can compare the result of your function to the next element in the collection.

`foldLeft` works just like `reduceLeft`, but it lets you set a seed value to be used for the first element.

`scanLeft` (and `scanRight`) walks through a sequence in a manner similar to `reduceLeft` and `reduceRight`, but they return a sequence instead of a single value.

`scanLeft` produces a collection containing cumulative results of applying the operator going left to right.
For example:
```
val product = (x: Int, y: Int) => {
  val result = x * y
  println(s"multiplied $x by $y to yield $result")
  result
}

scala> val a = Array(1, 2, 3)
a: Array[Int] = Array(1, 2, 3)

scala> a.scanLeft(10)(product)
multiplied 10 by 1 to yield 10
multiplied 10 by 2 to yield 20
multiplied 20 by 3 to yield 60
res0: Array[Int] = Array(10, 10, 20, 60)
```

## 10.21 Extracting Unique Elements from a Sequence
You can use `distinct` method on the collection:
```
scala> val x = Vector(1, 1, 2, 3, 3, 4)
x: scala.collection.immutable.Vector[Int] = Vector(1, 1, 2, 3, 3, 4)

scala> val y = x.distinct
y: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3, 4)
```
The `distinct` method returns a new collection with the duplicate values removed. You have to assign the result to a new variable - this is required for both immutable and mutable collections.

You can also convert a collection to a `Set` by calling the `toSet` method.

### Using distinct with your own classes
To use `distinct` with your own class, you need to implement the `equals` and `hashCode` methods.

## 10.22 Merging Sequential Collections
There are a few options to merging two sequences into one sequence:
- keeping all of the original elements,
- finding the elements that are common to both collections
- finding the difference between the two sequences

Depending on your needs, in a nutshell:
- The `++=` method merge a sequence into a mutable sequence
- The `++` method merge two mutable or immutable sequences.
- Use collection methods like `union`, `diff`, and `intersect`

The `++=` method merges a sequence into a mutable collection like an `ArrayBuffer`:
```
scala> val a = collection.mutable.ArrayBuffer(1,2,3)
a: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 2, 3)

scala> a ++= Seq(4,5,6)
res0: a.type = ArrayBuffer(1, 2, 3, 4, 5, 6)
```

The `++` method merges two mutable/immutable collections while assigning the result to a new variable:
```
scala> val a = Array(1,2,3)
a: Array[Int] = Array(1, 2, 3)

scala> val b = Array(4,5,6)
b: Array[Int] = Array(4, 5, 6)

scala> val c = a ++ b
c: Array[Int] = Array(1, 2, 3, 4, 5, 6)
```

Methods like `union` and `intersect` combine sequences to create a resulting sequence. The `diff` method results depend on which sequence it's called on (`a diff b` is different from `b diff a`).

## 10.23 Merging Two Sequential Collections into Pairs with zip
This is useful when you want to merge data from two sequential collections into a collection of key/value pairs.

The `zip` method joins two sequences into one like this:
```
scala> val women = List("Wilma", "Betty")
women: List[String] = List(Wilma, Betty)

scala> val men = List("Fred", "Barney")
men: List[String] = List(Fred, Barney)

scala> val couples = women zip men
couples: List[(String, String)] = List((Wilma,Fred), (Betty,Barney))
```
This creates an Array of Tuple2 elements, which is a merger of the two original sequences.

If one collection contains more items than the other collection, the items at the end of the longer collection will be dropped. You can use the `unzip` method to reverse `zip`

## 10.24 Creating a Lazy View on a Collection
You're working with a large collection a want to create a "lazy" version of it so it will only compute and return results as they are needed.

Except for the `Stream` class, whenever you create an instance of a Scala collection class, you're creating a *strict* version of the collection. You can optionally create a *view* on a collection to make it non-strict (a.k.a. lazy)

For instance:
```
scala> 1 to 100
res0: scala.collection.immutable.Range.Inclusive =
  Range(1, 2, 3, 4, ... 98, 99, 100)

scala> (1 to 100).view
res0: java.lang.Object with
  scala.collection.SeqView[Int,scala.collection.immutable.IndexedSeq[Int]] =
  SeqView(...)
```
Creating the `Range` with the view shows something called a `SeqView` instead of a `Range`.

The signature of the `SeqView` shows:
- `Int` is the type of the view's elements
- `scala.collection.immutable.IndexedSeq[Int]` portion of the output indicates the type you'll get if you `force` the collection back to a strict collection.
```
scala> val x = view.force
x: scala.collection.immutable.IndexedSeq[Int] =
  Vector(1, 2, 3, ... 98, 99, 100)
```

### Use cases
There are two primary use cases for using a view:
- Performance
- To treat a collection like a database view

The following examples show how a collection view works like a database view:
```
// create a normal array
scala> val arr = (1 to 10).toArray
arr: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// create a view on the array
scala> val view = arr.view.slice(2, 5)
view: scala.collection.mutable.IndexedSeqView[Int,Array[Int]] = SeqViewS(...)

// modify the array
scala> arr(2) = 42

// the view is affected:
scala> view.foreach(println)
42
4
5

// change the elements in the view
scala> view(0) = 10
scala> view(1) = 20
scala> view(2) = 30

// the array is affected:
scala> arr
res0: Array[Int] = Array(1, 2, 10, 20, 30, 6, 7, 8, 9, 10)
```

## 10.25 Populating a Collection with a Range
You can call the `range` method on sequences that support it, or create a `Range` and convert it to the sequence.

In the first approach, the `range` method is available on the companion object of supported types like `Array`, `List`, `Vector`, `ArrayBuffer`, and others:
```
scala> Array.range(1, 5)
res0: Array[Int] = Array(1, 2, 3, 4)

scala> List.range(0, 10)
res1: List[Int] = List(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> Vector.range(0, 10, 2)
res2: collection.immutable.Vector[Int] = Vector(0, 2, 4, 6, 8)
```
For the second approach:
```
scala> val a = (0 until 10).toArray
a: Array[Int] = Array(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val list = 1 to 10 by 2 toList
list: List[Int] = List(1, 3, 5, 7, 9)

scala> val list = (1 to 10).by(2).toList
list: List[Int] = List(1, 3, 5, 7, 9)
```
Using this approach is useful for some collections, like Set, which don’t offer a range method.

## 10.26 Creating and Using Enumerators
You can `extend` the `scala.Enumeration` class to create your enumeration:
```
package com.acme.app {
  object Margin extends Enumeration {
    type Margin = Value
    val TOP, BOTTOM, LEFT, RIGHT = Value
  }
}
```
Then, import it to your application
```
object Main extends App {
  import com.acme.app.Margin._

  // use an enumeration value in a test
  var currentMargin = TOP

  // later in the code ...
  if (currentMargin == TOP) println("working on Top")

  // print all the enumeration values
  import com.acme.app.Margin
  Margin.values foreach println
}
```
Enumerations are useful tool for creating groups of constants, such as days of the week, weeks of the year, and many other situations where you have a group of related, constant values.

## 10.27 Tuples, for when you just need a bag of things
Simply put, you just want to create a small collection of heterogeneous elements. For this, you can use a tuple.

A tuple gives you a way to store a group of heterogeneous items in a container. Simply create a tuple by enclosing the desired elements between parentheses. For example, this is a two element tuple:
```
scala> val d = ("Debi", 95)
d: (String, Int) = (Debi,95)
```
You can access tuple elements using an underscore construct:
```
scala> t._1
res1: java.lang.String = Debi

scala> t._2
res2: Int = 95
```
A common use case for a tuple is returning multiple items from a method.

Though a tuple isn't a collection, you can treat it as one when needed by creating an iterator:
```
scala> val x = ("AL" -> "Alabama")
x: (java.lang.String, java.lang.String) = (AL,Alabama)

scala> val it = x.productIterator
it: Iterator[Any] = non-empty iterator

scala> for (e <- it) println(e)
AL
Alabama
```
Like any other iterator, after it's used once, it will be exhausted.

## 10.28 Sorting a Collection
Let's say you want to sort a sequential collection, or you want to implement the `Ordered` trait in a custom class so you can use the `sorted` method, or operators like <, <=, >, and >= to compare instances of your class.

You can use `sorted` or `sortWith` methods to sort a collection.

The `sorted` method can sort collections with type `Double`, `Float`, `Int`, and any other type that has an implicit `scala.math.Ordering`

The “rich” versions of the numeric classes (like `RichInt`) and the `StringOps` class all extend the `Ordered` trait, so they can be used with the sorted method.

The `sortWith` method lets you provide your own sorting function. For example:
```
scala> List(10, 5, 8, 1, 7).sortWith(_ < _)
res1: List[Int] = List(1, 5, 7, 8, 10)

scala> List(10, 5, 8, 1, 7).sortWith(_ > _)
res2: List[Int] = List(10, 8, 7, 5, 1)

scala> List("banana", "pear", "apple", "orange").sortWith(_.length < _.length)
res5: List[java.lang.String] = List(pear, apple, banana, orange)

scala> List("banana", "pear", "apple", "orange").sortWith(_.length > _.length)
res6: List[java.lang.String] = List(banana, orange, apple, pear)
```
If your sorting method gets long, first declare it as a method, and then use it as the parameter for `sortWith`.

If the type a sequence is holding doesn’t have an implicit `Ordering`, you won’t be able to sort it with `sorted`. You can write a simple anonymous function to sort.

IF you'd rather use the `sorted` method, you can mix the `Ordered` trait into the class, and implement a `compare` method.

## 10.29 Converting a Collection to a String with mkString
You want to convert elements of a collection to a `String`, possible adding field separator, prefix, and suffix.

You can use the `mkString` method to print a collection as a `String`. Given:
```
val a = Array("apple, "banana", "cherry")

scala> a.mkString
res1: String = applebananacherry
```

You can add a separator:
```
scala> a.mkString(", ")
res3: String = apple, banana, cherry
```
If you happen to have a list of lists that you want to convert to a `String`, such as the
following array of arrays, first flatten the collection, and then call `mkString`:
```
scala> val a = Array(Array("a", "b"), Array("c", "d"))
a: Array[Array[java.lang.String]] = Array(Array(a, b), Array(c, d))

scala> a.flatten.mkString(", ")
res5: String = a, b, c, d
```
