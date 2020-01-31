## 11.24 Adding Elements to a Set
Mutable and immutable sets are handled differently.

### Mutable set  
Add elements to a *mutable* set with the `+=`, `++=`, and `add` methods
```
// use var with mutable
scala> var set = scala.collection.mutable.Set[Int]()
set: scala.collection.mutable.Set[Int] = Set()

// add one element
scala> set += 1
res0: scala.collection.mutable.Set[Int] = Set(1)

// add multiple elements
scala> set += (2, 3)
res1: scala.collection.mutable.Set[Int] = Set(2, 1, 3)

// notice that there is no error when you add a duplicate element
scala> set += 2
res2: scala.collection.mutable.Set[Int] = Set(2, 6, 1, 4, 3, 5)

// add elements from any sequence (any TraversableOnce)
scala> set ++= Vector(4, 5)
res3: scala.collection.mutable.Set[Int] = Set(2, 1, 4, 3, 5)

scala> set.add(6)
res4: Boolean = true

scala> set.add(5)
res5: Boolean = false
```

The last two examples demonstrate a unique characteristic of the `add` method on a set: It returns `true` or `false` depending on whether or not the element was added. The other methods silently fail if you attempt to add an element that's already in the set.

You can test to see whether a set contains an element before adding it by using the `.contains` method.

### Immutable set
Firstly, create an immutable set:
```
scala> val s1 = Set(1, 2)
s1: scala.collection.immutable.Set[Int] = Set(1, 2)
```
Create a new set by adding elements to a previous set with the `+` and `++` methods:
```
// add one element
scala> val s2 = s1 + 3
s2: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

// add multiple elements (+ method has a varargs field)
scala> val s3 = s2 + (4, 5)
s3: scala.collection.immutable.Set[Int] = Set(5, 1, 2, 3, 4)

// add elements from another sequence
scala> val s4 = s3 ++ List(6, 7)
s4: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 7, 3, 4)
```
You can also declare your variable as a `var`, and reassign the resulting set back to the same variable.

## 11.25 Deleting Elements from Sets
### Mutable set
When working with a *mutable* `Set`, remove elements using the `-=` and `--=` methods, as shown in the following examples:

```
scala> var set = scala.collection.mutable.Set(1, 2, 3, 4, 5)
set: scala.collection.mutable.Set[Int] = Set(2, 1, 4, 3, 5)

// one element
scala> set -= 1
res0: scala.collection.mutable.Set[Int] = Set(2, 4, 3, 5)

// two or more elements (-= has a varags field)
scala> set -= (2, 3)
res1: scala.collection.mutable.Set[Int] = Set(4, 5)

// multiple elements defined in another sequence
scala> set --= Array(4,5)
res2: scala.collection.mutable.Set[Int] = Set()
```

You can also use methods like `retain`, `clear`, and `remove`, depending on your needs.

### Immutable set
Similarly, you can use the `-` and `--` operators to remove elements while assigning the result to a new variable. You can also use the `filter` or `take` methods.

## 11.26 Using Sortable Sets
To retrieve values from a set in sorted order, use a `SortedSet`. To retrieve elements from a set in the order in which elements were inserted, use a `LinkedHashSet`.

The `SortedSet` is available only in an immutable version. If you need a mutable version, use the `java.util.TreeSet`.

The `LinkedHashSet` is available only as a mutable collection.

The examples shown in the Solution work because the types used in the sets have an implicit `Ordering`. Custom types won’t work unless you also provide an implicit `Ordering`. For example, the following code won’t work because the `Person` class is just a basic class:
```
class Person (var name: String)

import scala.collection.SortedSet
val aleka = new Person("Aleka")
val christina = new Person("Christina")
val molly = new Person("Molly")
val tyler = new Person("Tyler")

// this won't work
val s = SortedSet(molly, tyler, christina, aleka)
```
To solve this problem, modify the `Person` class to extend the `Ordered` trait, and implement a `compare` method:
```
class Person (var name: String) extends Ordered [Person]
{
  override def toString = name

  // return 0 if the same, negative if this < that, positive if this > that
  def compare (that: Person) = {
    if (this.name == that.name)
      0
    else if (this.name > that.name)
      1
    else
      −1
  }
}
```

