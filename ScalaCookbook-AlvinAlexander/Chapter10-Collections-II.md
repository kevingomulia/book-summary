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
