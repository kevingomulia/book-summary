# Chapter 10 - Collections - I

A goal of this chapter is to help you through this plethora of options to find the solutions you need. Recipes will help you decide which collections to use in different situations, and also choose a method to solve a problem.

The methods that are common to all collections are shown in this chapter, and methods specific to collections like `List, Array, Map`, and `Set` are shown in Chapter 11.

There are a few important concepts to know when working with the methods of the Scala collection classes:
- What a predicate is
- What an anonymous function is (Recipe 9.1)
- Implied loops

A *predicate* is a method, function, or anonymous function that takes one or more parameters and **returns a Boolean value**. For instance, this is a predicate:
```
def isEven(i: Int) = if (i % 2 == 0) true else false
```

Anonymous function was described in depth in Recipe 9.1, but here is an example of the long form for an anonymous function:
```
(i: Int) => i % 2 == 0
```
Here's the short form:
```
_ % 2 == 0
```
You can combine it with the `filter` method on a collection, and it makes for a lot of power in just a bit of code:
```
scala> val list = List.range(1, 10)
list: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val events = list.filter(_ % 2 == 0)
events: List[Int] = List(2, 4, 6, 8)
```
Lastly, *implied loops*. As you can see from the example above, the `filter` method contains a loop that applies your function to every element in the collection and returns a new collection.

Collection methods like `filter, foreach, map, reduceLeft`, and many more have loops built into their algorithms.

## 10.1 Understanding the Collections Hierarchy
Since Scala classes can inherit from traits, and well-designed traits are granular, a class hierarchy can look very complex.

At a high level, Scala's collection classes begin with the `Traversable` and `Iterable` traits, and extend into the three main categories of sequences (`Seq`), sets (`Set`) and maps (`Map`). Sequences further branch off into *indexed* and *linear* sequences.

The `Traversable` trait lets you traverse an entire collection, and its Scaladoc states that it "implements the behaviour common to all collections in terms of a `foreach` method", which lets you traverse the collection repeatedly.

The `Iterable` trait defines an *iterator*, which lets you loop through a collection’s elements one at a time, but when using an iterator, the collection can be traversed only once, because each element is consumed during the iteration process.

### Sequences
Scala contains a large number of sequences, which will be described in detail in Recipe 10.2.

As previously mentioned, sequences branch off into *indexed sequences* and *liner sequences* (linked lists). An `IndexedSeq` indicates that **random access of elements is efficient**, such ass accessing an `Array` element as `arr(500)`.

A `LinearSeq` implies that the collection can be efficiently split into head and tail components, and it's common to work with them using the `head`, `tail`, and `isEmpty` methods. Creating a `LinearSeq` creates a `List`, which is a singly linked list.

### Maps
Like in Java, a Scala `Map` is a collection of key/value pairs, where all the **keys must be unique**. The most common map classes are: `HashMap, WeakHashMap, SortedMap, TreeMap, LinkedHashMap, ListMap`.

Map traits and classes are discussed later on. When you just need a simple, *immutable* map, you can create one without requiring an import.
```
scala> val m = Map(1 -> "a", 2 -> "b")
m: scala.collection.immutable.Map[Int, java.lang.String] = Map(1 -> a, 2 -> b)
```
For *mutable* map, you have to either import it or specify its full path to use it:
```
scala> val m = collection.mutable.Map(1 -> "a", 2 -> "b")
m: scala.collection.mutable.Map[Int, String] = Map (2 -> b, 1-> a)
```

### Sets
Like a Java Set, a Scala `Set` is a collection of **unique elements**. The common set classes are `BitSet, HashSet, ListSet, SortedSet`.

Set traits and classes are discussed later on. If you need an immutable set, you can create it without an import statement:
```
scala> val set = Set(1, 2, 3)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
```
Similar to a map, if you want to use a mutable set, you must import it or specify its complete path.
```
scala> val s = collection.mutable.Set(1, 2, 3)
s: scala.collection.mutable.Set[Int] = Set(1, 2, 3)
```

