# Chapter 3 - Control Structures

Scala's `if/then/else` structure is similar to Java, but can also be used to return a value.
In Scala, you can use a normal `if` statement to achieve the ternary effect:
```
val x = if (a) y else z
```
The `try/catch/finally` structure is similar to Java, though Scala uses pattern matching in the `catch` clause (Scala doesn't have checked exceptions).

Scala's `for` loop's basic use is similar to Java, but it has an addition of *guards* and other conveniences. For example:

You could write two for loops to read every line in a file and then operate on each character in each line:
```
for (line <- source.getLines) {
  for {
    char <- line
    if char.isLetter
  } // char algorithm here ...
}
```
But with Scala’s for loop mojo, you can write this code even more concisely:
```
for {
  line <- source.getLines
  char <- line
  if char.isLetter
} // char algorithm here ...
```

## 3.1 Looping with for and foreach [COMMON]
There are many ways to loop over Scala collections, including `for` loops, `while` loops, and collection methods like `foreach`, `map`, `flatMap`, etc. We will focus first on the `for` loop and `foreach` method.

This is the common syntax for a for loop:
```
for (item <- collection) {
  // Do something
}
```

### Returning values from a for loop
Those examples perform an operation using the elements in an array, but they don’t return a value you can use, such as a new array. In cases where you want to build a new collection from the input collection, use the for/yield combination:
```
scala> val newArray = for (e <- a) yield {
  e.toUpperCase
}
newArray: Array[java.lang.String] = Array(APPLE, BANANA, ORANGE)
```
The for/yield construct returns a value, so in this case, the array newArray contains uppercase versions of the three strings in the initial array. **Notice that an input Array yields an Array.**

### for loop counters
Scala offers a `zipWithIndex` method to create a loop counter:
```
scala> for ((e, count) <- a.zipWithIndex) {
    |    println(s"$count is $e")
    |  }
0 is apple
1 is banana
2 is orange
```
Note that `zipWithIndex` returns an (item, count) pair.

### Looping over a map
When iterating over keys and values in a Map:
```
val names = Map("fname" -> "Robert",
                "lname" -> "Goren")
for ((k,v) <- names) println(s"key: $k, value: $v")
```

Remember: when you use the `for/yield` combination with a collection, you are building and returning a new collection. When you use a `for` loop without `yield`, you are just operating on each element in the collection.

## 3.2 Using for Loops with Multiple Counters
Such as when iterating over a multi-dimensional array.

You can create a `for` loop with two counters like this:
```
for {
  i <- 0 to 2
  j <- 0 to 2
} println(s"($i)($j) = ${array(i)(j)}")
```
## 3.3 Using a for Loop with Embedded if Statements (Guards)
You want to add one or more conditional clauses to a `for` loop, typically to filter out some elements in a collection while working on the others.

Add an `if` statement after your generator, like this:
```
for (i <- 1 to 10 if i % 2 == 0) println(i)
```
These `if` statements are referred to as filters or *guards*, and you can use as many guards as needed. e.g.
```
for {
  i <- 1 to 10
  if i > 3
  if i < 6
  if i % 2 == 0
} println(i) // Prints 4
```

## 3.4 Creating a for Comprehension (for/yield Combination)
What if: you want to create a new collection from an existing collection by applying an algorithm (and potentially one or more guards) to each element in the original collection.

Use a `yield` statement with a `for` loop (also known as a *for comprehension*) and your algorithm to create a new collection from an existing collection.
```
scala> val capNames = for (e <- names) yield e.capitalize
capNames: Array[String] = Array(Chris, Ed, Maurice)
```

Calling the `map` method on the collection does the same thing.

## 3.5 Implementing break and continue
Scala does not have `break` or `continue` keywords. However, it does offer similar functionality through `scala.util.control.Breaks`.
```
package main

import util.control.Breaks._

object BreakAndContinueDemo extends App {
  println("\n=== BREAK EXAMPLE ===")
  breakable {
    for (i <- 1 to 10) {
      println(i)
      if (i > 4) break // break out of the for loop
    }
  }

  println("\n=== CONTINUE EXAMPLE ===")
  val searchMe = "peter piper picked a peck of pickled peppers"
  var numPs = 0

  for (i <- 0 until searchMe.length) {
    breakable {
      if (searchMe.charAt(i) != 'p') {
        break // break out of the 'breakable', continue the outside loop
      } else {
        numPs += 1
      }
    }
  }
println("Found " + numPs + " p's in the string.")
}
```
Here’s the output from the code:
```
=== BREAK EXAMPLE ===
1
2
3
4
5

=== CONTINUE EXAMPLE ===
Found 9 p's in the string
```

Notice that for `BREAK EXAMPLE`, you have the `for` clause inside the `breakable` clause. For `CONTINUE EXAMPLE`, you have the `breakable` clause inside the `for` clause.
(You can actually use `val count = searchMe.count(_ == 'p')` for the CONTINUE clause).

You can have labeled breaks in case you need a nested break statements.

```
.
.

val Inner = new Breaks
val Outer = new Breaks

Outer.breakable {
  for (i <- 1 to 5) {
    Inner.breakable {
      for (j <- 'a' to 'e') {
        if (i == 1 && j == 'c') Inner.break else println(s"i: $i, j: $j")
        if (i == 2 && j == 'b') Outer.break
      }
    }
  }
}
```
This results in :
```
i: 1, j: a
i: 1, j: b
i: 2, j: a
```

## 3.6 Using the if Construct like a Ternary Operator
There is no special ternary operator in Scala, just the usual `if/else` expression.

## 3.7 Using a Match Expression like a Switch Statement
To use a Scala `match` expression like a Java `switch` statement, use this approach:

```
// i is an integer
(i: @switch) match {
  case 1 => ...
  case 2 => ...
  ...
  // catch the default with a variable so you can print it
  case whoa => println("Unexpected: " + whoa.toString)
  // You can also catch the default with an _  if you are not interested in the value
}
```
When writing match expressions, it is recommended to use the `@switch` annotation. This provides a warning at compile time is the switch can't be compiled to a `tableSwitch` or `lookupSwitch` (for performance reasons).

Sometimes, check if you really need a `switch` statement. Sometimes using a `Map` can also get the job done.

## 3.8 Matching Multiple Conditions with One Case Statement
You can place the match conditions on one line, separated by the | character.
```
i match {
  case 1 | 3 | 5 | 7 | 9 => println("odd")
  case 2 | 4 | 6 | 8 | 10 => println("even")
}
```
This works with strings and other types.

## 3.9 Assigning the Result of a Match Expression to a Variable
Simply insert the variable assignment before a match expression.
```
val evenOrOdd = someNumber match {
  case 1 | 3 | 5 | 7 | 9 => println("odd")
  case 2 | 4 | 6 | 8 | 10 => println("even")
}
```

## 3.10 Accessing the Value of a Default Case in a Match Expression
We have discussed this earlier in Section 3.7. Assign the default a variable.

## 3.11 Using Pattern Matching in Match Expressions
You need to match one or more patterns in a match expression, and the pattern may be a constant pattern, variable pattern, constructor pattern, sequence pattern, tuple pattern, or type pattern.

You can define a `case` statement for each pattern you want to match. For example:
```
def echoWhatYouGaveMe(x: Any): String = x match {
  // constant patterns
  case 0 => "zero"
  case true => "true"
  case "hello" => "you said 'hello'"
  case Nil => "an empty List"

  // sequence patterns
  case List(0, _, _) => "a three-element list with 0 as the first element"
  case List(1, _*) => "a list beginning with 1, having any number of elements"
  case Vector(1, _*) => "a vector starting with 1, having any number of elements"

  // tuples
  case (a, b) => s"got $a and $b"
  case (a, b, c) => s"got $a, $b, and $c"

  // constructor patterns
  case Person(first, "Alexander") => s"found an Alexander, first name = $first"
  case Dog("Suka") => "found a dog named Suka"

  // typed patterns
  case s: String => s"you gave me this string: $s"
  case i: Int => s"thanks for the int: $i"
  case f: Float => s"thanks for the float: $f"
  case a: Array[Int] => s"an array of int: ${a.mkString(",")}"
  case as: Array[String] => s"an array of strings: ${as.mkString
  case d: Dog => s"dog: ${d.name}"
  case list: List[_] => s"thanks for the List: $list"
  case m: Map[_, _] => m.toString

  // the default wildcard pattern
  case _ => "Unknown"
}
```

## 3.12 Using Case Classes in Match Expressions
Let's say that you want to match different case classes (or case objects) in a match expression, such as when receiving messages in an actor.

You can use the different patterns shown in recipe 3.11 to match case classes and objects, depending on your needs.

A case class **can take arguments**, so each instance of that case class can be different based on the values of its arguments.  
A case object on the other hand **does not take args** in the constructor, so there can only be one instance of it (a singleton, like a regular Scala `object` is).

The following example demonstrates how to use patterns to match case classes and case objects in different ways, depending **primarily on what information you need on the right side** of each `case` statement.

In this example, the `Dog` and `Cat` case classes and the `Woodpecker` case object are different subtypes of the `Animal` trait.

```
trait Animal

case class Dog(name: String) extends Animal
case class Cat(name: String) extends Animal
case object Woodpecker extends Animal

object CaseClassTest extends App {

  def determineType(x: Animal): String = x match {
    case Dog(moniker) => "Got a Dog, name = " + moniker
    case _:Cat => "Got a Cat (ignoring the name)"
    case Woodpecker => "That was a Woodpecker"
    case _ => "That was something else"
  }

  println(determineType(new Dog("Rocky")))
  println(determineType(new Cat("Rusty the Cat")))
  println(determineType(Woodpecker))
}
```
The code outputs:
```
Got a Dog, name = Rocky
Got a Cat (ignoring the name)
That was a Woodpecker
```

## 3.13 Adding `if` Expressions (Guards) to Case Statements
You can add an `if` *guard* to your `case` statement. For example:
```
i match {
  case a if 0 to 9 contains a => println("0-9 range: " + a)
  case b if 10 to 19 contains b => println("10-19 range: " + b)
  case c if 20 to 29 contains c => println("20-29 range: " + c)
  case _ => println("Hmmm...")
}
```

You can reference class fields in your `if` guards. You can also extract fields from case classes and use those in your guards.

For example:
```
def speak(p: Person) = p match {
  case Person(name) if name == "Fred" => println("Yubba dubba doo")
  case Person(name) if name == "Bam Bam" => println("Bam bam!")
  case _ => println("Watch the Flintstones!")
}
```

## 3.14 Using a Match Expression Instead of isInstanceOf
You can use the usual way: `if (x.isInstanceOf[Person]) { do something ...}`, but this is discouraged.

A better approach would be:
```
def isPerson(x: Any): Boolean = x match {
  case p: Person => true
  case _ => false
}
```
You may be given an object that extends a known supertype, and then want to take different actions based on the exact subtype.

This is a better method than just having a lot of `if/else` statements.

## 3.15 Working with a List in a Match Expression
A `List` data structure is built from *cons* cells and ends in a `Nil` statement. You want to use this to your advantage when working with a match expression, such as when writing a recursive function.

Firstly, you can create a `List` in two ways:
```
val x = List(1, 2, 3)
// OR
val y = 1 :: 2 :: 3 :: Nil
```
When writing a recursive algorithm, you can take advantage of the fact that the last element in a `List` is a `Nil` object.

For example, in the following `listToString` method, if the current element is not `Nil`, the method is **recursively with the remainder of the `List`**. If the current element is `Nil`, the recursive calls are stopped.
```
def listToString(list: List[String]): String = list match {
  case s :: rest => s + " " + listToString(rest)
  case Nil => ""
}
```
Other examples:
```
def sum(list: List[Int]): Int = list match {
  case Nil => 1
  case n :: rest => n + sum(rest)
}

def multiply(list: List[Int]): Int = list match {
  case Nil => 1
  case n :: rest => n * multiply(rest)
}
```

## 3.16 Matching One / More Exceptions with try/catch
The Scala `try/catch/finally` syntax is similar to Java, but it uses the match expression approach in the catch block:
```
val s = "Foo"
try {
  val i = s.toInt
} catch {
  case e: FileNotFoundException => e.printStackTrace
  case e: IOException => e.printStackTrace
}
```

If you're not concerned about which specific exceptions might be thrown, and watch to catch them all and do something with them (such as logging them), use this syntax:
```
try {
  // something
} catch {
  case t: Throwable => t.printStackTrace()
  // Or ignore them like this:
  case _: Throwable => println("Exception ignored")
}
```
If you prefer to declare the exceptions that your method throws, or if you need to interact with Java, add the `@throws` annotation to your method definition:

```
@throws(classOf[NumberFormatException])
def toInt(s: String): Option[Int] =
  try {
    Some(s.toInt)
  } catch {
    case e: NumberFormatException => throw e
  }
```

## 3.17 Declaring a Variable Before Using It in a try/catch/finally Block
What if you want to use an object in a `try` block, and need to access it in the `finally` portion of the block, such as when you need to call a `close` method on an object?

In general, declare your field as an `Option` before `try/catch` block, then create a `Some` inside the `try` clause. For example:
```
import java.io._

object CopyBytes extends App {

  var in = None: Option[FileInputStream]
  var out = None: Option[FileOutputStream]

  try {
    in = Some(new FileInputStream("/tmp/Test.class"))
    out = Some(new FileOutputStream("/tmp/Test.class.copy"))
    var c = 0
    while ({c = in.get.read; c != −1}) {
      out.get.write(c)
    }
  } catch {
    case e: IOException => e.printStackTrace
  } finally {
    println("entered finally ...")
    if (in.isDefined) in.get.close
    if (out.isDefined) out.get.close
  }

}
```
In this code, `in` and `out` are assigned to `None` before the `try` clause, and then reassigned to `Some` values inside the `try` clause if everything succeeds. Therefore, it is safe to call `in.get` and `out.get` in the `while` loop, because if an exception had occurred, flow control would have switched to the `catch` clause.

One key to this recipe is knowing the syntax for declaring `Option` fields that are not initially populated:
```
var in = None: Option[FileInputStream]
var out = None: Option[FileOutputStream]
```

## 3.18 Creating Your Own Control Structures
Let's say you want to create your own `whilst` loop, which you can use like this:
```
package foo

import com.alvinalexander.controls.Whilst._

object WhilstDemo extends App {
  var i = 0
  whilst (i < 5) {
    println(i)
    i += 1
  }
}
```
To create your own `whilst` control structure, define a function named `whilst` that takes two parameter lists. The first parameter list handles the test condition (`i < 5`) - and the second parameter list is the block of code the user wants to run.  
```
object Whilst {
  @tailrec
  def whilst(testCondition: => Boolean)(codeBlock: => Unit) {
    if (testCondition) {
      codeBlock
      whilst(testCondition)(codeBlock)
    }
  }
}
```
In this code, the `testCondition` is evaluated once, and if the condition is `true`, the `codeBlock` is executed, and then `whilst` is called recursively. This approach lets you keep checking the condition without needing a `while` or `for` loop.

In a simpler example, you don't need recursion.

For example, assume you want a control structure that takes two test conditions, and if both evaluate to true, you’ll run a block of code that’s supplied. An expression using that control structure might look like this:
```
doubleif(age > 18)(numAccidents == 0) { println("Discount!") }
```

In this case, define a function that takes three parameter lists:
```
// two 'if' condition tests
def doubleif(test1: => Boolean)(test2: => Boolean)(codeBlock: => Unit) {
  if (test1 && test2) {
    codeBlock
  }
}
```
