# Chapter 19 - Types

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
You can pass your current `dogs` instance into this method, but you won't be able to pass the `superDogs` collection into it.
