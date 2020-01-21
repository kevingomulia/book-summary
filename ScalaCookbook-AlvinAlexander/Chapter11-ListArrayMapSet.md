# Chapter11 - List,Array,Map,Set (and More)

### List
Despite their names, the Scala `List` class is nothing like the Java `List` classes. The Scala `List` is **immutable**, implemented as a linked list, and is generally thought of in terms of its `head`, `tail` and `isEmpty` methods.

Most operations on a `List` involve recursive algorithms, where the algorithm splits the list into its head and tail components.

### Array (and ArrayBuffer)
Arrays are mutable, indexed collections of values. The class is mutable in that its elements can be changed, but the size of an `Array` cannot be changed.

The recommendation with Scala 2.10.x is to use the `Vector` class as your "go to" *immutable, indexed* sequence class, and `ArrayBuffer` as your *mutable*, indexed sequence of choice.

### Maps
A Scala `Map`, by default, is immutable (unlike in Java).

### Sets
a Scala `Set` is also like a Java `Set`.

## 11.1 Different Ways to Create and Populate a List
There are many ways to create and initially populate a `List`:
```
// 1
scala> val list = 1 :: 2 :: 3 :: Nil
list: List[Int] = List(1, 2, 3)

// 2
scala> val list = List(1, 2, 3)
x: List[Int] = List(1, 2, 3)

// 3a
scala> val x = List(1, 2.0, 33D, 4000L)
x: List[Double] = List(1.0, 2.0, 33.0, 4000.0

// 3b
scala> val x = List[Number](1, 2.0, 33D, 4000L)
x: List[java.lang.Number] = List(1, 2.0, 33.0, 4000)

// 4
scala> val x = List.range(1, 10)
x: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val x = List.range(0, 10, 2)
x: List[Int] = List(0, 2, 4, 6, 8)

// 5
scala> val x = List.fill(3)("foo")
x: List[String] = List(foo, foo, foo)

// 6
scala> val x = List.tabulate(5)(n => n * n)
x: List[Int] = List(0, 1, 4, 9, 16)

// 7
scala> val x = collection.mutable.ListBuffer(1, 2, 3).toList
x: List[Int] = List(1, 2, 3)

// 8
scala> "foo".toList
res0: List[Char] = List(f, o, o)
```
The following quote from the Scala `List` Scaladoc discusses the important properties of the `List` class:
```
This class is optimal for last-in-first-out (LIFO), stack-like access patterns. If you need another access pattern, for example, random access or FIFO, consider using a collection more suited to this than List. List has O(1) prepend and head/tail access. Most other operations are O(n) on the number of elements in the list.
```
## 11.2 Creating a Mutable List
