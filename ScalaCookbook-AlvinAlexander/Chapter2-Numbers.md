# Chapter 2 - Numbers

In Scala, all the numeric types are objects, including `Byte, Char, Double, Float, Int,
Long, and Short`. They have the same data ranges as their Java primitive equivalents. IF you need to know the exact values of the data ranges, you can use the `MinValue` and `MaxValue` methods. e.g. `Short.MinValue -> -32768` `Short.MaxValue -> 32767`

If you need more powerful math classes, check out the **Spire project** and **ScalaLab**.

For processing dates, the Java **Joda Time project** is popular and well documented. A
project named **nscala-time** implements a Scala wrapper around Joda Time, and lets you
write date expressions in a more Scala-like way.

## 2.1. Parsing a Number from a String (Convert String to Number) [COMMON]
You can use the `to*` methods that are available on a String (thanks to the `StringLike` trait).

```
scala> "100".toInt
res0: Int = 100

scala> "100".toDouble
res1: Double = 100.0
```
These methods can throw the usual Java `NumberFormatException`.

BigInt and BigDecimal instances can also be created directly from strings (and can also throw a NumberFormatException):
```
scala> val b = BigInt("1")
b: scala.math.BigInt = 1

scala> val b = BigDecimal("3.14159")
b: scala.math.BigDecimal = 3.14159
```

If you need to perform calculations using bases other than 10, use the `parseInt` method instead of the `toInt` method. You can also create an implicit class and method to help solve the problem, as explained in **1.10**.

Scala does **not** have checked exceptions, so there are different ways of handling possible `NumberFormatException`.

For example, adding a Scaladoc comment, or mark your method with the `@throws` annotation.
```
// Not required to declare "throws NumberFormatException"
def toInt(s: String) = s.toInt

@throws(classOf[NumberFormatException])
def toInt(s: String) = s.toInt
```
If the method will be called from Java code, you **need** to annotate it.

In Scala, situations like this are often handled with the `Option/Some/None` pattern.
```
def toInt(s: String):Option[Int] = {
  try {
    Some(s.toInt)
  } catch {
    case e: NumberFormatException => None
  }
}
```
Then you can call the `toInt` method in several different ways, for example: with `getOrElse`:
```
println(toInt("1").getOrElse(0)) // 1
println(toInt("a").getOrElse(0)) // 0
```
or to use a match expression:
```
val result = toInt(aString) match {
  case Some(x) => x
  case None => 0 // however you want to handle this
}
```

## 2.2 Converting Between Numeric Types (Casting) [COMMON]
The `to*` methods are also available on all numeric types. If you want to avoid potential conversion errors when casting from one numeric type to another, you can use the `isValid*` method prior to conversion.
```
scala> val a = 1000L
a: Long = 1000

scala> a.isValidByte
res0: Boolean = false

scala> a.isValidShort
res1: Boolean = true
```

## 2.3 Overriding the Default Numeric Type
Scala automatically assigns types to numeric values when you assign them. What if you need to override the default type it assigns as you create a numeric field?

If you assign 1 to a variable, Scala assigns it the type Int:
```
scala> val a = 1
a: Int = 1
```
The following examples show one way to override simple numeric types:
```
scala> val a = 1d
a: Double = 1.0

scala> val a = 1f
a: Float = 1.0

scala> val a = 1000L
a: Long = 1000
```
Another approach is to annotate the variable with a type, like this:
```
scala> val a: Byte = 0:
a: Byte = 0

scala> val a: Int = 0:
a: Int = 0

scala> val a: Short = 0:
a: Short = 0
```

## 2.4 Replacements for ++ and -- [COMMON]
Because `val` fields are immutable, they can't be incremented or decremented, but var `Int` fields can be mutated with the `+=` and `-=` methods (or the `*=` and `/=` methods).

Note that these symbols aren't operators; they are implemented as methods that are available on `Int` fields declared as a `var`. Attempting to use them on `val` fields results in a compile time error.

So, use `var` , not `val`.

## 2.5 Comparing Floating-Point Numbers.
You need to compare two floating-point numbers, but as in some other programming languages, two floating-point numbers that *should* be equivalent may not be.

As in Java and other languages, you solve this problem by creating a method that lets you specify the precision for your comparison.

```
def ~=(x: Double, y: Double, precision: Double) = {
  if ((x - y).abs < precision) true else false
}
```
You can define an implicit conversion to add a method like
this to the Double class. This makes the following code very readable:
```
if (a ~= b) ...
```
Or, you can add the same method to a utilities object, if you prefer:
```
object MathUtils {
  def ~=(x: Double, y: Double, precision: Double) = {
    if ((x - y).abs < precision) true else false
  }
}
```

## 2.6 Handling Very Large Numbers
Use the Scala `BigInt` and `BigDecimal` classes. Unlike in Java, these classes support all the operators that you can use with numeric types. They are also mutable.

## 2.7 Generating Random Numbers
You can use the `scala.util.Random` class.
```
scala> val r = scala.util.Random
r: scala.util.Random = scala.util.Random@13eb41e5

scala> r.nextInt
res0: Int = −1323477914
```
You can limit the random numbers to a maximum value:
```
scala> r.nextInt(100)
res1: Int = 58
```
Similarly, you can create random `Float` values using `r.nextFloat`, and random `Double` values with `r.nextDouble`, or even random characters using `r.nextPrintableChar`. You can also set seed values using `r.setSeed(seed)`.

## 2.8 Creating a Range, List, or Array of Numbers [COMMON]
You can use the `to` method to create a `Range` with the desired elements:
```
scala> val r = 1 to 10
r: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5,
    6, 7, 8, 9, 10)
```
You can set the step with the `by` method.
```
scala> val r = 1 to 10 by 2
r: scala.collection.immutable.Range = Range(1, 3, 5, 7, 9)
```

## 2.9 Formatting Numbers and Currency
What if you want to format numbers/currency to control decimal places and commas?

We can use the `f` string interpolator, for a start.

```
scala> val pi = scala.math.Pi
pi: Double = 3.141592653589793

scala> println(f"$pi%1.5f")
3.14159
```
To add commas, use the `getIntegerInstance` method of the `java.text.NumberFormat` class:
```
scala> val formatter = java.text.NumberFormat.getIntegerInstance
formatter: java.text.NumberFormat = java.text.DecimalFormat@674dc

scala> formatter.format(10000)
res0: String = 10,000

scala> formatter.format(1000000)
res1: String = 1,000,000
```
You can also set a locale with the `getIntegerInstance` method.

If your value is not an integer, use `getInstance` instead.

For currency, use `getCurrencyInstance` formatter. Similarly, you can also set a locale with this method.

---
