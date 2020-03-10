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
