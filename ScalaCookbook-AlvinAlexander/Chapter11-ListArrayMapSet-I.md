# Chapter11 - List,Array,Map,Set (and More) - Part I

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
You can use a `ListBuffer` and then convert it to a `List` when needed. You can create a `ListBuffer` like this:
```
val fruits = new ListBuffer[String]()
```
The `ListBuffer` Scaladoc states that a `ListBuffer` is “a Buffer implementation backed by a list. It provides constant time prepend and append. Most other operations are linear.”

So, don’t use `ListBuffer` if you want to access elements **arbitrarily**, such as accessing items by index (like list(10000)); use `ArrayBuffer` instead.

## 11.3 Adding Elements to a List
Since List is immutable, you can't actually add elements to it. If you want a `List` that is constantly changing, use a `ListBuffer`, then convert it to `List` when necessary (as described in 11.2).

To work with a `List`, the general approach is to prepend items to the list with the `::` method while assigning the result to a new `List` (or reassign to itself).

The `::` method is right-associative; lists are constructed from right to left:
```
scala> val list1 = 3 :: Nil
list1: List[Int] = List(3)

scala> val list2 = 2 :: list1
list2: List[Int] = List(2, 3)

scala> val list3 = 1 :: list2
list3: List[Int] = List(1, 2, 3)
```

## 11.4 Deleting Elements from a List (or ListBuffer)
A `List` is immutable, so you can't delete elements from it, but you can filter out elements you don't want while you assign the result to a new variable.

If you are going to modify a list frequently, use a `ListBuffer` instead.

## 11.5 Merging Lists
You can merge two lists using the `++`, `concat`, or `:::` methods.

The `++` method is used consistently across immutable collections.

## 11.6 Using Stream, a Lazy Version of a List
What if you want to use a collection that works like a `List` but invokes its transformer methods lazily ?

A `Stream` is like a `List`, except that its elements are computed lazily, in a manner similar to how a *view* creates a lazy version of a collection. Other than this, a `Stream` behaves similarly to a `List`.

The REPL output shows that the stream begins with the number 1 but uses a `?` to denote the end of the stream. This is because the end of the stream hasn’t been evaluated yet.
For example, given a Stream:
```
scala> val stream = (1 to 100000000).toStream
stream: scala.collection.immutable.Stream[Int] = Stream(1, ?)
```
You can attempt to access the head and tail of the stream. The head is returned immediately, but the tail isn't evaluated yet.

The `?` symbol is the way a lazy collection shows that the end of the collection hasn't been evaluated yet. As discussed in 10.24, *transformer methods* are computed lazily, so when transformers are called, you see the `?` character that indicated the end of the stream hasn't been evaluated yet.

However, be careful with methods that aren't transformers. Calls to the following `strict` methods are evaluated immediately and can easily cause `OutOfMemoryError` errors:
```
stream.max
stream.size
stream.sum
```
If you attempt to use a `List` in these examples, as soon as you try to create the `List`, you will encounter a `java.lang.OutOfMemory` error.

Using a `Stream` gives you a chance to specify a huge list, and begin working with its elements.

## 11.7 Different Ways to Create and Update an Array
There are many different ways to define and populate an `Array`.

You can create an array with initial values, Scala will try to determine the array type implicitly:
```
scala> val a = Array(1, 2, 3)
a: Array[Int] = Array (1, 2, 3)

scala> val fruits = Array("Apple", "Banana", "Orange")
fruits: Array[String] = Array(Apple, Banana, Orange)
```
You can assign an array type manually, as such:
```
// scala makes this Array[Double]
scala> val x = Array(1, 2.0, 33D, 400L)
x: Array[Double] = Array(1.0, 2.0, 33.0, 400.0)

// manually override the type
scala> val x = Array[Number](1, 2.0, 33D, 400L)
x: Array[java.lang.Number] = Array(1, 2.0, 33.0, 400)
```
You can define an array with an initial size and type, and then populate it later:
```
// create an array with an initial size
val fruits = new Array[String](3)

// somewhere later in the code ...
fruits(0) = "Apple"
fruits(1) = "Banana"
fruits(2) = "Orange"
```
The elements inside an array can be changed, but its size cannot be changed. Scala arrays correspond one-to-one to Java arrays. A Scala array `Array[Int]` is represented as a Java `int[]`, `Array[Double]` is `double[]`, and `Array[String]` is `String[]` in Java.

The `Array` is an *indexed* sequential collection, so accessing and changing values by their index position is straightforward and fast.

