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

## 4.6 Overriding Default Accessors and Mutators
This is tricky because you can't override the getter and setter methods Scala generates for you, at least not if you want to stick with the Scala naming conventions.

For example, this won't compile:
```
// error: this won't work
class Person(private var name: String) {
  // this line essentially creates a circular reference
  def name = name
  def name_=(aName: String) { name = aName }
}
```
This code generates 3 errors on cpmilation:
```
Person.scala:3: error: overloaded method name needs result type
  def name = name
             ^

Person.scala:4: error: ambiguous reference to overloaded definition,
both method name_= in class Person of type (aName: String)Unit
and method name_= in class Person of type (x$1: String)Unit
match argument types (String)
  def name_=(aName: String) { name = aName }
      ^

Person.scala:4: error: method name_= is defined twice
  def name_=(aName: String) { name = aName }
      ^
three errors found
```
Both the constructor parameter and the getter method are named `name`, and Scala won't allow that.

To solve this, change the name of the field you use in the class constructor so it won't collide with the name of the getter method you want to use. A common approach is to add a leading underscore to the parameter name. So, if you want to manually create a getter method called `name`, use the parameter name `_name` in the constructor, then declare your getter and setter methods according to Scala conventions:
```
class Person(private var _name: String) {
  def name = _name              // accessor - getter
  def name_=(aName: String) {    // mutator - setter
    _name = aName
  }
}
```
Note that the **constructor parameter is declared `private` and `var`**. This keeps Scala from exposing that field to other classes and make the field mutable.

The getter method is named `name`, and the setter method is named `name_=`, which conforms to the Scala convention and allows:
```
val p = new Person("John")
p.name = "Adam"    // setter
println(p.name)    // getter
```

As shown above, the recipe for overriding default getter and setter methods is:
1. Create a private `var` constructor parameter with a name you want to reference from within your class. In the example above, the field is named `_name`.
2. Define getter and setter names that you want other classes to use. Above, the getter name is `name`, and the setter name is `name_=`
3. Modify the body of the getter and setter methods as desired.

## 4.7 Preventing Getter and Setter Methods from Being Generated
As above, define the field with the `private` or `private[this]` access modifiers.

Defining a field as `private` limits the field so it's only available to instances of the same class, in this case instances of the `Stock` class. **Any instance of a `Stock` class** can access a **private field of any other `Stock` instance**.

For example, the `isHigher` method in the `Stock` class can access the `price` field both in its object and in the other `Stock` object it is being compared to:
```
class Stock {
// a private field can be seen by any Stock instance
private var price: Double = _
  def setPrice(p: Double) { price = p }
  def isHigher(that: Stock): Boolean = this.price > that.price
}
```

### Object-private fields
Defining a field as `private[this]` makes the field *object-private*, which means that it can only be accessed from the object that contains it. It cannot be accessed by other instances of the same type.

This makes it more private than the plain `private` setting.

## 4.8 Assigning a Field to a Block or Function
So you want to initialize a field in a class using a block of code, or by calling a function?

You can set the field equal to the desired block of code or function. Optionally, define the field as `lazy` if the algorithm requires a long time to run.

For example:
```
class Foo {
  // set 'text' equal to the result of the block of code
  val text = {
    var lines = ""
    try {
      lines = io.Source.fromFile("/etc/passwd").getLines.mkString
    } catch {
      case e: Exception => lines = "Error happened"
    }
    lines
  }

  println(text)
}
object Test extends App {
  val f = new Foo
}
```
the assignment of the code block to the text field and the `println` statement are both in the body of the Foo class, they are in the class’s constructor, and will be executed when a new instance of the class is created. Compiling and running this example will either print the contents of the file, or the “Error happened” message from the catch block.

When necessary, define a field like this to be `lazy`, which means it won't b e evaluated until it is accessed.

## 4.9 Setting Uninitialized var Field Types
In Scala, you don't initialize a var as null like in Java's: `var x = null`

In general, define the field as an `Option`. For example, `Option[String]` If the field hasn't been assigned, it becomes a `None`. Otherwise, it will be a `Some[String]`. This helps in case you want to call a method on the field, but you don't know if there will be a value there.

