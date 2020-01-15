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
