# Chapter 5 - Methods

Scala methods and Java methods differ significantly in their implementation details.
- Specifying method access control (visibility)
- The ability to set default values for method parameters
- The ability to specify the names of method parameters when calling a method
- How you declare the exceptions a method can throw
- Using varargs fields in methods

## 5.1 Controlling Method Scope
Scala methods are public by default. But what if you want to control their scope in ways similar to Java?

Scala lets you control method visibility in a more granular and powerful way than Java. Scala provides these scope options:
- Object-private scope
- Private
- Package
- Package-specific
- Public

### Object-private scope
This is the most restrictive access to mark a method. When you mark a method object-private, the method is only available to the **current instance** of the current object. **Other instances of the same class cannot access the method.**

You mark a method as object-private by placing the access modifier `private[this]` before the method declaration:
```
class Foo {

  private[this] def isFoo = true
  def doFoo(other: Foo) {
      if (other.isFoo) { // this line won't compile
    }
  }
}
```
This won't compile because the current `Foo` instance can't access the `isFoo` method of the `other` instance, because `isFoo` is declared as `private[this]`.

### Private scope
A `private` method is available to both the current class and other instances of the current class. A `private` method is not available to its subclasses. This is similar as marking a method `private` in Java.

By changing the access modifier from `private[this]` to `private`, the code above will compile.
```
class Animal {
  private def heartBeat {}
}

class Dog extends Animal {
  heartBeat // won't compile
}
```

### Protected scope
Marking a method `protected` makes the method available to its subclasses.
```
class Animal {
  protected def breathe {}
}

class Dog extends Animal {
  breathe // This compiles
}
```

The meaning of `protected` is slightly different in Scala than in Java. In Java, `protected` methods can be accessed by other classes in the same package, but this isn't true in Scala.

```
package world {

  class Animal {
    protected def breathe {}
  }

  class Jungle {
    val a = new Animal
    a.breathe       // Error: this line won't compile
  }
}
```

### Package scope
To make a method available to all members of the current package, mark the method as being private to the current package with the `private[packageName]` syntax.
```
package com.acme.coolapp.model {

  class Foo {
    private[model] def doX {}
    private def doY {}
  }

  class Bar {
    val f = new Foo
    f.doX // compiles
    f.doY // won't compile
  }
}
```
You can make methods available to other packages aside from the current package using similar syntax as above.

### Public scope
If no access modifier is added to the method declaration, the method is public.

In summary,

| Access Modifier | Description |
| --- | --- |
|private[this] | The method is available only to the current instance of the class it’s declared in. |
|private | The method is available to the current instance and other instances of the class it’s declared in. |
|protected | The method is available only to instances of the current class and subclasses of the current class. |
|private[packageName] | The method is available to all classes beneath the `packageName` package. |
|(no modifier) | The method is public |

## 5.2 Calling a Method on a Superclass
In the basic use case, the syntax is the same as Java: Use `super` to refer to the parent class, and then provide the method name.

This calls a method named `onCreate` that is defined in the `Activity` parent class:
```
class WelcomeActivity extends Activity {
  override def onCreate(bundle: Bundle) {
    super.onCreate(bundle)
    // more code here ...
  }
}
```

### Controlling which trait you call a method from
If your class inherits from multiple traits, and those traits implement the same method, you can select a trait name when invoking a method using `super`.

For example: `super[Trait1].methodName`, `super[Trait2].methodName`

## 5.3 Setting Default Values for Method Parameters
You can specify default values for parameters in the method signature (similar to Python).
For example:
```
class Connection {
  def makeConnection(timeout: Int = 5000, protocol: = "http") {
    println("timeout = %d, protocol = %s".format(timeout, protocol))
  }
}
```
Just as with constructor parameters, you can provide default values for method arguments.

## 5.4 Using Parameter Names When Calling a Method
The general syntax for calling a method with named parameters is:
```
methodName(param1=value1, param2=value2, ... )
```
This helps with readability, and also avoids misunderstanding.

