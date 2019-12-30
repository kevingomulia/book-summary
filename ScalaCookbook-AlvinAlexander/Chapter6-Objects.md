# Chapter6 - Objects
In Scala, object refers to both an instance of a class, and it is also a keyword.

The first 3 recipes in this chapter look at an object as an instance of a class, show how to cast objects from one type to another, demonstrate the Scala equivalent of Java's `.class` approach, and show how to determine the class of an object.

The remaining recipes demonstrate how the `object` keyword is used for other purposes.

## 6.1 Object Casting [COMMON]
Let's say we want to cast an instance of a class from one type to another, such as when creating objects dynamically.

**Answer:** We can cast an instance to the desired type using the `asInstanceOf` method. The `asInstanceOf` method is defined in the Scala `Any` class and is therefore available on all objects.

In dynamic programming, it is often necessary to cast from one type to another. For instance, this approach is needed when using the Spring Framework and instantiating beans from an application context file:

```
// open/read the application context file
val ctx = new ClassPathXmlApplicationContext("applicationContext.xml")

// instantiate our dog and cat objects from the application context
val dog = ctx.getBean("dog").asInstanceOf[Animal]
val cat = ctx.getBean("cat").asInstanceOf[Animal]
```

Or when reading a YAML configuration file:
```
val yaml = new Yaml(new Constructor(classOf[EmailAccount]))
val emailAccount = yaml.load(text).asInstanceOf[EmailAccount]
```
In Scala, we will write something like:
```
val cm = new ConfigurationManager("config.xml")

// instance of Recognizer
val recognizer = cm.lookup("recognizer").asInstanceOf[Recognizer]
```
Be aware that this type of coding can lead to a `ClassCastException`. Hence, use a `try/catch` expression to handle this situation.

## 6.2 The Scala Equivalent of Java's .class
Use the Scala `classOf` method instead of Java's `.class`. For example, this is how to pass a class of type `TargetDataLine` to a method named `DataLine.Info`:
```
val info = new DataLine.Info(classOf[TargetDataLine], null)

// In Java:
info = new DataLine.Info(TargetDataLine.class, null)
```
The `classOf` method is defined in the Scala `Predef` object and is therefore available in all classes without requiring an import.

## 6.3 Determining the Class of an Object [COMMON]
Since you don't have to explicitly declare types with Scala, you may occasionally want to print the class/type of an object to understand how Scala works, or to debug code.

**Answer:** You can call the `getClass` method on the object.

This can be very useful when working with something like Scala's XML library, so you can understand which classes you're working with in different situations.

When you can't see object types in an IDE, writing little tests like this in the REPL helps a lot.

## 6.4 Launching an Application with an Object [COMMON]
Let's say you want to start an application with a `main` method, or provide the entry point for a script.

**Answer:** There are two ways to create a launching point for your app:
1. Define an object that extends the `App` trait
2. Define an object with a properly defined `main` method

An example for approach #1:
```
object Hello extends App {
  println("Hello world")
}
```
The code in the body of the `object` is automatically run, just as if it were inside a `main` method. Simply save the code to a file named `Hello.scala`, compile it with `scalac`, and then run it with `scala`:

```
$ scalac Hello.scala

$ scala Hello
Hello world
```

For approach #2:
Manually implement a `main` method with the correct signature in an `object`, similar to Java.

```
object Hello2 {
  def main(args: Array[String]) {
    println("Hello world")
  }
}
```

Note that in both cases, Scala applications are launched from an **object**, not a class.

## 6.5 Creating Singletons with object
Create Singleton objects in Scala with the `object` keyword. For instance:
```
object CashRegister {
  def open { println("opened") }
  def close { println("closed") }
}
```
With `CashRegister` defined as an object, there can only be one instance of it. Its methods are called like static methods on a Java class:
```
object Main extends App {
  CashRegister.open
  CashRegister.close
}
```
This pattern is common when creating utility methods, like `DateUtils`. Singleton objects make great reusable messages when using actors. If you have a number of actors that can all receive start and stop messages, you can create Singletons like this:

```
case object StartMessage
case object StopMessage
```
You can then use those objects as messages that can be sent to actors:
```
inputValve ! StopMessage
outputValve ! StopMessage
```
Actors and concurrency will be discussed more in depth in Chapter 13.

## 6.6 Creating Static Members with Companion Objects
Now, what if you want to create a class that has instance methods and static methods? Unlike Java, Scala does not have a `static` keyword.

You can define nonstatic (instance) members in your *class*, and define "static" members in an *object* that has the same name as the class, and is in the same file as the class.

This object is known as a *companion object*.