## 11.8 Creating an Array Whose Size Can Change (ArrayBuffer)
To create a mutable, indexed sequence whose size can change, use the `ArrayBuffer` class. To use an `ArrayBuffer`, import it into scope and then create an instance. You can declare an `ArrayBuffer` without initial elements, and then add them later:

```
import scala.collection.mutable.ArrayBuffer
var characters = ArrayBuffer[String]()
characters += "Ben"
characters += "Jerry"
```
Or you can create the `ArrayBuffer` with elements inside:
```
val characters = collection.mutable.ArrayBuffer("Ben", "Jerry")
```

## 11.9 Deleting Array and ArrayBuffer Elements
An `ArrayBuffer` is a mutable sequence, so you can delete elements with the usual `-=`, `--=`, `remove`, and `clear` methods.

```
import scala.collection.mutable.ArrayBuffer
val x = ArrayBuffer('a', 'b', 'c', 'd', 'e')

// remove one element
x -= 'a'

// remove multiple elements (methods defines a varargs param)
x -= ('b', 'c')
```

Use `--=` to remove multiple elements that are declared in another collection (any collection that extends `TraversableOnce`):

```
val x = ArrayBuffer('a', 'b', 'c', 'd', 'e')
x --= Seq('a', 'b')
x --= Array('c')
x --= Set('d')
```

Use the `remove` method to delete one element by its position in the `ArrayBuffer`, or a series of elements beginning at a starting position:
```
scala> val x = ArrayBuffer('a', 'b', 'c', 'd', 'e', 'f')
x: scala.collection.mutable.ArrayBuffer[Char] = ArrayBuffer(a, b, c, d, e, f)

scala> x.remove(0)
res0: Char = a

scala> x
res1: scala.collection.mutable.ArrayBuffer[Char] = ArrayBuffer(b, c, d, e, f)

scala> x.remove(1, 3)

scala> x
res2: scala.collection.mutable.ArrayBuffer[Char] = ArrayBuffer(b, f)
```

Lastly, the `clear` method removes all the elements from an `ArrayBuffer`.

### Array
The size of an `Array` can't be changed, so you can't directly delete elements. You can reassign the elements in an `Array` by replacing them with `""` or `null`, which has the effect of replacing them.

You can also filter elements out of one array while you assign the result to a new array. If you define the array variable as a `var`, you can reassign the result back to itself.

## 11.10 Sorting Arrays
If you are working with an `Array` that holds elements that have an implicit `Ordering`, you can sort the `Array` in place using the `scala.util.Sorting.quickSort` method:
```
scala> val fruits = Array("cherry", "apple", "banana")
fruits: Array[String] = Array(cherry, apple, banana)

scala> scala.util.Sorting.quickSort(fruits)

scala> fruits
res0: Array[String] = Array(apple, banana, cherry)
```

If the type an `Array` is holding doesn't have an implicit `Ordering`, you can either modify it to mix in the `Ordered` trait, which gives it an implicit `Ordering`, or sort it using the `sorted`, `sortWith`, or `sortBy` methods. These approaches are shown in Recipe 10.29.

## 11.11 Creating Multidimensional Arrays
There are two main solutions:
1. Use `Array.ofDim` to create a multidimensional array of up to five dimensions. With this approach you need to know the number of rows and columns at creation time.
2. Create arrays of arrays as needed.

### Using Array.ofDim
```
scala> val rows = 2
rows: Int = 2

scala> val cols = 3
cols: Int = 3

scala> val a = Array.ofDim[String](rows, cols)
a: Array[Array[String]] = Array(Array(null, null, null), Array(null, null, null))
```
Then, you can assign elements to it. You can access the elements using parentheses, similar to a one-dimensional array.

To create an array with more dimensions, just follow the same pattern. For example:
```
val x, y, z = 10
val a = Array.ofDim[Int](x,y,z)
for {
  i <- 0 until x
  j <- 0 until y
  k <- 0 until z
} println(s"($i)($j)($k) = ${a(i)(j)(k)}")
```

### Using an array of arrays
```
scala> val a = Array( Array("a", "b", "c"), Array("d", "e", "f") )
a: Array[Array[String]] = Array(Array(a, b, c), Array(d, e, f))

scala> a(0)
res0: Array[String] = Array(a, b, c)

scala> a(0)(0)
res1: String = a
```
This gives you more control of the process, and lets you create “ragged” arrays (where each contained array may be a different size):

Finally, the Array.ofDim approach is unique to the Array class; there is no ofDim method on a List, Vector, ArrayBuffer, etc. But the “array of arrays” solution is not unique to the Array class. You can have a “list of lists,” “vector of vectors,” and so on.