### More collection classes
There are many additional collection traits and classes, including `Stream, Queue, Stack,` and `Range`. You can also create *views* on collections; use iterators; and work with the `Option`, `Some`, and `None` types as collections.

## 10.2 Choosing a Collection Class
There are three main categories of collection classes to choose from:
- Sequence
- Map
- Set
A sequence is a linear collection of elements and may be indexed or linear (a linked list). A map contains a collection of key/value pairs, like a Java Map, Ruby Hash, or Python dictionary. A set is a collection that contains no duplicate elements.

There are also other useful collection types, including Stack, Queue, and Range. There are a few other classes that act like collections, including tuples, enumerations, and the Option/Some/None and Try/Success/Failure classes.

### Choosing a sequence
You have two main decisions when choosing a sequence:
- Should the sequence be indexed (like an array), allowing rapid access to any elements, or should it be implemented as a linked list?
- Do you want a mutable or immutable collection?

As of Scala 2.10, the recommended, general-purpose, "go to" sequential collections for the combinations are:

| | Immutable | Mutable |
|---|---|---|
| Indexed | Vector | ArrayBuffer |
| Linear (Linked lists) | List | ListBuffer |

There are many more sequence alternative; the most common immutable sequence choices are:

| | IndexedSeq | LinearSeq | Description |
|---|---|---|---|
|List| | ✓ | A singly linked list. Suited for recursive algorithms that work by splitting the head from the remainder of the list.|
|Queue| | ✓ | A FIFO data structure. |
|Range|✓| | A range of integer values. |
|Stack| |✓| A FILO data structure. |
|Stream| |✓| Similar to `List`, but it's lazy and persistent. Good for a large or infinite sequence.|
|String|✓| | Can be treated as an immutable, indexed sequence of characters.|
|Vector|✓| | To "go to" immutable, indexed sequence. "Implemented as a set of nested arrays that's efficient at splitting and joining"|

The most common mutable sequence choices are:

| | IndexedSeq | LinearSeq | Description |
|---|---|---|---|
|Array|✓| | Backed by a Java array, its elements are mutable, but it can't change in size.|
|ArrayBuffer|✓| |"Go to" class for a mutable, sequence collection. The amortized cost for appending elements is constant.|
|ArrayStack|✓| |A FILO data structure. Better than `Stack` when performance is important|
|DoubleLinkedList| |✓| Like a singly linked list, but with a `prev` method as well. The additional links make element removal very fast.|
|LinkedList| |✓| A mutable, singly linked list.|
|ListBuffer| |✓| Like an ArrayBuffer, but backed by a list. If you plan to convert the buffer to a list, use ListBuffer instead of ArrayBuffer. Constant time prepend and append, and most other operations are linear. |
|MutableList| |✓| A mutable, singly linked list with constant-time append.|
|Queue| |✓| A FILO data structure.
|Stack| |✓| A FILO data structure. (ArrayStack is slightly more efficient.)
|StringBuilder|✓| | Used to build strings, as in a loop. Similar to Java's StringBuilder.|

Traits commonly used in library APIs:

| Trait | Description |
| --- | --- |
|IndexedSeq | Implies that random access of elements is efficient |
|LinearSeq | Implies that linear access to elements is efficient |
|Seq | Used when it isn't important to indicate whether the sequence is indexed or linear in nature |

### Choosing a map
| | Immutable | Mutable | Description |
|---|---|---|---|
|HashMap|✓|✓|The immutable version implements maps using a hash trie. The mutable version implements maps using a hashtable|
|LinkedHashMap| |✓|Implements mutable maps using a hashtable. Returns elements by the order in which they were inserted.|
|ListMap|✓|✓|Implemented using a list. Returns elements in the opposite order by which they were inserted.|
|Map|✓|✓|Base map, with both mutable and immutable implementations|
|SortedMap|✓| |A base trait that stored its keys in sorted order (Creating a variable as a SortedMap currently returns a TreeMap)|
|TreeMap|✓| |An immutable, sorted map, implemented as a red-black tree|
|WeakHashMap| |✓| A hash map with weak references|

