# Chapter 9 - Functional Programming

Scala is both an OOP and a functional programming language. This chapter demonstrates functional programming techniques, including the ability to define functions and pass them around as instances.

Scala encourages an expression-oriented programming model (EOP). In EOP, every statement (expression) yields a value. This paradigm can be as obvious as an `if/else` statement returning a value, or as surprising as a `try/catch` statement returning a value.

## 9.1 Using Function Literals (Anonymous Functions)
You can use an anonymous function (a.k.a. a *function literal*) and you can pass it into a method that takes a function, or to assign it to a variable.

Given this `List`: `val x = List.range(1, 10)`
you can pass an anonymous function to the `List`'s `filter` method to create a new `List` that contains only even numbers:

```
val evens = x.filter((i: Int) => i % 2 == 0)
```

This part here is a function literal: `(i: Int) => i % 2 == 0`

The expression can be simplified to this:
```
val evens = x.filter(_ % 2 == 0)
```

It helps to think of the => symbol as a `transformer`, because the expression transforms the **parameter list** on the **left** side of the symbol into a new result using the **algorithm** on the **right** side of the symbol.

In other examples, you can simplify your anonymous functions further. For instance, beginning with the most explicit form, you can print each element in the list using this anonymous function with the foreach method:
```
x.foreach((i:Int) => println(i))
```
As before, the Int declaration isn’t required:
```
x.foreach((i) => println(i))
```
Because there is only one argument, the parentheses around the i parameter aren’t needed:
```
x.foreach(i => println(i))
```
Because i is used only once in the body of the function, the expression can be further simplified with the _ wildcard:
```
x.foreach(println(_))
```
Finally, if a function literal consists of one statement that takes a single argument, you need not explicitly name and specify the argument, so the statement can finally be reduced to this:
```
x.foreach(println)
```

## 9.2 Using Functions as Variables
What if you want to pass a function around like a variable?

You can use the syntax shown in 9.1 to define a function literal, and then assign that literal to a variable.

For example:
```
val double = (i: Int) => { i * 2 }
```

The variable `double` is an instance. In this case, it's an instance of a function, also known as a *function value*. You can now invoke `double` just like you'd call a method.

```
double(2)     // 4
double(3)     // 6
```
You can also pass it to any method (or function) that takes a function parameter with its signature. For instance:
```
val list = List.range(1, 5)
list.map(double)
// List(2, 4, 6, 8)
```
There are two ways of declaring a function literal:

```
// Implicit approach
val add = (x: Int, y: Int) => { x + y }
val add = (x: Int, y: Int) => x + y

// explicit approach
val add: (Int, Int) => Int = (x, y) => { x + y }
val add: (Int, Int) => Int = (x, y) => x + y
```

### Using a method like an anonymous function
You can define an anonymous function and assign it to a variable. You can also define a method and then pass it around like an instance variable.

For instance, you can define a method in any of these ways:
```
def modMethod(i: Int) = i % 2 == 0
def modMethod(i: Int) = { i % 2 == 0 }
def modMethod(i: Int): Boolean = i % 2 == 0
def modMethod(i: Int): Boolean = { i % 2 == 0 }
```

Any of these methods can be passed into collection method that expect a function that has one `Int` parameter and returns a `Boolean`, such as the `filter` method of a `List[Int]`.

```
val list = List.range(1, 10)
list.filter(modMethod)
```

This is a function that works just like the previous method:
```
val modFunction = (i: Int) => i % 2 = 0
list.filter(modFunction)
```

At a coding level, the obvious difference is that `modMethod` is a *method* defined in a class, whereas `modFunction` is a *function* that’s assigned to a variable. Under the covers, `modFunction` is an instance of the Function1 trait, which defines a function that takes one argument.

### Assigning an existing function/method to a function variable
You can assign an existing method or function to a function variable. For instance, you can create a function named `c` from the `scala.math.cos` method:
```
val c = scala.math.cos(_)
```
This is called a *partially applied function*. It is partially applied because the `cos` method requires one argument, which you have not yet supplied.

