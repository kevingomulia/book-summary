# Chapter 4 - Classes and Properties

## 4.1 Creating a Primary Constructor
The primary constructor of a Scala class is a combination of:
- The constructor parameters
- Methods that are called in the body of the class
- Statements and expressions that are executed in the body of the class

Fields declared in the body of a Scala class are **assigned when the class is first instantiated**.
```
class Person(var firstName: String, var lastName: String) {
  println("the constructor begins")

  // some class fields
  private val HOME = System.getProperty("user.home")
  var age = 0

  // some methods
  override def toString = s"$firstName $lastName is $age years old"
  def printHome { println(s"HOME = $HOME") }
  def printFullName { println(this) } // uses toString

  printHome
  printFullName
  println("still in the constructor")
}
```

The methods in the body of the class are part of the constructor. When an instance of a `Person` class is created, you'll see the output from the `println` statements at the beginning and end of the class declaration, along with the call to `printHome` and `printFullName` methods.
```
scala> val p = new Person("Adam", "Meyer")
the constructor begins
HOME = /Users/Al
Adam Meyer is 0 years old
still in the constructor
```
The two constructor arguments `firstName` and `lastName` are defined as `var` fields **(IMPORTANT)**, which means they are variable (mutable). As such, Scala generates both accessor and mutator methods for them. You can change their values like: `p.firstName = "Scott"`.

Similarly, age is also declared as a `var`, so it can also be mutated and accessed.

Conversely, the field `HOME` is declared as `private val`, which is akin to Java's `private final`. It can't be accessed directly by other objects and it is immutable.

## 4.2 Controlling the Visibility of the Constructor Fields
The visibility of constructor fields is controlled by the declaration of the fields.

- If the field is a `var`, Scala generates both getter and setter methods for it.
- If the field is a `val`, Scala generates getter methods for it.
- If a field doesn't have either `var` or `val` modifiers, Scala doesn't generate getter or setter methods.
- If `var` or `val` fields are modified with the `private` keyword, it prevents getters and setters from being generated. (rare occasion)

### Case classes
Parameters in the constructor of a case class differ in one way. Case class constructor parameters are `val` **by default**.

## 4.3 Defining Auxiliary Constructors
What if you want to define one or more auxiliary constructors for a class to give consumers of thte class different ways to create object instances?

You can define auxiliary constructors as methods in the class with the name `this`. You can define multiple auxiliary constructors, but they must have different signatures (parameter lists).  Also, each constructor must call one of the previously defined constructors.

This example demonstrates a primary constructor and 3 auxiliary constructors:
```
// primary constructor
class Pizza (var crustSize: Int, var crustType: String) {

  // one-arg auxiliary constructor
  def this(crustSize: Int) {
    this(crustSize, Pizza.DEFAULT_CRUST_TYPE)
  }

  // one-arg auxiliary constructor
  def this(crustType: String) {
    this(Pizza.DEFAULT_CRUST_SIZE, crustType)
  }

  // zero-arg auxiliary constructor
  def this() {
    this(Pizza.DEFAULT_CRUST_SIZE, Pizza.DEFAULT_CRUST_TYPE)
  }
  override def toString = s"A $crustSize inch pizza with a $crustType crust"
}

object Pizza {
  val DEFAULT_CRUST_SIZE = 12
  val DEFAULT_CRUST_TYPE = "THIN"
}
```
Given these constructors, the same pizza can be created in 4 ways:
```
val p1 = new Pizza(Pizza.DEFAULT_CRUST_SIZE, Pizza.DEFAULT_CRUST_TYPE)
val p2 = new Pizza(Pizza.DEFAULT_CRUST_SIZE)
val p3 = new Pizza(Pizza.DEFAULT_CRUST_TYPE)
val p4 = new Pizza
```

- Auxiliary constructors are defined by creating **methods** named `this`.
- Each auxiliary constructor  must begin with a call to a previously defined constructor. (i.e. must call either **previously defined auxiliary constructors** or primary **constructors** in the **first line of its body**.)
- Each constructor must have a different signature.
- One constructor calls another constructor with the name `this`.

