# Chapter 1 - Strings

A Scala String is a Java String, so you can use all the normal Java string methods.

While the String class is declared as `final` in Java, you can add your own methods to the String class in Scala.

A Scala String have **both string and collection features**. The following code uses the `drop` and `take` methods that are available on Scala sequences, along with the `capitalize` method from the StringOps class.

```
scala> "scala".drop(2).take(2).capitalize // Chaining methods is called fluent programming style
res0: String = Al
```

## 1.1 Testing String Equality
You can test if two String instances contain the same sequence of characters by using the `==` operator. This `==` method does not throw a `NullPointerException` if a String is `null`. However, calling a method on a `null` string can throw a `NullPointerException`.

In Scala, you test object equality with the `==` method. In Java, you use the `equals` method to compare two objects.

In Scala, the `==` method defined in the `AnyRef` class first checks for `null` values, and then calls the `equals` method on the first object (i.e., `this`) to see if the two objects are equal. As a result, you don’t have to check for `null` values when comparing strings.

**NOTE:** In Scala, use an `Option` instead of using a `null` value (Scala does not have a `null` keyword)

## 1.2 Creating Multiline Strings
You can use three double quotes : `"""` to envelope the multiline strings. However, the lines will end up with whitespace at the beginning of their lines. You can add a `stripMargin` method to the end of the multiline string and begin all lines after the first line with the pipe symbol (`|`).

```
val speech = """Four score and
|seven years ago""".stripMargin  
```
This results in a true multiline string, with a hidden `\n` character after the word “and” in the first line.

## 1.3 Splitting Strings
You can use the `split` method that are available on `String` objects (like in Java).
```
scala> "hello world".split(" ")
res0: Array[java.lang.String] = Array(hello, world)
```
It is best to trim each string after splitting them. Use the `map` method to call `trim` on each string. `s.split(",").map(_.trim)`

To split a string on whitespace characters: `s.split("\\s+")`

## 1.4 Substituting Variables into Strings
To use basic string interpolation in Scala, precede your string with the letter s and include your variables inside the string, with each variable name preceded by a $ character
```
scala> val name = "Fred"
name: String = Fred

scala> val age = 33
age: Int = 33

scala> val weight = 200.00
weight: Double = 200.0

scala> println(s"$name is $age years old, and weighs $weight pounds.")
Fred is 33 years old, and weighs 200.0 pounds.
```

Prepending `s` to any string literal allows the usage of variables directly in the string.

In addition to putting variables inside strings, you can include expressions inside a string by placing the expression inside curly braces. e.g.

```
scala> println(s"Age next year: ${age + 1}")
Age next year: 34
```
You can also use curly braces to print object fields.
```
scala> println(s"${hannah.name} has a score of ${hannah.score}")
Hannah has a score of 95
```
The `s` that's placed before each string literal is actually a method.
### The f string interpolator (printf style formatting)
Using the "f string interpolator" allows you to use `printf` style formatting specifiers inside strings.

You can use string interpolation in ways other than just `println` methods. For example:
```
scala> val out = f"$name, you weigh $weight%.0f pounds."
out: String = Fred, you weigh 200 pounds.
```
### The raw interpolator
The raw interpolator **performs no escaping of literals within the string.**

Example:
```
scala> s"foo\nbar"
res0: String =
foo
bar

scala> raw"foo\nbar"
res1: String = foo\nbar
```
This is particularly useful when you want to avoid having a sequence of characters like \n turn into a newline character.

Common printf style format specifiers

|Format specifier|Description|
|---|---|
|%c | Character |
|%d | Decimal number (integer, base 10) |
|%e | Exponential floating-point number |
|%f | Floating-point number |
|%i | Integer (base 10) |
|%o | Octal number (base 8) |
|%s | A string of characters |
|%u | Unsigned decimal (integer) number |
|%x | Hexadecimal number (base 16) |
|%% | Print a "percent" character |
|\% | Print a "percent" character |

For more `printf` cheat sheet: [click here](http://alvinalexander.com/programming/printf-format-cheat-sheet)

## 1.5 Processing a String One Character at a Time
Mostly, you can use the `map` or `foreach` methods, or a `for` loop. Here you can see what the Scala underscore character is useful for:

```
scala> val upper = "hello, world".map(c => c.toUpper)
upper: String = HELLO, WORLD

scala> val upper = "hello, world".map(_.toUpper)
upper: String = HELLO, WORLD
```

## 1.6 Finding Patterns in Strings
Create a `Regex` object by invoking the `.r` method on a String, and then use that pattern with `findFirstIn` when  you're looking for one match, and `findAllIn` when looking for all matches.

Example:

Firstly create a Regex for the pattern you want to search for.
```
scala> val numPattern = "[0-9]+".r
numPattern: scala.util.matching.Regex = [0-9]+
```
Next, create a sample String you can search:
```
scala> val address = "123 Main Street Suite 101"
address: java.lang.String = 123 Main Street Suite 101
```
The findFirstIn method finds the first match:
```
scala> val match1 = numPattern.findFirstIn(address)
match1: Option[String] = Some(123)
```