## 5.5 Defining a Method That Returns Multiple Items (Tuples)
You want to return multiple values from a method, but don't want to wrap those values in a makeshift class. You can return objects from methods just as in other OOP languages, but Scala also lets you return multiple values from a method using *tuples*.

First, define a method that returns a tuple, then call that method, assigning variable names to the expected return values:

```
def getStockInfo = {
  // Other code..
  ("NFLX", 100.00, 101.00) // This is a Tuple3
}

val (symbol, currentPrice, bidPrice) = getStockInfo
```
In Java, you typically need to create a wrapper class to return multiple values from a method. In Scala, you can just return the data as a tuple.

Tuples can contain up to 22 variables. IF you don't want to assign variable names when calling the method, you can set a variable equal to the tuple the method returns, and then access the values using the tuple underscore syntax. For example:

```
scala> val result = getStockInfo
x: (java.lang.String, Double, Double) = (NFLX,100.0)

scala> result._1
res0: java.lang.String = NFLX

scala> result._2
res1: Double = 100.0
```

## 5.6 Forcing Callers to Leave Parentheses off Accessor Methods
What if you want to enforce a coding style where getter methods can't have parentheses when they are invoked?

You have to define your getter method without parentheses after the method name. This forces consumers of your class to call `crustSize` without parentheses:
```
class Pizza {
  def crustSize = 12  // no parentheses after crustSize
}
```

```
scala> p.crustSize()
<console>:10: error: Int does not take parameters
              p.crustSize()
                             ^

// this works
scala> p.crustSize
res0: Int = 12                              
```

The recommended strategy for calling getter methods that have no side effects is to leave the parentheses off when calling the method.

## 5.7 Creating Methods That Take Variable-Argument Fields
Let's say you want to define a method parameter that can take a variable number of arguments, i.e., a varargs field.

Define a *varargs* field in your method declaration by adding a * character after the field type:
```
def printAll(strings: String*) {
  strings.foreach(println)
}
```
Given that method declaration, the `printAll` method can be called with zero or more parameters.

### Use \_* to adapt a sequence
You can use Scala's \_* operator to adapt a sequence (`Array`, `List`, `Seq`, `Vector`, etc.) so it can be used as an argument for a varargs field.
```
val fruits = List("apple", "banana", "cherry")

printAll(fruits: _*)
```

**Important:** When declaring that a method has a field that can contain varargs field, the varargs field must be the last field in the method signature.

You can call a method with only a varargs field with no arguments. `printAll()  // this is legal`

## 5.8 Declaring That a Method Can Throw an Exception
Use the `@throws` annotation to declare the exception(s) that can be thrown. For example:
```
@throws(classOf[IOException])
@throws(classOf[UnsupportedAudioFileException])   // for more than 1 exception, just add below
def methodName {
  // code here ...
}
```

## 5.9 Supporting a Fluent Style of Programming
What if you want to create an API so developers can write code in a fluent programming style, also known as `method chaining`?

To support this style of programming:
- If your class can be extended, specify `this.type` as the return of fluent style methods.
- If you're sure that your class won't be extended, you can optionally return `this` from your fluent style methods.

For example:
```
class Person {
  protected var fname = ""
  protected var lname = ""

  def setFirstName(firstName: String): this.type = {
    fname = firstName
    this
  }

  def setLastName(lastName: String): this.type = {
    lname = lastName
    this
  }

}

class Employee extends Person {
  protected var role = ""

  def setRole(role: String): this.type = {
    this.role = role
    this
  }

  override def toString = {
    "%s, %s, %s".format(fname, lname, role)
  }
}
```
And here shows how these methods can be chained together:
```
object Main extends App {
  val employee = new Employee

  employee.setFirstName("Al")
          .setLastName("Alexander")
          .setRole("Developer")
  println(employee)
}
```
If you're sure your class won't be extended, specifying `this.type` as the return type of your `set*` methods isn't necessary; you can just return the `this` reference at the end of each fluent style method.

Otherwise, explicitly setting `this.type` as the return type of your `set*` methods ensures that the fluent style will continue to work in your subclasses. In this example, this makes sure that the methods like `setFirstName` on an `Employee` object return an `Employee` reference and not a `Person` reference.
