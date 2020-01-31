# Chapter11 - List,Array,Map,Set (and More) - Part II

## 11.12 Creating Maps
To use an immutable map, you can just create a Map:
```
scala> val states = Map("AL" -> "Alabama", "AK" -> "Alaska")
states: scala.collection.immutable.Map[String,String] =
  Map(AL -> Alabama, AK -> Alaska)
```
To create a mutable map, you can use an import statement to bring it into scope, or specify the full path to the `scala.collection.mutable.Map` class when you create an instance.

```
scala> var states = collection.mutable.Map("AL" -> "Alabama")
states: scala.collection.mutable.Map[String,String] = Map(AL -> Alabama)
```
You can also create maps like this:
```
scala> val states = Map( ("AL", "Alabama"), ("AK", "Alaska") )
states: scala.collection.immutable.Map[String,String] =
  Map(AL -> Alabama, AK -> Alaska)
```

## 11.13 Choosing a Map Implementation
There are a few map types to choose from.

If you're looking for a basic map class, where sorting or insertion order doesn't matter, you can either choose the default immutable `Map`, or the mutable `Map`.

If you want a map that returns its elements in sorted order by keys, use a `SortedMap`.
```
scala> import scala.collection.SortedMap
import scala.collection.SortedMap

scala> val grades = SortedMap("Kim" -> 90,
  | "Al" -> 85,
  | "Melissa" -> 95,
  | "Emily" -> 91,
  | "Hannah" -> 92
  | )
grades: scala.collection.SortedMap[String,Int] =
Map(Al -> 85, Emily -> 91, Hannah -> 92, Kim -> 90, Melissa -> 95)
```

If you want a map that remembers the insertion order of its elements, use a `LinkedHashMap` or `ListMap`. Scala only has *mutable* `LinkedHashMap`, and it returns its elements in the order you inserted them. `ListMap` return elements in the opposite order in which you inserted them.

Here are the summary of the basic Scala map classes and traits:

| Class or trait | Description |
| --- | --- |
| collection.immutable.Map | This is the default, general-purpose immutable map you get if you don’t import anything. |
| collection.mutable.map | A mutable version of the basic map. |
| collection.mutable.LinkedHashMap | All methods that traverse the elements will visit the elements in their insertion order. |
| collection.immutable.ListMap & collection.mutable.ListMap | Per the Scaladoc, “implements immutable maps using a list-based data structure.” As shown in the examples, elements that are added are prepended to the head of the list. |
| collection.SortedMap | Keys of the map are returned in sorted order. Therefore, all traversal methods (such as `foreach`) return keys in that order. |
| collection.immutable.HashMap | Implements immutable maps using a hash trie |
| collection.mutable.ObservableMap | This class is typically used as a mixin. It adds a subscription mechanism to the Map class into which this abstract class is mixed in. |
| collection.mutable.MultiMap | A trait for mutable maps with multiple values assigned to a key |
| collection.mutable.SynchronizedMap | This trait should be used as a mixin. It synchronizes the map functions of the class into which it is mixed in. |
| collection.immutable.TreeMap | Implements immutable maps using a tree. |
| collection.mutable.WeakHashMap | A wrapper around java.util.WeakHashMap, a map entry is removed if the key is no longer strongly referenced |

## 11.14 Adding, Updating, and Removing Elements with a Mutable Map
Add elements to a mutable map by simply assigning them, or with the `+=` method. Remove elements with `-=` or `--=`. Update elements by reassigning them.

You can also use `put` to add an element (or replace an existing element); `retain` to keep only the elements by its key value; and `clear` to delete all elements in the map. The `remove` method returns an Option that contains the value that was removed. Here are the examples:
```
scala> val states = collection.mutable.Map(
    | "AK" -> "Alaska",
    | "IL" -> "Illinois",
    | "KY" -> "Kentucky"
    | )
states: collection.mutable.Map[String,String] =
Map(KY -> Kentucky, IL -> Illinois, AK -> Alaska)

scala> states.put("CO", "Colorado")
res0: Option[String] = None

scala> states.retain((k,v) => k == "AK")
res1: states.type = Map(AK -> Alaska)

scala> states.remove("AK")
res2: Option[String] = Some(Alaska)

scala> states
res3: scala.collection.mutable.Map[String,String] = Map()

scala> states.clear

scala> states
res4: scala.collection.mutable.Map[String,String] = Map()
```
If the element put into the collection by `put` replaced another element, that value would be returned. Because this example didn’t replace anything, it returned `None`.

## 11.15 Adding, Updating, and Removing Elements with Immutable Maps
You can use the same operators as the ones mentioned above, but you have to assign the results to a new map.

To update a key/value pair with an immutable map, reassign the key and value while using the + method, and the new values replace the old.

You can declare an immutable map as either a `val` or as a `var`. You can create an immutable map as a `var`, you still have an immutable map.

