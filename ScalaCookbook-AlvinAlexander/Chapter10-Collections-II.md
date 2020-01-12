# Chapter 10 - Collections - II

## 10.4 Understanding the Performance of Collections
In many cases, you can reason about the performance of a collection by understanding its basic structure. For instance, a `List` is a singly linked list. It’s not indexed, so if you need to access the one-millionth element of a List as `list(1000000)`, that will be slower than accessing the one-millionth element of an `Array`, because the `Array` is indexed, whereas accessing the element in the List requires traversing the length of the `List`.

| Key | Description |
| --- | --- |
| C | The operation takes constant time |
| eC | The operation takes effectively constant time, but this might depend on some assumptions, such as maximum length of a vector, or distribution of hash key |
| aC | The operation takes amortized constant time. |
| Log | The operation takes time proportional to the logarithm of the collection size |
| L | The operation is linear - proportional to the collection size |
| - | The operation is not supported |

The table below shows the performance characteristics for operations on immutable and mutable sequential collections

| | head | tail | apply | update | prepend | append | insert |
|---|---|---|---|---|---|---|---|
|Immutable|
|List| C | C | L | L | C | L | - |
|Stream| C | C | L | L | C | L | - |
| Vector | eC | eC | eC | eC | eC | eC | - |
|Stack| C | C | L | L | C | C | L |
|Queue| aC | aC | L | L | L | C | - |
|Range| C | C | C | - | - | - | - |
|String| C | L | C | L | L | L | - |
| |
|Mutable|
|ArrayBuffer| C | L | C | C | L | aC | L |
|ListBuffer| C | L | L | L | C | C | L |
|StringBuilder| C | L | C | C | L | aC | L |
|MutableList| C | L | L | L | C | C | L |
|Queue| C | L | L | L | C | C | L |
|ArraySeq| C | L | C | C | - | - | - |
|Stack| C | L | L | L | C | L | L |
|ArrayStack| C | L | C | C | aC | L | L |
|Array| C | L | C | C | - | - | - |

- `head` - Selecting the first element of the sequence
- `tail` - Produces a new sequence that consists of all elements of the sequence but the first one
- `apply` - Indexing
- `update` - Functional update for immutable sequences, side-effecting update (with update) for mutable sequences
- `prepend` - Adding an element to the front of the sequence. For immutable sequences, this produces a new sequence. For mutable sequences, it modifies the existing sequence.
- `append` - Adding an element at the end of the sequence. Similar to prepend.
- `insert` - Inserting an element at an arbitrary position of the sequence. This is supported directly only for mutable sequences.

### Map and set performance characteristics
| | lookup | add | remove | min |
|---|---|---|---|---|
|Immutable|
|HashSet/HashMap| eC | eC | eC | L |
|TreeSet/TreeMap| Log | Log | Log | Log |
|BitSet | C | L | L | eC |
|ListMap| L | L | L | L |
||
|Mutable|
|HashSet/HashMap| eC | eC | eC | L |
|WeakHashMap| eC | eC | eC | L |
|BitSet| C | aC | C | eC |
|TreeSet| Log | Log | Log | Log |

- `lookup` - Testing whether an element is contained in a set, or selecting a value associated with a map key`
- `add` - adding a new element to a set or key/value pair to a Map
- `remove` - Removing an element from a set or a key from a map
- `min` - The smallest element of the set or the smallest key of a map.

## 10.5 Declaring a Type When Creating a Collection
Let's say you want to create a collection of mixed types, and Scala isn't automatically assigning the type you want.

For example, here Scala automatically assigns a type of `Double` to the list:
```
scala> val x = List(1, 2.0, 33D, 400L)
x: List[Double] = List(1, 2.0, 33.0, 400.0)
```
If you'd rather have the collection be of type `AnyVal` or `Number`, specify the type in brackets before your collection declaration:
```
scala> val x = List[Number](1, 2.0, 33D, 400L)
x: List[java.lang.Number] = List(1, 2.0, 33.0, 400)

