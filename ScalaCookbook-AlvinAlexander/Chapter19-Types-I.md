# Chapter 19 - Types - Part I

This chapter provides recipes for the most common problems you'll encounter, but if you need to go deeper, read  *Programming in Scala*.

Scala's type system uses a collection of symbols to express different generic type concepts, including variance, bounds and constraints.

## Variance
*Type variance* is a generic type concept, and defines the rules by which parameterized types can be passed into methods.

Descriptions of type variance symbols:
|Symbols|Name|Description|
|---|---|---|
|Array[T]|Invariant|Used when the elements in the container are mutable, e.g. can only pass Array[String] to a method expecting Array[String]|
|Seq[+A]|Covariant|Used when elements in the container are immutable. This makes the container more flexible, e.g. Can pass a Seq[String] to a method expecting Seq[Any]|
|Foo[-A] and Function1[-A, +B]|Contravariant|Contravariance is the opposite of covariance, and it's hardly used.|

Here's an example to help with understanding variance:
```
class Grandparent
class Parent extends Grandparent
class Child extends Parent

class InvariantClass[A]
class CovariantClass[+A]
class ContravariantClass[-A]

class VarianceExamples {
  def invarMethod(x: InvariantClass[Parent]) {}
  def covarMethod(x: CovariantClass[Parent]) {}
  def contraMethod(x: ContravariantClass[Parent]) {}

  invarMethod(new InvariantClass[Child]) // ERROR - won't compile
  invarMethod(new InvariantClass[Parent]) // success
  invarMethod(new InvariantClass[Grandparent]) // ERROR - won't compile

  covarMethod(new CovariantClass[Child]) // success
  covarMethod(new CovariantClass[Parent]) // success
  covarMethod(new CovariantClass[Grandparent]) // ERROR - won't compile

  contraMethod(new ContravariantClass[Child]) // ERROR - won't compile
  contraMethod(new ContravariantClass[Parent]) // success
  contraMethod(new ContravariantClass[Grandparent]) // success
}
```

## Bounds
Bounds let you place restrictions on type parameters.

|Symbol|Name|Description|
|---|---|---|
|A <: B|Upper bound|A must be a subtype of B|
|A >: B|Lower bound|A must be a supertype of B. Not common|
|A <: Upper >: Lower|Lower and upper bounds used together|The type A has both an upper and lower bound|

To find some lower bound examples, search the Scaladoc of classes like `List` for the `>:` symbol.

There are several additional symbols for bounds. For instance, a *view bound* is written as `A <% B`, and a *context bound* is written as `T : M`

## Type Constraints
Scala lets you specify additional type constraints.

- `A =:= B` // A must be equal to B
- `A <:< B` // A must be a subtype of B
- `A <%< B` // A must be viewable as B

## Type Examples in Other Chapters
- **Recipe 2.2**, “Converting Between Numeric Types (Casting)” and **Recipe 2.3** demonstrate ways to convert between types.
- **Recipe 5.9**, “Supporting a Fluent Style of Programming” demonstrates how to return this.type from a method.
- Implicit conversions let you add new behavior to closed types like String, which is declared final in Java. They are demonstrated in **Recipe 1.10**, “Add Your Own Methods to the String Class” and **Recipe 2.1**, “Parsing a Number from a String”.
- **Recipe 6.1**, “Object Casting” demonstrates how to cast objects from one type to another.

## 19.1 Creating Classes That Use Generic Types
As a library writer, creating a class and methods that takes a generic type is similar to Java. For instance, if Scala didn't have a linked list class and you write your own, you can write:

```
class LinkedList[A] {
  private class Node[A] (elem: A) {
    var next: Node[A] = _
    override def toString = elem.toString
  }

  private var head: Node[A] = _

  def add(elem: A) {
    val n = new Node(elem)
    n.next = head
    head = n
  }

  private def printNodes(n: Node[A]) {
    if (n != null) {
      println(n)
      printNodes(n.next)
    }
  }

  def printAll() { printNodes(head) }
}
```

Notice how the generic type `A` is sprinkled throughout the class definition. This is similar to Java, but Scala uses `[A]` instead of `<T>` in Java.

To create a list of integers with this class, first create an instance of it, declaring its type as `Int`:
```
val ints = new LinkedList[Int]()
```
Then populate it with `Int` values:
```
ints.add(1)
ints.add(2)
```
Because the class uses a generic type, you can also create a `LinkedList` of `String`, or `Double`, or any other custom class.

When using generics like this, the container can take subtypes of the base type you specify in your code. For instance, given this class hierarchy:
```
trait Animal
class Dog extends Animal { override def toString = "Dog" }
class SuperDog extends Dog { override def toString = "SuperDog" }
class FunnyDog extends Dog { override def toString = "FunnyDog" }
```
You can define a `LinkedList` that holds `Dog` instances, and then add `Dog` subtypes to the list:
```
val dogs = new LinkedList[Dog]

val fido = new Dog
val wonderDog = new SuperDog
val scooby = new FunnyDog

dogs.add(fido)
dogs.add(wonderDog)
dogs.add(scooby)
```
Of course, you can add `Dog` subtypes to a `LinkedList[Dog]`. However, you might run into a problem when you define a method like this:
```
def printDogTypes(dogs: LinkedList[Dog]) {
  dogs.printAll()
}
```
You can pass your current `dogs` instance into this method, but you won't be able to pass the `superDogs` collection into `printDogTypes`:
```
val superDogs = new LinkedList[SuperDog]
superDogs.add(wonderDog)

// error: this line won't compile
printDogTypes(superDogs)
```
The last line won't compile because:
1. `makeDogsSpeak` wants a `LinkedList[Dog]`
2. `LinkedList` elements are mutable
3. `superDogs` is a `LinkedList[SuperDog]`