## 4.10 Handling Constructor Parameters When Extending a Class
Let's say that you want to extend a base class and you need to work with the constructor parameters declared in the base class as well as new parameters in the subclass.

Declare your base class as usual with `val` or `var` constructor parameters. When defining a subclass constructor, leave the `val` or `var` declaration off of the fields that are common to both classes. Then define new constructor parameters in the subclass as `val` or `var`
fields, as usual.

For example, we have a `Person` base class:
```
class Person (var name: String, var address: Address) {
  override def toString = if(address == null) name else s"$name @ $address"
}
```

Next, define `Employee` as a subclass of `Person`, and it takes the constructor parameters `name, address,` and `age`.

```
class Employee (name: String, address: Address, var age: Int) extends Person (name, address) {
  // rest of the class
}
```

Since `name` and `address` parameters are common to the parent `Person` class, leave the `var` declaration off of those fields. Since `age` is new, declare it as a `var`.

Scala already generated the getter and setter methods for the `name` and `address` fields in the `Person` class, so when you declare the `Employee` class, Scala won't attempt to generate methods for those fields. Therefore, the `Employee` class inherits that behavior from `Person`, as expected.

## 4.11 Calling a Superclass Constructor
In Scala, you can control the superclass constructor that's called by the primary constructor in a subclass, but you can't control the superclass constructor that's called by an auxiliary constructor in the subclass.

For instance, in the following code, the `Dog` class is defined to call the primary constructor of the `Animal` class:

```
class Animal (var name: String) { ... }

class Dog (name: String) extends Animal (name) { ... }
```
However, if the `Animal` class has multiple constructors, the primary constructor of the `Dog` class can call any of those constructors.

For example:
```
// (1) primary constructor
class Animal (var name: String, var age: Int) {
  // (2) auxiliary constructor
  def this (name: String) {
    this(name, 0)
  }

  override def toString = s"$name is $age years old"
}

// calls the Animal one-arg constructor
// specify in the constructor in the extends clause
class Dog (name: String) extends Animal(name) {
  println("Dog constructor called")
}
```
Alternatively, it could call the two-arg primary constructor of the `Animal` class:
```
// call the two-arg constructor
class Dog (name: String) extends Animal (name, 0) {
  println("Dog constructor called")
}
```

### Auxiliary constructors
Since the first line of an auxiliary constructor must be a call to another constructor of the current class, there is no way for auxiliary constructors to call a superclass constructor.

## 4.12 When to Use an Abstract Class
Scala has traits, and a trait is more flexible than an abstract class. So when should you use an abstract class?

There are two main reasons to use an abstract class in Scala:
- You want to create a base class that requires constructor arguments.
- The code will be called from Java code.

**Traits don't allow constructor parameters**, so use an abstract class whenever a base behavior must have constructor parameters.

Secondly, Java code can't call Scala traits (because Java doesn't have traits).

Also, beware that a class can extend only one abstract class. In Scala, there is no need for an `abstract` keyword; simply leaving the body of the method undefined makes it abstract. This is also how abstract methods in traits are defined.

## 4.13 Defining Properties in an Abstract Base Class (or Trait)
You can declare both `val` or `var` fields in abstract class (or trait), and those fields can be abstract or have concrete implementations.

### Abstract val and var fields
The following example demonstrates an `Animal` trait with abstract `val` and `var` fields, along with a simple concrete method named `sayHello`, and an override of the `toString` method:
```
abstract class Pet (name: String) {
  val greeting: String
  var age: Int
  def sayHello { println(greeting) }
  override def toString = s"I say $greeting, and I'm $age"
}
```
The following `Dog` and `Cat` classes extend `Animal` class and provide values for the `greeting` and `age` fields. Notice that the fields are again specified as `val` or `var`:
```
class Dog (name: String) extends Pet (name) {
  val greeting = "Woof"
  var age = 2
}

class Cat (name: String) extends Pet (name) {
  val greeting = "Meow"
  var age = 5
}
```
You can declare abstract fields in an abstract class as either `val` or `var`. The way abstract fields work in abstract classes (or traits) is interesting:
- An abstract `var` field results in getter and setter methods being generated for the field.
- An abstract `val` field results in a getter method being generated for the field.
- When you define an abstract field in an abstract class or trait, the Scala compiler does not create a field in the resulting code; it only generates the methods that correspond to the `val` or `var` field.