Given a sample map:
```
scala> val states = Map("AL" -> "Alabama", "AK" -> "Alaska", "AZ" -> "Arizona")
states: scala.collection.immutable.Map[String,String] =
  Map(AL -> Alabama, AK -> Alaska, AZ -> Arizona)
```
Access the value associated with a key in the same way you access an element in an array:
```
scala> val az = states("AZ")
az: String = Arizona
```
If the map doesn't contain the requested key, a `java.util.NoSuchElementException` exception is thrown. One way to avoid this problem is to create the map with the `withDefaultValue` method. Another option is to use the `getOrElse` method when attempting to find a value.

## 11.17 Traversing a Map
For example, given this map:
```
val ratings = Map("Lady in the Water"-> 3.0,
                  "Snakes on a Plane"-> 4.0,
                  "You, Me and Dupree"-> 3.5)
```
One way to loop over all of the map elements is with a `for loop` syntax:
```
for ((k,v) <- ratings) println(s"key: $k, value: $v")
```
You can also use a match expression with the `foreach` method:
```
ratings.foreach {
  case(movie, rating) => println(s"key: $movie, value: $rating")
}
```
You can also just iterate over the keys or just over the values:
```
ratings.keys.foreach(println)
ratings.values.foreach(println)
```
### Operating on map values
If you want to traverse the map to perform an operation on its values, the `mapValues` method may be a better solution. It lets you perform a function on each map value and returns the modified map:
```
scala> var x = collection.mutable.Map(1 -> "a", 2 -> "b")
x: scala.collection.mutable.Map[Int,String] = Map(2 -> b, 1 -> a)

scala> val y = x.mapValues(_.toUpperCase)
y: scala.collection.Map[Int,String] = Map(2 -> B, 1 -> A)
```
The ``transform`` method gives you another way to create a new map from an existing map. Unlike `mapValues`, it lets you use both the key and value to write a transformation method:
```
scala> val map = Map(1 -> 10, 2 -> 20, 3 -> 30)
map: scala.collection.mutable.Map[Int,Int] = Map(2 -> 20, 1 -> 10, 3 -> 30)

scala> val newMap = map.transform((k,v) => k + v)
newMap: map.type = Map(2 -> 22, 1 -> 11, 3 -> 33)
```

## 11.18 Getting the Keys or Values from a Map
To get the keys, use `keySet` to get the keys as a `Set`, `keys` to get an `Iterable`, or `keysIterator` to get the keys as an iterator.

To get the values, use `values` method to get the values as an `Iterable`, or `valuesIterable` to get the keys as an `Iterator`.

## 11.19 Reversing Keys and Values
You can reverse the keys and values of a map with a *for comprehension*, and assign the result to a new variable.
```
val reverseMap = for ((k, v) <- map) yield (v, k)
```
Be aware that values don't have to be unique, and keys must be, so you might lose some content.

## 11.20 Testing for the Existence of a Key or Value in a Map
To test for the existence of a **key** in a map, use the `contains` method.
```
scala> if (states.contains("FOO")) println("Found foo") else println("No foo")
```
To test whether a **value** exists in a map, use the `valuesIterator` method to search for the value using `exists` and `contains`:
```
scala> states.valuesIterator.exists(_.contains("foo"))
```
When chaining methods like this together, be careful about intermediate results.

## 11.21 Filtering a Map
You can use the `retain` method to define the elements to retain when using a mutable map, and use `filterKeys` or `filter` to filter the elements in a mutable/immutable map (assign the result to a new variable for immutable maps).

```
scala> var x = collection.mutable.Map(1 -> "a", 2 -> "b", 3 -> "c")
x: scala.collection.mutable.Map[Int,String] = Map(2 -> b, 1 -> a, 3 -> c)

scala> x.retain((k,v) => k > 1)
res0: scala.collection.mutable.Map[Int,String] = Map(2 -> b, 3 -> c)

scala> x
res1: scala.collection.mutable.Map[Int,String] = Map(2 -> b, 3 -> c)
```
`retain` modifies a mutable map in place. Your algorithm can test both the key and value of each element to decide which elements to retain in the map.

The `transform` method doesn't filter a map, but it lets you transform the elements in a mutable map.

### Mutable and immutable maps
You can use a predicate with the `filterKeys` methods to define which map elements to retain. When using this method, remember to assign the filtered result to a new variable.

The predicate you supply should return `true` for the elements you want to keep in the new collection and `false` for the elements you don't want.

You can also use all the filtering methods that are shown in Chapter 10.

## 11.22 Sorting an Existing Map by Key or Value
You can sort the map by *key* from low to high using `sortBy`. You can also sort the keys in ascending or descending order using `sortWith`.

You can sort the map by *value* using `sortBy`. Similarly, you can also sort the keys in ascending or descending order using `sortWith`.

## 11.23 Finding the Largest Key or Value in a Map
You can use the `max` method on the map. Alternatively, use the map's `keysIterator` or `valuesIterator` with other approaches, depending on your needs.

You can call `keysIterator` to get an iterator over the map keys, and call its `max` method: `mapName.keysIterator.max`.

You can find the same maximum using `keysIterator` and `reduceLeft`:
```
mapName.keysIterator.reduceLeft((x,y) => if (x > y) x else y)
```
This approach is flexible, because if your definition of “largest” is the longest string, you can compare string lengths instead:

```
mapName.keysIterator.reduceLeft((x,y) => if (x.length > y.length) x else y)
```

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