## 11.27 Using a Queue
A queue is a FIFO data structure. Scala offers both an immutable and mutable queue. This recipe demonstrates the *mutable* queue.

You can create an empty, mutable queue of any data type:
```
import scala.collection.mutable.Queue
var ints = Queue[Int]()
var fruits = Queue[String]()
var q = Queue[Person]()
```

You can create a queue with initial elements:
```
scala> val q = Queue(1, 2, 3)
q: scala.collection.mutable.Queue[Int] = Queue(1, 2, 3)
```
Once you have a mutable queue, add elements to it using `+=`, `++=`, and `enqueue`. To remove elements from the head of the queue, one element at a time, using `dequeue`. You can use `dequeueFirst` and `dequeueAll` methods to remove elements from the queue.

A Queue is a collection class that extends from `Iterable` and `Traversable`, so it has all the usual collection methods, including `foreach`, `map`, etc. See the `Queue` Scaladoc for more information.

## 11.28 Using a Stack
A stack is a LIFO data structure. Scala has both immutable and mutable versions of a stack, as well as an `ArrayStack`.

Create an empty, mutable stack of any data type:
```
import scala.collection.mutable.Stack
var ints = Stack[Int]()
var fruits = Stack[String]()

case class Person(var name: String)
var people = Stack[Person]()
```

You can also populate a stack with initial elements when you create it:
```
val ints = Stack(1, 2, 3)
```
Push elements into the stack with `push`,  and take elements off the stack with `pop`. To peek at the element at the top of the stack without removing it, use `peek`. You can empty a mutable stack with `clear`.

`ArrayStack` provides fast indexing and is generally slightly more efficient for most operations than a normal mutable stack. If you want to use an immutable stack, use a `List` instead. A `List` has less layers of code, and you can push elements onto the `List` with `::` and access the first element with the `head` method.

## 11.29 Using a Range
Ranges are often used to populate data structures, and to iterate over `for loops`.
```
scala> 1 to 10
res0: scala.collection.immutable.Range.Inclusive =
  Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> 1 until 10
res1: scala.collection.immutable.Range = Range(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> 1 to 10 by 2
res2: scala.collection.immutable.Range = Range(1, 3, 5, 7, 9)

scala> 'a' to 'c'
res3: collection.immutable.NumericRange.Inclusive[Char] = NumericRange(a, b, c)
```
You can use ranges to create and populate sequences:
```
scala> val x = (1 to 10).toList
x: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> val x = (1 to 10).toArray
x: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> val x = (1 to 10).toSet
x: scala.collection.immutable.Set[Int] = Set(5, 10, 1, 6, 9, 2, 7, 3, 8, 4)
```

Some sequences have a `range` method in their objects:
```
scala> val x = Array.range(1, 10)
x: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val x = Vector.range(1, 10)
x: collection.immutable.Vector[Int] = Vector(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val x = List.range(1, 10)
x: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> val x = List.range(0, 10, 2)
x: List[Int] = List(0, 2, 4, 6, 8)

scala> val x = collection.mutable.ArrayBuffer.range('a', 'd')
x: scala.collection.mutable.ArrayBuffer[Char] = ArrayBuffer(a, b, c)
```
In addition to the approaches shown, a Range can be combined with the map method to populate a collection:
```
scala> val x = (1 to 5).map { e => (e + 1.1) * 2 }
x: scala.collection.immutable.IndexedSeq[Double] =
  Vector(4.2, 6.2, 8.2, 10.2, 12.2)
```
While discussing ways to populate collections, the tabulate method is another nice approach:
```
scala> val x = List.tabulate(5)(_ + 1)
x: List[Int] = List(1, 2, 3, 4, 5)

scala> val x = List.tabulate(5)(_ + 2)
x: List[Int] = List(2, 3, 4, 5, 6)

scala> val x = Vector.tabulate(5)(_ * 2)
x: scala.collection.immutable.Vector[Int] = Vector(0, 2, 4, 6, 8)
```