In the above example, all of the auxiliary constructors call the primary constructor. That is not necessary, for one auxiliary constructor that takes the `crustType` parameter could have been written like:
```
def this(crustType: String) {
  this(Pizza.DEFAULT_CRUST_SIZE)
  this.crustType = Pizza.DEFAULT_CRUST_TYPE
}
```
Note that `crustSize` and `crustType` parameters are declared in the primary constructor, which lets Scala generate the accessor and mutator methods for you.

### Generating auxiliary constructors for case classes
A *case class* is a special type of class that generates a **lot of boilerplate code** for you. As such, adding what appears to be an auxiliary constructor to a case class is different than usual. This is because they're not really constructors, they're `apply` methods in the comopanion objet of the class.

For example:

```
// initial case class
case class Person (var name: String, var age: Int)
```

This lets you create a new `Person` instance without using the `new` keyword: `val p = Person("John Smith", 30)`. Actually, behind the scenes, Scala compiler converts it to: `val p = Person.apply("John Smith", 30)`.

For instance, if you decide that you want to add auxiliary constructors to let you create new `Person` instances (a) without specifying any parameters, and (b) by only specifying their name, the solution is to add apply methods to the companion object of the `Person` case class in the *Person.scala* file:

```
// the case class
case class Person (var name: String, var age: Int)

// the companion object
object Person {
  def apply() = new Person("<no name>", 0)
  def apply(name: String) = new Person(name, 0)
}
```

This test code demonstrates that this works as desired:
```
object CaseClassTest extends App {

  val a = Person() // corresponds to apply()
  val b = Person("Pam") // corresponds to apply(name: String)
  val c = Person("William Shatner", 82)

  println(a)
  println(b)
  println(c)

  // verify the setter methods work
  a.name = "Leonard Nimoy"
  a.age = 82
  println(a)
}
```
Which results in the following output:
```
Person(<no name>,0)
Person(Pam,0)
Person(William Shatner,82)
Person(Leonard Nimoy,82)
```

## 4.4 Defining a Private Primary Constructor
For example, to enforce the Singleton pattern

To make the primary constructor private, insert the `private` keyword in between the class name and any parameters the constructor accepts:
```
// a private no-args primary constructor
class Order private { ...

// a private one-arg primary constructor
class Person private (name: String) { ...
```
This keeps you from being able to create an instance of the class.
```
scala> class Person private (name: String)
defined class Person

scala> val p = new Person("Mercedes")
<console>:9: error: constructor Person in class Person cannot be accessed
in object $iw
        val p = new Person("Mercedes")
                ^
```
A simple way to enforce the Singleton pattern in Scala is to make the primary constructor `private`, then put a `getInstance` method in the *companion object* of the class:
```
class Brain private {
  override def toString = "This is the brain."
}

object Brain {
  val brain = new Brain
  def getInstance = brain
}

object SingletonTest extends App {
  // this won't compile
  // val brain = new Brain

  // this works
  val brain = Brain.getInstance
  println(brain)
}
```
Alternatively you can put all the methods in a Scala *object* instead of a class.
```
object FileUtils {
  def readFile(filename: String) = {
    // code here ...
  }
  def writeToFile(filename: String, contents: String) {
    // code here ...
  }
}
```

```
val utils = new FileUtils  // won't compile
```
So there's no need for a private class constructor; just don't define a class.

## 4.5 Providing Default Values for Constructor Parameters
Give the parameter a default value in the constructor declaration (similar to Python).

e.g. `class Socket (val timeout: Int = 10000)`

If this feature didn't exist, two constructors would be required to get the same functionality: 1 primary one-arg constructor and an auxiliary zero-args constructor:
```
class Socket(val timeout: Int) {
  def this() = this(10000)
  override def toString = s"timeout: $timeout"
}
```

## 4.6