Now you can use `c` like you would have used `cos`:
```
c(0)  // Double = 1.0
```
We will take a look at this deeper at **9.6**.

Summary:
- Think of the `=>` symbol as a transformer. It transforms the input data from the left side to a new output data using the algorithm on the right side.
- Use `def` to define a method. Use `val` to create a function.
- When assigning a function to a variable, a *function literal* is the code on the right side of the expression.
- A *function value* is an object, and extends the `FunctionN` traits in the main `scala` package, such as `Function0` for a function that takes no parameters.

### Intermezzo: What is the difference between a function and a method?
- To define a method, you use `def`, while to create a function, you use `val`.
- `def` evaluates every time it gets called while `val` evaluates only once.
A Scala method, as in Java, is a part of a class. It has a name, a signature, optionally some annotations, and some bytecode.

A function in Scala is a complete object. There are a series of traits in Scala to represent functions with various numbers of arguments: Function0, Function1, Function2, etc. As an instance of a class that implements one of these traits, a function object has methods. One of these methods is the apply method, which contains the code that implements the body of the function. Scala has special "apply" syntax: if you write a symbol name followed by an argument list in parentheses (or just a pair of parentheses for an empty argument list), Scala converts that into a call to the apply method for the named object. When we create a variable whose value is a function object and we then reference that variable followed by parentheses, that gets converted into a call to the apply method of the function object.

When we treat a method as a function, such as by assigning it to a variable, Scala actually creates a function object whose apply method calls the original method, and that is the object that gets assigned to the variable. Defining a function object and assigning it to an instance variable this way consumes more memory than just defining the functionally equivalent method because of the additional instance variable and the overhead of another object instance for the function. Thus you would not want every method to be a function; but functions give you a great deal of power that is not available with just methods, and in those situations that power is well worth the additional memory used.