You can create a thread-safe mutable map by mixing the `SynchronizedMap` trait into the map implementation you want.

### Choosing a set
There are base mutable and immutable set classes, a `SortedSet` to return elements in sorted order by key, a `LinkedHashSet` to store elements in insertion order, and a few other sets for special purposes.

| | Immutable | Mutable | Description |
|---|---|---|---|
|BitSet|✓|✓|Used to save memory when you have a set of integers|
|HashSet|✓|✓|The immutable version implements sets using a hash trie. The mutable version implements sets using a hashtable.|
|LinkedHashSet| |✓|A mutable set implemented using a hashtable. Returns elements in the order in which they were inserted.|
|ListSet|✓| |A set implemented using a list structure|
|TreeSet|✓|✓|The immutable version implements immutable sets using a tree. The mutable version is a mutable `SortedSet` with an immutable AVL tree as underlying data structure.|
|Set|✓|✓|Generic base traits, with both mutable and immutable implementations|
|SortedSet|✓|✓|A base trait. Creating a variable as a SortedSet returns a TreeSet|

You can create a thread-safe mutable map by mixing the `SynchronizedSet` trait into the map implementation you want.

### Types that act like collections

| | Description |
|---|---|
|Enumeration |A finite collection of constant values (i.e., the days in a week or months in a year).|
|Iterator |An iterator isn’t a collection; instead, it gives you a way to access the elements in a collection. It does, however, define many of the methods you’ll see in a normal collection class, including foreach, map, flatMap, etc. You can also convert an iterator to a collection when needed.|
|Option| Acts as a collection that contains zero or one elements. The Some class and None object extend Option. Some is a container for one element, and None holds zero elements.|
|Tuple| Supports a heterogeneous collection of elements. There is no one “Tuple” class; tuples are implemented as case classes ranging from Tuple1 to Tuple22, which support 1 to 22 elements|

### Strict and lazy collections
Firstly, let's understand the concept of a transformer method. A *transformer method* is a method that constructs a new collection from an existing collection. This includes methods like `map, filter, reverse`, etc.

Given that definition, collections can also be thought of in terms of being strict or lazy. In a *strict* collection, memory for the elements is allocated immediately, and all of its elements are immediately evaluated when a transformer method is invoked.

In a *lazy* collection, memory for the elements is not allocated immediately, and transformer methods do not construct new elements until they are demanded.

All of the collection classes except `Stream` are strict, but the other collection classes can be converted to a lazy collection by creating a view on the collection. (More on this on Recipe 10.24)

## 10.3 Choosing a Collection Method to Solve a Problem
The methods that the Scala collection classes provide are listed in two ways in this recipe. In the tables that follow, a brief description and method signature is provided.

### Methods organized by category
*Filtering methods*
Methods that can be used to filter a collection include `collect, diff, distinct, drop, dropWhile, filter, filterNot, find, foldLeft, foldRight, head, headOption, init, intersect, last, lastOption, reduceLeft, reduceRight, remove, slice, tail, take, takeWhile`, and `union`.

*Transformer methods*
Transformer methods take at least one input collection to create a new output collection, typically using an algorithm you provide. They include `+, ++, −, −−, diff, distinct, collect, flatMap, map, reverse, sortWith, takeWhile, zip`, and `zipWithIndex`.

*Grouping methods*
These methods let you take an existing collection and create multiple groups from that one collection. These methods include `groupBy, partition, sliding, span, splitAt`, and `unzip`.

*Informational and mathematical methods*
These methods provide information about a collection, and include `canEqual, contains, containsSlice, count, endsWith, exists, find, forAll, hasDefiniteSize, indexOf, indexOfSlice, indexWhere, isDefinedAt, isEmpty, lastIndexOf, lastIndexOfSlice, lastIndexWhere, max, min, nonEmpty, product, segmentLength, size, startsWith, sum. The methods foldLeft, foldRight, reduceLeft`, and `reduceRight` can also be used with a function you supply to obtain information about a collection.