```
class Pizza (var crustType: String) {
  override def toString = "Crust type is " + crustType
}

// Companion object
object Pizza {
  val CRUST_TYPE_THIN = "thin"
  val CRUST_TYPE_THICK = "thick"
  def getFoo = "Foo"
}
```
Now, members of the `Pizza` object can be accessed as static members:
```
println(Pizza.CRUST_TYPE_THIN)
println(Pizza.getFoo)
```
You can also create a new `Pizza` instance and use it as usual:
```
var p = new Pizza(Pizza.CRUST_TYPE_THICK)
println(p)
```
**Important:** A class and its companion object can access each other's private members.
```
class Foo {
  private val secret = 2
}

object Foo {
  // access the private class field 'secret'
  def double(foo: Foo) = foo.secret * 2
}

object Driver extends App {
  val f = new Foo
  println(Foo.double(f)) // prints 4
}
```
## 6.7 Putting Common Code in Package Objects
You want to make functions, fields, and other code available at a package level, without requiring a class or object.

**Answer:** Put the code within a package in a *package object*.

By convention, put your code in a file named *package.scala* in the directory where you want your code to be available. For instance, if you want your code to be available to all classes in the `com.kevingomulia.myapp.model` package, create a file named `package.scala` in the *com/kevingomulia/myapp/model* directory of your project.

In the *package.scala* source code, remove the word `model` from the end of the package statement and use that name to declare the name of the package object.
```
package com.kevingomulia.myapp

package object model {

  // field
  val MAGIC_NUM = 42

  // method
  def echo(a: Any) { println(a) }

  // enumeration
  object Margin extends Enumeration {
    type Margin = Value
    val TOP, BOTTOM, LEFT, RIGHT = Value
  }

  // type definition
  type MutableMap[K, V] = scala.collection.mutable.Map[K, V]
  val MutableMap = scala.collection.mutable.Map

}
```
You can now access this code directly from other classes, traits, and objects in the package `com.kevingomulia.myapp.model`:

```
package com.kevingomulia.myapp.model

object MainDriver extends App {
  echo(MAGIC_NUM)
  echo(Margin.LEFT)
  ...
}
```
Package objects are a great place to put methods and functions that are common to the package, as well as constants, enumerations, and implicit conversions.

## 6.8 Creating Object Instances without Using the New Keyword
You have seen that Scala code looks cleaner when you don't always have to use the `new` keyword to create a new instance of a class.

There are two ways to do this:
1. Create a companion object for your class, and define an `apply` method in the companion object with the desired constructor signature
2. Define your class as a *case class*.

### Approach 1: Create a companion object with an apply method.
First, define a `Person` class and `Person` object in the same file. Define an `apply` method that takes the desired parameters. This method is essentially the constructor of your class.

```
class Person {
  var name: String = _
}

object Person {
  def apply(name: String): Person = {
    var p = new Person
    p.name = name
    p     // return the person
  }
}
```
Now you can create new `Person` instances without using the `new` keyword.

### Approach 2: Declare your class as a case class
Declare your class as a *case class*, and define it with the desired constructor.
```
case class Person (var name: String)
```
Case class generates an `apply` method in the companion object for you. However, *case class* also creates much more code for you than just the `apply` method.

### Providing multiple constructors with additional apply methods
You can define multiple `apply` methods in the companion object that provide the constructor signatures you want.

```
class Person {
  var name = ""
  var age = 0
}

object Person {

  // a one-arg constructor
  def apply(name: String): Person = {
    var p = new Person
    p.name = name
    p
  }

  // a two-arg constructor
  def apply(name: String, age: Int): Person = {
    var p = new Person
    p.name = name
    p.age = age
    p
  }
}
```
### Providing multiple constructors for case classes
When a case class is created, it writes the accessor and (optional) mutator methods *only* for the default constructor. So, (a), it is best to define all class parameters in the default constructor, or (b) write `apply` methods for the auxiliary constructors you want.

## 6.9 Implement the Factory Method in Scala with apply
To let subclasses declare which type of object should be created, and to keep the object creation point in **one location**, you want to implement the factory method in Scala.

One approach is to take advantage of how a Scala companion object's `apply` method works. Rather than creating a "get" method for your factory, you can place the factory's decision making algorithm in the `apply` method.

For instance, suppose you want to create an Animal factory that returns instances of Cat and Dog classes, based on what you ask for. By writing an apply method in the companion object of an Animal class, users of your factory can create new Cat and Dog instances like this:

```
val cat = Animal("cat") // creates a Cat
val dog = Animal("dog") // creates a Dog
```

To implement this, create a parent `Animal` trait : `trait Animal { def speak }`

In the same file, create:
1. A companion object
2. The classes that extend the base trait
3. A suitable `apply` method
```
object Animal {
  private class Dog extends Animal {
    override def speak { println("woof") }
  }

  private class Cat extends Animal {
    override def speak { println("meow") }
  }

  // the factory method
  def apply(s: String): Animal = {
    if (s == "dog") new Dog
    else new Cat
  }
}
```