Because of this, when you provide concrete values for these fields in your concrete classes, you **must** again define your fields to be `val` or `var`. Because the fields don't actually exist in the abstract base class (or trait), the `override` keyword is not necessary.

As a result, you may see developers define a `def` that takes no parameters in the abstract base class rather than defining a `val`. They can then define a `val` in the concrete class, if desired. For example:
```
abstract class Pet (name: String) {
  def greeting: String
}

class Dog (name: String) extends Pet (name) {
  val greeting = "Woof"
}

object Test extends App {
  val dog = new Dog("Fido")
  println(dog.greeting)
}
```

### Concrete val fields in abstract classes
When defining a concrete `val` field in an abstract class, you can provide an initial value, and then override that value in the concrete subclasses:
```
abstract class Animal {
  val greeting = { println("Animal"); "Hello" }
}

class Dog extends Animal {
  override val greeting = { println("Dog"); "Woof" }
}

object Test extends App {
  new Dog
}
```
This results in :
```
Animal
Dog
```
You can prevent a concrete `val` field in an abstract base class from being overridden in a subclass by declaring the field as `final val`.

### Concrete var fields in abstract classes
You can also give `var` fields an initial value in your trait or abstract class, then refer them in your concrete subclasses.

The fields are declared and initialized in the abstract base class, so there is no need to redeclare the fields as `val` or `var` in the concrete subclass.

## 4.14 Generating Boilerplate Code with Case Classes
Define your class as a *case class*, defining any parameters it needs in its constructor:
```
// name and relation are `val` by default
case class Person(name: String, relation: String)
```
Defining a class as a case class results in a lot of boilerplate code being generated, with the following benefits:
- An `apply` method is generated, so you don't need to use the `new` keyword to create a new instance of the class.
- Accessor methods are generated for the constructor parameters because case class constructor parameters are `val` by default. Mutator methods are generated for parameters declared as `var`.
- A default `toString` method is generated.
- An `unapply` method is generated, making it easy to use case classes in match expressions.
- `equals` and `hashCode` methods are generated.
- A `copy` method is generated.

When you define a class as a case class, you don't have to use the `new` keyword to create a new instance.

Case classes are primarily intended to create "immutable records" that you can easily use in pattern-matching expression.

## 4.15 Defining an equals Method (Object Equality)
Like Java, you define an `equals` method (and `hashCode` method) in your class to compare two instances. Unlike Java, you then use the `==` method to compare the equality of two instances.

In Java, the `==` operator compares "reference equality" but in Scala, `==` is a method you use on each class to compare the equality of two instances, calling your `equals` method under the covers.

An equivalence relation should have these three properties:
- It is reflexive: for any instance x of type Any, x.equals(x) should return true.
- It is symmetric: for any instances x and y of type Any, x.equals(y) should return true if and only if y.equals(x) returns true.
- It is transitive: for any instances x, y, and z of type `AnyRef`, if `x.equals(y)` returns `true` and `y.equals(z)` returns `true`, then `x.equals(z)` should return true.

## 4.16 Creating Inner Classes
You can declare one class inside another class. For example:
```
class PandorasBox {
  case class Thing (name: String)

  var things = new collection.mutable.ArrayBuffer[Thing]()
  things += Thing("Evil Thing #1")
  things += Thing("Evil Thing #2")

  def addThing(name: String) { things += new Things(name) }
}
```

This lets users of `PandorasBox` access the collection of `things` inside the box, while code of `PandorasBox` generally doesn't have to worry about the concept of a `Thing`:
```
object ClassInAClassExample extends App {

  val p = new PandorasBox
  p.things.foreach(println)

}
```
As shown, you can access the things in `PandorasBox` with the `things` method. You can also add new things to `PandorasBox` by calling the `addThing` method.

In Scala, inner classes are bound to the outer object.
