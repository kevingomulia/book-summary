# Chapter 19 - Types - Part II

## 19.5 Make Immutable Collections Covariant
You can define a collection of immutable elements as invariant, but your collection will be **much more flexible** if you declare that your type parameter is *covariant*. To make a type parameter covariant, declare it with the `+` symbol, like `[+A]`.

Covariant type parameters are shown in the Scaladoc for immutable collection classes like `List`, `Vector`, and `Seq`:
```
class List[+T]
class Vector[+A]
trait Seq[+A]
```
By defining the type parameter to be covariant, you create a situation where the collection can be used in a more flexible manner.

For example:
```
trait Animal { def speak }

class Dog(var name: String) extends Animal {
  def speak { println("Woof") }
}

class SuperDog(name: String) extends Dog(name) {
  override def speak {println("I'm a SuperDog") }
}
```
Next, the `makeDogsSpeak` method accepts an immutable `Seq[Dog]` (instead of a mutable `ArrayBuffer[Dog]` in the previous recipe).

```
def makeDogsSpeak(dogs: Seq[Dog]) {
  dogs.foreach(_.speak)
}
```
As with the `ArrayBuffer` in the previous recipe, you can pass a sequence of type `[Dog]` into `makeDogsSpeak` without a problem, and you can also pass a `Seq[SuperDog]` into the `makeDogsSpeak` method successfully:
```
// this works
val dogs = Seq(new Dog("Fido"), new Dog("Tanner"))
makeDogsSpeak(dogs)

// this also works
val superDogs = Seq(new SuperDog("Wonder Dog"), new SuperDog("Scooby"))
makeDogsSpeak(superDogs)
```

Because `Seq` is immutable and defined with a covariant parameter type, `makeDogsSpeak` can now accept collections of both `Dog` and `SuperDog`.

You can create a collection class with a covariant type parameter.
For example, create a collection class that can hold one element. Because you don't want the collection element to be mutated, define the element as a `val`, and make the type parameter covariant with `+A`:
```
class Container[+A] (val elem: A)
```

Using the same type hierarchy as shown in the Solution, modify the `makeDogsSpeak` method to accept a `Container[Dog]`:
```
def makeDogsSpeak(dogHouse: Container[Dog]) {
  dogHouse.elem.speak()
}
```
Now, you can pass a Container[Dog] into makeDogsSpeak
```
val dogHouse = new Container(new Dog("Tanner"))
makeDogsSpeak(dogHouse)
```
Lastly, since we added the `+` symbol to the parameter, you can also pass a `Container[SuperDog]` into `makeDogsSpeak`
```
val superDogHouse = new Container(new SuperDog("Wonder Dog"))
makeDogsSpeak(superDogHouse)
```
Because the `Container` element is immutable and its mutable type parameter is marked as covariant, all of this code works successfully. Note: if you change the `Container`’s type parameter from `+A` to `A`, the last line of code won’t compile.

In summary, defining an immutable collection to take a covariant generic type parameter makes the collection more flexible.

## 19.6 Create a Collection Whose Elements Are All of Some Base Type
Let's say you want to specify that a class or method takes a type parameter, and that parameter is limited so it can only be a base type, or a subtype of that base type (not custom defined type).

Define the class/method by specifying the type parameter with an *upper bound*.

If you're working with an implicit conversion, you'll want to use a *view bound* instead of an upper bound. To do this, use the `<%` symbol instead of the `<:` symbol.

You can use the same technique when you need to limit your class to take a type that extends multiple traits. For example, to create a `Crew` that only allows types that extend `CrewMember` and `StarfleetTrained`, declare the `Crew` like this:
```
class Crew [A <: CrewMember with StarfleetTrained] extends ArrayBuffer[A]
```
Then, if you have a `class Officer` that `extends CrewMember`, you can define them like so:
```
val kirk = new Officer with Captain with StarfleetTrained
val spock = new Officer with FirstOffier with StarfleetTrained
```
Then, when you want to construct a list of officers:
```
val officers = new Crew[Officer with StarfleetTrained]()
officers += kirk
officers += spock
```
This approach works as long as the instances have those types somewhere in their class hierarchy. For instance, if you define a new `StarfleetOfficer` like this:
```
class StarfleetOfficer extends Officer with StarfleetTrained

// then you can define the kirk instance like this
val kirk = new StarfleetOfficer with Captain
```
With this definition, `kirk` can still be added to the `officers` collection; the instance still extends `Officer` and `StarfleetTrained`.