[Read more](http://jim-mcbeath.blogspot.com/2009/05/scala-functions-vs-methods.html)

## 9.3 Defining a Method That Accepts a Simple Function Parameter
1. Define your method, including the signature for the function you want to take as a method parameter.
2. Define one or more functions that match this signature.
3. Pass the function(s) as a parameter to your method.

Let's use an example. First, define a method named `executeFunction`, which will take a function, named `callback` as a parameter. The function has no input parameters and returns nothing.

```
def executeFunction(callback:() => Unit) {
  callback()
}
```
Two things to take note:
- The `callback:()` syntax defines a function that has no parameters.
- The `=> Unit` portion of the code indicates that this method returns nothing.

Next, define a function that matches this signature.

The following function named `sayHello` takes no input parameters and returns nothing.

```
val sayHello = () => { println("Hello") }
```

Lastly, pass the function to the `executeFunction` method:
```
executeFunction(sayHello)
// prints "Hello"
```
The general syntax for defining a function as a method parameter is:
```
parameterName: (parameterType(s)) => returnType
```

## 9.4 More Complex Functions
Now, let's say you want to define a method that takes a function as a parameter, and that function may have one or more input parameters, and may also return a value.

Similar to the solution for 9.3, follow the 3 steps above.

1. The following example defines a method named `exec` that takes a function as an input parameter. That function must take one `Int` as an input parameter and return nothing:

```
def exec(callback: Int => Unit) {
  callback(1)
}
```
2. Define a function that matches the expected signature (takes an `Int` and outputs nothing (`Unit`)):
```
val plusOne = (i: Int) => { println(i+1) }
```
3. Pass the function to the method
```
exec(plusOne)
```
You can pass in other functions to the method, as long as your function signature matches what your method expects, your algorithms can do anything you want.

Remember that aside from accepting the function as a parameter, the method can also accept other parameters.

## 9.5 Using Closures
Let's say you want to pass a function around like a variable, and while doing so, you want that function to be able to refer to one or more fields that were in the same scope as the function when it was declared.

We can use closures to do this. This is a demonstration of closures in Scala:
```
package otherscope {
  class Foo {
    // a method that takes a function and a string, and passes the string into
    // the function, and then executes the function
    def exec(f:(String) => Unit, name: String) {
      f(name)
    }
  }
}

object ClosureExample extends App {
  var hello = "Hello"
  def sayHello(name: String) { println(s"$hello, $name") }

  // execute sayHello from the exec method foo
  val foo = new otherscope.Foo
  foo.exec(sayHello, "Al")

  // change the local variable 'hello', then execute sayHello from
  // the exec method of foo, and see what happens
  hello = "Hola"
  foo.exec(sayHello, "Lorenzo")
}
```
When this code is run, the output will be:
```
Hello, Al
Hola, Lorenzo
```

On the first run, the `sayHello` method references the variable `hello` from within the `exec` method of the `Foo` class  (where `hello` was no longer in scope).

On the second run, it also picked up the change to the `hello` variable (to `Hola`).  How is this possible?

The answer is that Scala supports closure functionality, and this is how closures work.

There are two *free variables* in the `sayHello` method: `name` and `hello`. The `name` variable is a formal parameter to the function; this is something you're used to.

However, `hello` is not a formal parameter; it is a reference to a variable in the enclosing scope (similar to how a method in a Java class can refer to a field in the same class). Hence, the Scala compiler creates a closure that encompasses `hello`.

## 9.6 Using Partially Applied Functions
You want to eliminate repetitively passing variables into a function by (a) passing common variables into the function to (b) create a new function that is preloaded with those values, and then (c) use the new function, passing it only the unique variables it needs.

Let's use an example: a simple `sum` function:
```
val sum = (a: Int, b: Int, c: Int) => a + b + c
```
What happens if you call the function without providing the third parameter?
```
val f = sum(1, 2, _: Int)
```
The resulting variable `f` is a *partially applied function*. `f` is now a function that implements the `function1` trait, meaning that it takes one argument. As such, when you give `f` an int, i.e. the number 3, you will get the sum of the three numbers.
```
scala> f(3)
res0: Int = 6
```
This partially applied function is a variable that you can pass around. This variable is called a *function value*, and when you provide all the parameters needed to complete the function value, the original function is executed and a result is yielded.

## 9.7 Creating a Function That Returns a Function
1. Define a function that returns an algorithm (an anonymous function)
2. Assign that to a new function
3. and then call that new function

This declares an anonymous function that takes a `String` argument and returns a `String`:
```
(s: String) => { prefix + " " + s }
```
You can return that anonymous function from the body of another function like this:
```
def saySomething(prefix: String) = (s: String) => {
  prefix + " " + s
}
```
Since `saySomething` returns a function, you can assign that resulting function to a variable.
```
val sayHello = saySomething("Hello")
```

The `sayHello` function is now equivalent to your anonymous function, with the `prefix` set to `hello`. Now you can call `sayHello` with a `String` parameter:
```
scala> sayHello("Al")
res0: java.lang.String = Hello Al
```
You can use this approach any time you want to encapsulate an algorithm inside a function. Similar to Factory or Strategy pattern, the function your method returns can be based on the input parameter it receives.

## 9.8 Creating Partial Functions
Let's say you want to define a function that will only work for a subset of possible input values, or you want to define a series of functions that only work for a subset of input values, and combine those functions to completely solve a problem.

You can use a *partial function*. A partial function is a function that does not provide an answer for every possible input value it can be given. It provides an answer only for a subset of possible data, and defines the data it can handle.

As an example, imagine a normal function that divides one number by another:
```
val divide = (x: Int) => 42 / x
```
When the input parameter is zero, this will throw an ArithmeticException, division by zero. Although you can handle this by catching and throwing an exception, Scala lets you define the `divide` function as a `PartialFunction`.
```
val divide = new PartialFunction[Int, Int] {
  def apply(x: Int) = 42 / x
  def isDefinedAt(x: Int) = x != 0
}
```
Using this, you can test the function before attempting to use it:
```
scala> divide.isDefinedAt(1)
res0: Boolean = true

scala> if (divide.isDefinedAt(1)) divide(1)
res1: AnyVal = 42

scala> divide.isDefinedAt(0)
res2: Boolean = false
```
While that `divide` function is explicit about what data it handles, partial functions are often written using `case` statements:
```
val divide2: PartialFunction[Int, Int] = {
  case d: Int if d != 0 => 42 / d
}
```
This code doesn't explicitly implement the `isDefinedAt` method, but it works exactly the same as the previous `divide` function definition.

You can chain partial functions together. For instance, two functions are defined that can each handle a small number of `Int` inputs, and convert them to `String` results:
```
// converts 1 to "one", etc., up to 5
val convert1to5 = new PartialFunction[Int, String] {
  val nums = Array("one", "two", "three", "four", "five")
  def apply(i: Int) = nums(i-1)
  def isDefinedAt(i: Int) = i > 0 && i < 6
}

// converts 6 to "six", etc., up to 10
val convert6to10 = new PartialFunction[Int, String] {
  val nums = Array("six", "seven", "eight", "nine", "ten")
  def apply(i: Int) = nums(i-6)
  def isDefinedAt(i: Int) = i > 5 && i < 11
}
```
You can combine them like this:
```
scala> val handle1to10 = convert1to5 orElse convert6to10
handle1to10: PartialFunction[Int,String] = <function1>

scala> handle1to10(3)
res0: String = three

scala> handle1to10(8)
res1: String = eight
```
The `orElse` method comes from the Scala `PartialFunction` trait, which also includes the `andThen` method to further help chain partial functions together.

## 9.9 A Real-World Example
To demonstrate some of the techniques introduced in this chapter, the following example shows one way to implement Newton's Method.

As you can see from the code, the method named `newtonsMethod` takes functions as its first two parameters. It also takes two other `Double` parameters, and returns a `Double`. The two functions that are passed in should be the original equation (`fx`) and the derivative of that equation (`fxPrime`).

The method `newtonsMethodHelper` also takes two functions as parameters, so you can see how the functions are passed from `newtonsMethod` to `newtonsMethodHelper`.

Here is the complete source code for this example:
```
object NewtonsMethod {
  def main(args: Array[String]) {
    driver
  }

  /**
  * A "driver" function to test Newton's method.
  * Start with (a) the desired f(x) and f'(x) equations,
  * (b) an initial guess and (c) tolerance values.
  */
  def driver {
    // the f(x) and f'(x) functions
    val fx = (x: Double) => 3*x + math.sin(x) - math.pow(math.E, x)
    val fxPrime = (x: Double) => 3 + math.cos(x) - math.pow(Math.E, x)

    val initialGuess = 0.0
    val tolerance = 0.00005

    // pass f(x) and f'(x) to the Newton's Method function, along with
    // the initial guess and tolerance
    val answer = newtonsMethod(fx, fxPrime, initialGuess, tolerance)

    println(answer)
  }

  /**
  * Newton's Method for solving equations.
  * @todo check that |f(xNext)| is greater than a second tolerance value
  * @todo check that f'(x) != 0
  */
  def newtonsMethod(fx: Double => Double,
                    fxPrime: Double => Double,
                    x: Double,
                    tolerance: Double): Double = {
    var x1 = x
    var xNext = newtonsMethodHelper(fx, fxPrime, x1)
    while (math.abs(xNext - x1) > tolerance) {
      x1 = xNext
      println(xNext) // debugging (intermediate values)
      xNext = newtonsMethodHelper(fx, fxPrime, x1)
    }

    xNext
  }   

  /**
  * This is the "x2 = x1 - f(x1)/f'(x1)" calculation
  */
  def newtonsMethodHelper(fx: Double => Double,
                          fxPrime: Double => Double,
                          x: Double): Double = {
    x - fx(x) / fxPrime(x)
  }
}
```
As you can see, a majority of this code involves defining functions, passing those functions to methods, and then invoking the functions from within a method.

The method name `newtonsMethod` will work for any two functions `fx` and `fxPrime`, where `fxPrime` is the derivative of `fx` (within the limits of the “to do” items that are not implemented).

To experiment with this example, try changing the functions `fx` and `fxPrime`, or implement the `@todo` items in `newtonsMethod`.