The compiler can't resolve this conflict. In Scala 2.10, the compiler will print out this error:
```
[error] Note: SuperDog <: Dog, but class LinkedList is invariant in type A.
[error] You may wish to define A as +A instead. (SLS 4.5)
```
This situation is discussed in detail in **Recipe 19.5**

### Type parameter symbols
If a class requires more than one type parameter, use the symbols shown in the table below.

|Symbol|Description|
|---|---|
|A| Refers to a simple type, such as List[A]|
|B, C, D| Used for the subsequent types|
|K| Typically refers to a key in a Java map. Scala collections use A in this situation|
|N| Refers to a numeric value|
|V| Typically refers to a value in a Java map. Scala collections use B in this situation|

For example, this is how Java defines an interface:
```
// from http://docs.oracle.com/javase/tutorial/java/generics/types.html
public interface Pair<K, V> {
  public K getKey();
  public V getValue();
}
```
In Scala:
```
trait Pair[A, B] {
  def getKey: A
  def getValue: B
}
```

## 19.2 Creating a Method That Takes a Simple Generic Type
Specify the generic type parameters in brackets, like `[A]`. Note that specifying the method’s return type isn’t necessary, so you can simplify the signature slightly, if desired:
```
def randomElement[A](seq: Seq[A]): A = {
  val randomNum = util.Random.nextInt(seq.length)
  seq(randomNum)
}

// change the return type from ':A =' to just '='
def randomElement[A](seq: Seq[A]) = { ...
```
This is a simple example that shows how to pass a generic collection to a method that doesn't attempt to mutate the collection.

## 19.3 Using Duck Typing (Structural Types)
Scala's version of "Duck Typing" is known as using a *structural type*.

As an example of this approach, the following code shows how a `callSpeak` method can require that its obj type parameter have a `speak()` method:
```
def callSpeak[A <: {def speak(): Unit }](obj: A) {
  // code here ...
  obj.speak()
}
```
Given that definition, an instance of any class that has a `speak()` method that takes no parameters and returns nothing can be passed as a parameter to `callSpeak`

The structural type syntax is necessary in this example because the `callSpeak` method invokes a speak method on the object that’s passed in. In a statically typed language, there must be some guarantee that the object that’s passed in will have this method, and this recipe shows the syntax for that situation.

Here's a working example:
```
class Dog { def speak() { println("woof") } }
class Klingon { def speak() { println("Qapla!") } }

object DuckTyping extends App {
  def callSpeak[A <: { def speak(): Unit }](obj: A) {
    obj.speak()
  }

  callSpeak(new Dog)
  callSpeak(new Klingon)
}
```

This won't compile because the compiler can't guarantee that the type `A` (generic) has a `speak` method:
```
// won't compile
def callSpeak[A](obj: A) {
  obj.speak()
}
```

The type parameter `A` is defined as a structural type like this:
```
[A <: { def speak(): Unit }]
```

As seen previously, `<:` means upper bound. This means that **`A` must be a subtype of a type that has a `speak` method.** The `speak` method (or function) can't take any parameters and must not return anything.

## 19.4 Make Mutable Collections Invariant
What if you want to create a collection whose elements can be mutated, and want to know how to specify the generic type parameter for its elements?

When creating a collection of elements that can be changed(mutated), its generic type parameter should be declared as `[A]`, making it *invariant*.

For instance, elements in a Scala Array or `ArrayBuffer` can be mutated, and their signatures are declared like this:
```
class Array[A]
class ArrayBuffer[A]
```
Declaring a type as invariant has several effects. First, the container can **hold both the specified types as well as its subtypes**.

Given the same code, you can define a method as follows to accept an `ArrayBuffer[Dog]`, and then have each `Dog` speak:
```
import collection.mutable.ArrayBuffer
  def makeDogsSpeak(dogs: ArrayBuffer[Dog]) {
  dogs.foreach(_.speak)
}
```
This works fine when you pass it an `ArrayBuffer[Dog]`, but not `ArrayBuffer[SuperDog]`.

This code won't compile because of the conflict built up in this situation:
- Elements in an `ArrayBuffer` can be mutated.
- `makeDogsSpeak` is defined to accept a parameter of type `ArrayBuffer[Dog]`
- You're attempting to pass in `superDogs`, whose type is `ArrayBuffer[SuperDog]`
- If the compiler allowed this, `makeDogsSpeak` could replace `SuperDog` elements in `superDogs` with plain old Dog elements. This can’t be allowed.

One of the reasons this problem occurs is that `ArrayBuffer` elements can be mutated. If you want to write a method to make all `Dog` types **and** subtypes speak, define it to accept a collection of immutable elements, such as a `List`, `Seq`, or `Vector`.

### The take-away
The elements of the `Array`, `ArrayBuffer`, and `ListBuffer` classes can be mutated, and they're all defined with invariant type parameters:
```
class Array[T]
class ArrayBuffer[A]
class ListBuffer[A]
```
Conversely, collections classes that are immutable identify their generic type parameters differently, with the `+` symbol, as shown here:
```
class List[+T]
class Vector[+A]
trait Seq[+A]
```
The `+` symbol used on the type parameters of the immutable collections defines their parameters to be *covariant*. Because their elements can't be mutated, adding this symbol makes them more flexible.