### Methods
Methods can also take advantage of this syntax. For instance, you can add a little behavior to `CrewMember` and `RedShirt`:
```
trait CrewMember {
  def BeamDown { println("beaming down") }
}
class RedShirt extends CrewMember {
  def putOnRedShirt { println("putting on my red shirt") }
}
```
Now, you can write methods to work specifically on their types. This method works for any `CrewMember`:
```
def beamDown[A <: CrewMember](crewMember: Crew[A]) {
  crewMemeber.foreach(_.beamDown)
}
```
But this method will only work for `RedShirt` types:
```
def getReadyForDay[A <: RedShirt](redShirt: Crew[A]) {
  redShirt.foreach(_.putOnRedShirt)
}
```
In both cases, you control which type can be passed into the method using an appropriate upper bound definition on the method's type parameter.

## 19.7 Selectively Adding New Behavior to a Closed Model
You have a closed model, and want to add new behavior to certain types within that model, while potentially excluding that behavior from being added to other types.

Implement your solution as a *type class*.

In Scala, to write a single `add` method that would add any two numeric parameters, regardless of whether they were an `Int`, `Double`, `Float`, or other numeric value; spoiler: this won't work.

Because a `Numeric` type class already exists in the Scala library, you can create an `add` method that accepts different numeric types like this:
```
def add[A](x: A, y: A)(implicit numeric: Numeric[A]): A = numeric.plus(x, y)
```

### Creating a type class
The process of creating a type class has a formula:
- Usually you start with a need, like having a closed model to which you want to add new behavior.
- To add the new behavior, you define a type class. The typical approach is to create a base trait, and then write specific implementations of that trait using implicit objects.
- Back in your main application, create a method that uses the type class to apply the behavior to the closed model, such as writing the `add` method in the previous example.

To give an example, assume you have a closed model that contains `Dog` and `Cat` types, and you want to give a `Dog` the capability to speak. However, while doing this, you don't want to make a `Cat` speak.

The closed model is defined in a class named *Animals.scala*:
```
package typeclassdemo

trait Animal
final case class Dog(name: String) extends Animal
final case class Cat(name: String) extends Animal  
```
To begin making a new `speak` behavior available to `Dog`, create a type class that implements the `speak` behavior for a `Dog`, but not a `Cat`:
```
package typeclassdemo

object Humanish {
  // the type class
  // defines an abstract method named 'speak'
  trait HumanLike[A] {
    def speak(speaker: A): Unit
  }

  // companion object
  object HumanLike {
    // implement the behavior for each desired type (in this case, Dog)
    implicit object DogIsHumanLike extends HumanLike[Dog] {
      def speak(dog: Dog) {println("Woof")}
    }
  }
}
```
Now, you can use the new functionality back in your main application:
```
package typeclassdemo

object TypeClassDemo extends App {
  import Humanish.HumanLike

  // create a method to make an animal speak
  def makeHumanLikeThingSpeak[A](animal: A)(implicit humanLike: HumanLike[A]) {
    humanLike.speak(animal)
  }

  // because HumanLike implemented this for a Dog, it will work
  makeHumanLikeThingSpeak(Dog("Rover"))

  // this won't compile for a Cat
  // makeHumanLikeThingSpeak(Cat("Lea"))
}
```
There are a few other things to notice from this code:
- The `makeHumanLikeThingSpeak` is similar to the `add` method in the first example.
- In the first example, the `Numeric` type class already existed, so you could just use it to create a `add` method. But if you're starting from scratch, you need to create your own type class (the code in the `HumanLike` trait)
- Because a `speak` method is defined in the `DogIsHumanLike` implicit object, which extends `HumanLike[Dog]`, a `Dog` can be passed into the `makeHumanLikeThingSpeak` method.

As shown in the examples, one benefit of a type class is that you can add behavior to a closed model (`final`). Another benefit is that it lets you define methods that take generic types, and provide control over what those types are.