*Others*
A few other methods are hard to categorize, including `par, view, flatten, foreach`, and `mkString`. par creates a parallel collection from an existing collection; `view` creates a lazy view on a collection (see Recipe 10.24); `flatten` converts a list of lists down to one list; `foreach` is like a for loop, letting you iterate over the elements in a collection; `mkString` lets you build a String from a collection.

### Common collection methods
The following table lists methods that are common to all collections via `Traversable`. The following symbols are used in the first column of the table:
- `c` refers to a collection
- `f` refers to a function
- `p` refers to a predicate
- `n` refers to a number
- `op` refers to a simple operation (usually a simple function)

|Method |Description|
|---|---|
|c collect f |Builds a new collection by applying a partial function to all elements of the collection on which the function is defined. |
|c count p |Counts the number of elements in the collection for which the predicate is satisfied.|
|c1 diff c2 |Returns the difference of the elements in c1 and c2.|
|c drop n |Returns all elements in the collection except the first n elements.|
|c dropWhile p |Returns a collection that contains the “longest prefix of elements that satisfy the predicate.”|
|c exists p |Returns true if the predicate is true for any element in the collection.|
|c filter p |Returns all elements from the collection for which the predicate is true.|
|c filterNot p |Returns all elements from the collection for which the predicate is false.|
|c find p |Returns the first element that matches the predicate as Some[A]. Returns None if no match is found.|
|c flatten |Converts a collection of collections (such as a list of lists) to a single collection (single list).|
|c flatMap f |Returns a new collection by applying a function to all elements of the collection c (like map), and then flattening the elements of the resulting collections.|
|c foldLeft(z)(op) |Applies the operation to successive elements, going from left to right, starting at element z.|
|c foldRight(z)(op) |Applies the operation to successive elements, going from right to left, starting at element z.|
|c forAll p |Returns true if the predicate is true for all elements, false otherwise.|
|c foreach f |Applies the function f to all elements of the collection.|
|c groupBy f |Partitions the collection into a Map of collections according to the function.|
|c hasDefiniteSize |Tests whether the collection has a finite size. (Returns false for a Stream or Iterator, for example.)|
|c head |Returns the first element of the collection. Throws a NoSuchElementException if the collection is empty.|
|c headOption |Returns the first element of the collection as Some[A] if the element exists, or None if the collection is empty.|
|c init |Selects all elements from the collection except the last one. Throws an UnsupportedOperationException if the collection is empty.|
|c1 intersect c2 |On collections that support it, it returns the intersection of the two collections (the elements common to both collections).|
|c isEmpty |Returns true if the collection is empty, false otherwise.|
|c last |Returns the last element from the collection. Throws a NoSuchElementException if the collection is empty.|
|c lastOption |Returns the last element of the collection as Some[A] if the element exists, or None if the collection is empty.|
|c map f |Creates a new collection by applying the function to all the elements of the collection.|
|c max |Returns the largest element from the collection.|
|c min |Returns the smallest element from the collection|
|c nonEmpty |Returns true if the collection is not empty.|
|c par |Returns a parallel implementation of the collection, e.g., Array returns ParArray.|
|c partition p |Returns two collections according to the predicate algorithm.|
|c product |Returns the multiple of all elements in the collection.|
|c reduceLeft op |The same as foldLeft, but begins at the first element of the collection.|
|c reduceRight op |The same as foldRight, but begins at the last element of the collection.|
|c reverse |Returns a collection with the elements in reverse order. (Not available on Traversable, but common to most collections, from GenSeqLike.)|
|c size |Returns the size of the collection.|
|c slice(from, to) |Returns the interval of elements beginning at element from and ending at element to.|
|c sortWith f |Returns a version of the collection sorted by the comparison function f.|
|c span p |Returns a collection of two collections; the first created by c.takeWhile(p), and the second created by c.dropWhile(p).|
|c splitAt n |Returns a collection of two collections by splitting the collection c at element n.|
|c sum |Returns the sum of all elements in the collection.|
|c tail |Returns all elements from the collection except the first element.|
|c take n |Returns the first n elements of the collection.|
|c takeWhile p |Returns elements from the collection while the predicate is true. Stops when the predicate becomes false.|
|c1 union c2 |Returns the union (all elements) of two collections.|
|c unzip |The opposite of zip, breaks a collection into two collections by dividing each element into two pieces, as in breaking up a collection of Tuple2 elements.|
|c view |Returns a nonstrict (lazy) view of the collection.|
|c1 zip c2 |Creates a collection of pairs by matching the element 0 of c1 with element 0 of c2, element 1 of c1 with element 1 of c2, etc.|
|c zipWithIndex |Zips the collection with its indices.|