scala> val x = List[AnyVal](1, 2.0, 33D, 400L)
x: List[AnyVal] = List(1, 2.0, 33.0, 400)
```

## 10.6 Understanding Mutable Variables with Immutable Collections
Mixing a mutable variable (`var`) with an immutable collection causes surprising behaviour. For example, hen you create an immutable `Vector` as a `var`, you can add new elements to it.

```
scala> var sisters = Vector("Melinda")
sisters: collection.immutable.Vector[String] = Vector(Melinda)

scala> sisters = sisters :+ "Melissa"
sisters: collection.immutable.Vector[String] = Vector(Melinda, Melissa)

scala> sisters = sisters :+ "Marisa"
sisters: collection.immutable.Vector[String] = Vector(Melinda, Melissa, Marisa)

1scala> sisters.foreach(println)
Melinda
Melissa
Marisa
```
Each time you use the `:+` method, `sisters` variable points to a new collection. The `sisters` variable is mutable, and it is being reassigned to a new collection during each step.

To be clear about *variables*:
- A mutable variable `var` can be reassigned to point at new data
- An immutable variable `val` is like a `final` variable in Java; it can never be reassigned

To be clear about *collections*:
- The elements in a mutable collection can be changed
- The elements in an immutable collection cannot be changed

## 10.7 Make Vector Your "Go To" Immutable Sequence
If you want a **fast, general-purpose, immutable, sequential** collection type, you can use the `Vector` class.
**Note:** Use `Vector` for **indexed, immutable** sequential collection. For **linear, immutable** sequential collection, use a `List` instead.

If you create an instance of an `IndexedSeq`, Scala returns a `Vector`.

## 10.8 Make ArrayBuffer Your "Go To" Mutable Sequence
The `ArrayBuffer` class is recommended as the general-purpose class for *mutable* sequential collections. Again, `ArrayBuffer` is an indexed sequential collection, while `ListBuffer` is a linear sequential collection.

Append, update, and random access takes constant time (amortized time). Prepends and removes are linear in the buffer size. Also, array buffers are useful for efficiently building up a large collection whenever the new items are always added to the end.

A `ListBuffer` is like an array buffer except that it uses a linked list internally instead of an array. If you plan to convert the buffer to a list once it is built up, use a list buffer instead of an array buffer.

## 10.9 Looping over a Collection with foreach
The `foreach` method takes a function as an argument. The function you define should take an element as an input parameter, and should not return anything. The input parameter type should match the type sorted in the collection. As `foreach` executes, it passes one element at a time from the collection to your function until it reaches the last element in the collection.

As an example, a common use of `foreach` is to output information:
```
scala> val x = Vector(1, 2, 3)
x: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3)