For instance, in the first example, the `add` method takes `Numeric` types, and it works for all numeric types. Also, the `add` method is type safe, so if you attempted to pass a `String` into it, it won't compile.

## 19.8 Building Functionality with Types

### Example 1: Creating a Timer
First, let's create a timer that looks like a control structure, and works like the Unix time command.

On Unix, you can run a `time` command to see how long commands take to execute:
```
$ time find . -name "*.scala"
```
You can create a similar `timer` method in Scala:
```
val (result, time) = timer(someAlgorithm)
println(s"result: $result, time: $time)
```
In this example, the `timer` runs a method called `someAlgorithm`, and then returns the result of the algorithm along with its execution time.

The `timer` code involves the use of a generic type parameter:
```
def timer[A](blockOfCode: => A) = {
  val startTime = System.nanoTime
  val result = blockOfCode
  val stopTime = System.nanoTime
  val delta = stopTime - startTime
  (result, delta/1000000d)
}
```
The `timer` method uses Scala's *call-by-name* syntax to accept a block of code as a parameter. Rather than declare a specific return type from the method, you declare the return type to be a generic type parameter.

This is a simple use of specifying a generic type in a noncollection class.

### Example 2: Writing Your Own "Try" Classes
Before Scala 2.10, there was no such thing as the `Try`, `Success`, and `Failure` classes in *scala.util*. Previously, you have to write your own solution that you called `Attempt`, `Succeeded` and `Failed` that you can use like this:
```
val x = Attempt("10".toInt) //  Succeeded(10)
val y = Attempt("10A".toInt) // Failed(Exception)
```
Firstly, you'll need a class named `Attempt` in *Attempt.scala*:
```
// version 1
sealed class Attempt[A]

object Attempt {
  def apply[A](f: => A): Attempt[A] =
    try {
      val result = f
      return Succeeded(result)
    } catch {
      case e: Exception => Failed(e)
    }
}

final case class Failed[A](val exception: Throwable) extends Attempt[A]
final case class Succeeded[A](value: A) extends Attempt[A]
```
Similar to the previous `timer` code, the `apply` method takes a call-by-name parameter, and the return type is specified as a generic type parameter. Because `apply` returns a type of `Attempt`, it's necessary there; because `Failed` and `Succeeded` extend `Attempt`, it's propagated there as well.

While this lets you write the basic `x` and `y` examples, to be really useful, your API needs a new method named `getOrElse` that lets you get the information from the result, whether that result happens to be of type `Succeeded` or `Failed`.

Let's add it to our code:
```
// version 2
sealed abstract class Attempt[A] {
  def getOrElse[B >: A](default: => B): B = if (isSuccess) get else default
  var isSuccess = false
  def get : A
}

object Attempt {
  def apply[A](f: => A): Attempt[A] =
    try {
      val result = f
      Succeeded(result)
    } catch {
      case e: Exception => Failed(e)
    }
}

final case class Failed[A](val exception: Throwable) extends Attempt[A] {
  isSuccess = false
  def get: A = throw exception
}

final case class Succeeded[A](result: A) extends Attempt[A] {
  isSuccess = true
  def get = result
}
```

The variable `isSuccess` is added to `Attempt` and it can be set in `Succeeded` or `Failed` method. An abstract method named `get` is also declared in `Attempt` so it can be implemented in two subclasses. These changes let the `getOrElse` method in `Attempt` to work.

Most importantly is the `getOrElse` method signature:
```
def getOrElse[B >: A](default: => B): B = if (isSuccess) get else default
```
Because of the way `getOrElse` works, it can either return the type `A`, which is the result of the expression, or type `B`, which the user supplies.

In Scala 2.10, the `getOrElse` method declared in `Try`:
```
def getOrElse[U >: T](default: => T): T = if (isSuccess) get else default
```
The `map` method declared in `Success` shows how to define a call-by-name parameter that transforms a type `T` to a type `U`:
```
def map[U](f: T => U): Try[U] = Try[U](f(value))
```
Its `flatten` method uses the `<:<` symbol that wasn't covered in this chapter. When used as `A <:< B`, it declares that "A must be a subtype of B". Here's how it's used in the `Success` class:
```
def flatten[U](implicit ev: T <:< Try[U]): Try[U] = value
```