### Mutable collection methods

|Operator (method) |Description|
|---|---|
|c += x |Adds the element x to the collection c.|
|c += (x,y,z) |Adds the elements x, y, and z to the collection c.|
|c1 ++= c2 |Adds the elements in the collection c2 to the collection c1.|
|c −= x |Removes the element x from the collection c.|
|c −= (x,y,z) |Removes the elements x , y, and z from the collection c.|
|c1 −−= c2 |Removes the elements in the collection c2 from the collection c1.|
|c(n) = x |Assigns the value x to the element c(n).|
|c clear |Removes all elements from the collection.|
|c remove n OR c.remove(n, len)|Removes the element at position n, or the elements beginning at position n and continuing for length len.|

### Immutable collection operators
|Operator (method) |Description|
|---|---|
|c1 ++ c2 |Creates a new collection by appending the elements in the collection c2 to the collection c1.|
|c :+ e |Returns a new collection with the element e appended to the collection c.|
|e +: c |Returns a new collection with the element e prepended to the collection c.|
|e :: list |Returns a List with the element e prepended to the List named list. (:: works only on List.)|

### Maps
In this table, the following symbols are used:
- `m` refers to a map
- `mm` refers to a mutable map
- `k` refers to a key
- `p` refers to a predicate (function)
- `v` refers to a map value
- `c` refers to a collection

|Map method |Description|
|---|---|
|Methods for immutable maps| |
|m - k |Returns a map with the key k (and its corresponding value) removed.|
|m - (k1, k2, k3) |Returns a map with the keys k1, k2, and k3 removed.|
|m -- c (e.g. m -- List(k1, k2))| Returns a map with the keys in the collection removed. (Although List is shown, this can be any sequential collection.)|
|Methods for mutable maps|
|mm += (k -> v)| Add the key/value pair(s) to the mutable map mm.|
|mm += (k1 -> v1, k2 -> v2)| Add the key/value pair(s) to the mutable map mm. |
|mm ++= c (e.g. mm ++= List(3 -> "c"))| Add the elements in the collection c to the mutable map mm.
|mm -= k (e.g. mm -= (k1, k2, k3))| Remove map entries from the mutable map mm based on the given key(s).|
|mm --= c |Remove the map entries from the mutable map mm based on the keys in the collection c.|
|Methods for both mutable and immutable maps| |
|m(k) |Returns the value associated with the key k.|
|m contains k |Returns true if the map m contains the key k.|
|m filter p |Returns a map whose keys and values match the condition of the predicate p.|
|m filterKeys p |Returns a map whose keys match the condition of the predicate p.|
|m get k |Returns the value for the key k as Some[A] if the key is found, None otherwise.|
|m getOrElse(k, d) |Returns the value for the key k if the key is found, otherwise returns the default value d.|
|m isDefinedAt k |Returns true if the map contains the key k.|
|m keys |Returns the keys from the map as an Iterable.|
|m keyIterator |Returns the keys from the map as an Iterator.|
|m keySet |Returns the keys from the map as a Set.|
|m mapValues f |Returns a new map by applying the function f to every value in the initial map.|
|m values |Returns the values from the map as an Iterable.|
|m valuesIterator |Returns the values from the map as an Iterator|

## Continued in Part 2