scala> x.foreach((i: Int) => println(i))
1
2
3
```
You can use a shorthand notation, which is what is typically used:
```
x.foreach(println)
```
As long as your function/method takes one parameter of the same type as the elements in the collection and returns nothing (`Unit`), it can be called from a `foreach` method.

## 10.10 Looping over a Collection with a for Loop
You can loop over the elements in a collection using a `for` loop, and may create a new collection from the existing collection using the `for/yield` combination.

You can loop over any `Traversable` type (basically any sequence) using a `for` loop. You can also use the `zipWithIndex` method when you need a loop counter:

```
for ((elem, count) <- fruits.zipWithIndex) {
  println(s"element $count is $elem)
}

element 0 is apple
element 1 is banana
...
```
Alternatively, use `zip` with a `Stream`:
```
for ((elem, count) <- fruits.zip(Stream from 1)) {
  println(s"element $count is $elem)
}
```
### The for/yield construct
To build a new collection from an input collection (returns an output), use the `for/yield` construct. The following example shows the `for/yield` construct:
```
scala> val fruits = Array("apple", "banana", "orange")
fruits: Array[java.lang.String] = Array(apple, banana, orange)

scala> val newArray = for (e <- fruits) yield e.toUpperCase
newArray: Array[java.lang.String] = Array(APPLE, BANANA, ORANGE)
```
The `for/yield` construct returns (yields) a new collection from the input collection by applying your algorithm to the elements of the input collection, so the array `newArray` contains uppercase versions of the three strings in the initial array. Using the `for/yield` like this is known as a *for comprehension*.

If your for/yield processing requires multiple lines of code, perform the work in a block after the `yield` keyword.

You can also combine a `for` loop with `if` statements, which are known as *guards*:
```
for {
  file <- files
  if file.isFile
  if file.getName.endsWith(".txt")
} doSomething(file)
```
## 10.11 Using zipWithIndex or zip to Create Loop Counters
We have seen how to use zipWithIndex to create a loop counter in Recipe 10.10. `zipWithIndex` returns a series of `Tuple2` elements in an `Array`, like this:
```
Array((Sunday, 0), (Monday, 1), ...)
```
Because `zipWithIndex` creates a new sequence from the existing sequence, you may want to call `view` before invoking `zipWithIndex`, like this:
```
scala> val zwi2 = list.view.zipWithIndex
zwi2: scala.collection.SeqView[(String, Int),Seq[_]] = SeqViewZ(...)
```
As shown, this creates a lazy view on the original list, so the tuple elements won't be created until they're needed.

Calling `zipWithIndex` alone trigger an extra iteration through the collection (traverses twice), and also creates an intermediary array of pairs. When using a `view`, the collection is only traversed when required so there is no performance loss.

## 10.12 Using Iterators
In Scala, we hardly use the iterator method (with `hasNext()` and `next()` in Java). Scala collections have methods like `map` and `foreach` that let you implement algorithms more concisely.

However, you might still run into an iterator. One of the best examples being the `io.Source.fromFile` method. An important part of using an iterator is knowing that it's exhausted after you use it. As you access each element, you mutate the iterator, and the previous element is discarded.
```
scala> val it = Iterator(1,2,3)
it: Iterator[Int] = non-empty iterator

scala> it.foreach(println)
1
2
3
```
But when you try to run the same call again, you won't get any output because the iterator has been exhausted.

## 10.13 Transforming One Collection to Another with for/yield
You can use the `for/yield` construct and your algorithm to create a new collection. For instance:
```
scala> val a = Array(1, 2, 3, 4, 5)
a: Array[Int] = Array(1, 2, 3, 4, 5)
```
You can create a copy of that collection by just "yielding" each element.
```
scala> for (e <- a) yield e
res0: Array[Int] = Array(1, 2, 3, 4, 5)
```
Or you can modify them too:
```
scala> for (e <- a) yield e * 2
res1: Array[Int] = Array(2, 4, 6, 8, 10)
```
This combination of a `for` loop and `yield` statement is known as *for comprehension* or *sequence comprehension*. It yields a new collection from an existing collection.

In general, the collection type that's returned b y a for comprehension will be the same type that you begin with. However, this is not always the case.

### Using gurads
When you add guards to a for comprehension and want to write it as a multiline expression, use the curly braces.
```
for {
  file <- files
  if hasSoundFileExtension(file)
  if !soundFileIsLong(file)
} yield file
```
## 10.14 Transforming One Collection to Another with map
Instead of using `for/yield` combination shown in the previous recipe, call the `map` method on your collection, passing it a function, an anonymous function, or method to transform each element. For example:
```
scala> val helpers = Vector("adam", "kim", "melissa")
helpers: scala.collection.immutable.Vector[java.lang.String] =
Vector(adam, kim, melissa)

// the long form
scala> val caps = helpers.map(e => e.capitalize)
caps: scala.collection.immutable.Vector[String] = Vector(Adam, Kim, Melissa)

// the short form
scala> val caps = helpers.map(_.capitalize)
caps: scala.collection.immutable.Vector[String] = Vector(Adam, Kim, Melissa)
```

For simple cases, using `map` is the same as using a basic `for/yield` loop. But once you add a guard, a `for/yield` loop is no longer directly equivalent to just a `map` method call. If you attempt to use an `if` statement in the algorithm you pass to a `map` method, you'll get a very different result:
```
scala> val fruits = List("apple", "banana", "lime", "orange", "raspberry")
fruits: List[java.lang.String] = List(apple, banana, lime, orange, raspberry)

scala> val newFruits = fruits.map( f =>
 if (f.length < 6) f.toUpperCase
)
newFruits: List[Any] = List(APPLE, (), LIME, (), ())
```
You could filter the result after calling `map` to clean up the result:
```
scala> newFruits.filter(_ != ())
res0: List[Any] = List(APPLE, LIME)
```

## Continued in Part 3
