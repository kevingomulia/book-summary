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
