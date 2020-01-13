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
