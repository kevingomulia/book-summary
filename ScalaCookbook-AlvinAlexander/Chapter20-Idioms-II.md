# Chapter 20 - Idioms - Part II

## 20.5 Eliminate null Values from Your Code
There are several alternatives to using `null` values

### Initialise var fields with Option, not null
Possibly the most tempting time to use a `null` value is when a field in a class or method won't be initialised immediately. For example. this is not nice because `firstName`, `lastName`, and `address` are all declared to be `null`, and can cause problems in your application if they're not assigned before they're accessed:
```
case class Address (city: String, state: String, zip: String)

class User(email: String, password: String) {
  var firstName: String = _
  var lastName: String = _
  var address: Address = _
}
```
This is a better approach:
```
case class Address (city: String, state: String, zip: String)

class User(email: String, password: String) {
  var firstName = None: Option[String]
  var lastName = None: Option[String]
  var address = None: Option[Address]
}
```
Later on, you can access the fields like this:
```
println(firstName.getOrElse("<Not Assigned>"))
```
On a related note, you should also use an `Option` in a constructor when a field is optional.

### Don't return null from methods
Instead of returning `null`, return an `Option`. Or if you need to know about an error that may have occurred in the method, use `Try` instead of `Option`.

With an `Option`, your method signature should look like this:
```
def doSomething: Option[String] = { ... }
def toInt(s: String): Option[Int] = { ... }
def lookupPerson(name: String): Option[Person] = { ... }
```
For instance, when reading a file, a method could return `null` if the process fails, but this code shows how to read a file and return an `Option` instead:
```
def readTextFile(filename: String): Option[List[String]] = {
  try {
    Some(io.Source.fromFile(filename).getLines.toList)
  } catch {
    case e: Exception => None
  }
}
```

### Converting a null into an Option, or something else
For instance, this method converts a result from a Java method that may be `null` and returns an `Option[String]` instead.
```
def getName: Option[String] = {
  var name = javaPerson.getName
  if (name == null) None else Some(name)
}
```

### Benefits
- You'll eliminate `NullPointerExceptions`
- Your code will be safer
- You won't have to write `if` statements to check for `null` values.
- Adding an `Option[T]` return type declaration to a method is a terrific way to indicate that something is happening in the method such that the caller may receive a `None` instead of a `Some[T]`.

## 20.6 Using the Option/Some/None Pattern
If you want to remove `null` values from your code, use the `Option/Some/None` pattern. If you're interested in the problem that occurred while processing code, use the `Try/Success/Failure` method instead.

### Returning an Option from a method
See the `readTextFile` example in **Recipe 20.5**.

### Getting the value form an Option
The `toInt` example shows how to declare a method that returns an `Option`. You can use:
- `getOrElse`
- `foreach`
- Use a match expression
```
scala> val x = toInt("1").getOrElse(0)
x: Int = 1

toInt("1").foreach{ i =>
  println(s"Got an int: $i")
}

toInt("1") match {
  case Some(i) => println(i)
  case None => println("That didn't work.")
}
```

### Using Option with Scala collections
Imagine you have a list of strings:
```
val bag = List("1", "2", "foo", "3", "bar")
```
And you want to list of all the integers that can be converted from that list of strings.
```
scala > bag.map(toInt)
res0: List[Option[Int]] = List(Some(1), Some(2), None, Some(3), None)

scala> bag.map(toInt).flatten
res1: List[Int] = List(1, 2, 3)

// This is the same as calling `flatMap`
scala> bag.flatMap(toInt)
res2: List[Int] = List(1, 2, 3)

// The `collect` method provides another way to achieve the same result
scala> bag.map(toInt).collect{case Some(i) => i}
res3: List[Int] = List(1, 2, 3)
```

### Using Option with other frameworks
Other frameworks use `Option` to handle situations where a variable may not have a value.

If you like the `Option/Some/None` approach, but want to write a method that returns error information in the failure case (instead of `None`, which doesn't return any error information).

There are two similar approaches:
- `Try`, `Success`, and `Failure`
- `Either`, `Left`, `Right`

### Using Try, Success, and Failure
Scala 2.10 introduced `scala.util.Try` as an approach that's similar to `Option`, but returns failure information rather than a `None`.

Let's use an example here:
```
def divideXByY(x: Int, y: Int): Try[Int] = {
  Try(x / y)
}
```
This method returns a successful result as long as y != 0.
```
scala> divideXByY(1,1)
res0: scala.util.Try[Int] = Success(1)

scala> divideXByY(1,0)
res1: scala.util.Try[Int] = Failure(java.lang.ArithmeticException: / by zero)
```
As with an `Option`, you can access the `Try` result using `getOrElse`, a `foreach` method, or a match expression. If you don't care about the error message, use `getOrElse`:
```
// Success
scala> val x = divideXByY(1, 1).getOrElse(0)
x: Int = 1

// Failure
scala> val y = divideXByY(1, 0).getOrElse(0)
y: Int = 0
```
If you're interested in the `Failure` message, you can use a match expression:
```
divideXByY(1, 1) match {
  case Success(i) => println(s"Success, value is: $i")
  case Failure(s) => println(s"Failed, message is: $s")
}
```
The `Try` class has the added benefit that you can chain operations together, catching exceptions as you go.

The `Try` class includes a nice collection of methods that let you handle situations in many ways, including:
- Collection-like implementations of `filter`, `flatMap`, `flatten`, `foreach`, and `map`
- `get`, `getOrElse`, and `orElse`
- `toOption`, which lets you treat the result as an Option
- `recover`, `recoverWith`, and `transform`, which let you gracefully handle `Success` and `Failure` results

### Use Either, Left, Right
```
def divideXByY(x: Int, y: Int): Either[String, Int] = {
  if (y == 0) Left("Dude, can't divide by 0")
  else Right(x / y)
}
```
As shown, your method should be declared to return an `Either`, and the method body should return a `Right` on success and ` Left` on failure. The `Right` type is the type your method returns when it runs successfully, and the `Left` type is typically a `String` (error message).

As with `Option` and `Try`, a method returning an `Either` can be called in a variety of ways, including `getOrElse` or a match expression:
```
val x = divideXByY(1, 1).right.getOrElse(0) // returns 1
val x = divideXByY(1, 0).right.getOrElse(0) // returns 0

// prints "Answer: Dude, can't divide by 0"
divideXByY(1, 0) match {
  case Left(s) => println("Answer: " + s)
  case Right(i) => println("Answer: " + i)
}
```
