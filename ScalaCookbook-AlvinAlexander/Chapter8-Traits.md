# Chapter 8 - Traits

A Scala trait is similar to a Java interface. Scala classes can extend multiple traits, which is done with the `extends` and `with` keywords.

```
class WoodPecker extends Bird with TreeScaling with Pecking
```
Just like abstract methods in Java, they can also have implemented methods. However, unlike Java's abstract classes, you can mix more than one trait into a class, and a trait can also control what classes it can be mixed into.

## 8.1 Using a Trait as an Interface
You can use a trait just like a Java interface. As with interfaces, just declare the methods in your trait that you want extending classes to implement:

```
trait BaseSoundPlayer {
  def play
  def close
  def pause
  def stop
  def resume()
}
```
If the methods don't take any argument, you only need to declare the names of the methods after the `def` keyword, as shown.

When a class extends a trait, it uses the `extends` keyword. When it extends multiple traits, use the `extends` and `with` keywords.

Unless the class implementing a trait is an abstract class, the class must implement all of the abstract trait methods.

A trait can extend another trait.

If a class extends a class (or abstract class) and a trait, always use `extends` before the class name, and use `with` before the trait name(s).

## 8.2 Using Abstract and Concrete Fields in Traits
You want to put abstract or concrete fields in your traits so they are declared in one place and available to all types that implement the trait.

Define a field with an initial value to make it *concrete*. Don't assign it an initial value to make it *abstract*.

```
trait PizzaTrait {
  var numToppings: Int    // abstract
  var size = 14           // concrete
  val maxNumToppings = 10 // concrete
}
```
In the class that extends the trait, you'll need to define the values for the abstract fields, or make the class abstract. There's no need for the `override` keyword to override a `var` field in a subclass or a trait. You need to use it to override a `val` field.

## 8.3 Using a Trait Like an Abstract Class
You can define methods in your trait like regular Scala methods. In the class that extends the trait, you can override those methods if you want to.

Although Scala has abstract classes, it is much more common to use traits than abstract classes to implement base behaviour. A class can extend only one abstract class, but it can implement multiple traits.

## 8.4 Using Traits as Simple Mixins
You want to design a solution where multiple traits can be mixed into a class to provide a robust design.

To implement a simple *mixin*, define the methods you want in your trait, then add the trait to your class. You can mix traits with abstract classes as well.

## 8.5 Limiting Which Classes Can Use a Trait by Inheritance
Let's say you want to limit a trait so it can only be added to classes that extend a superclass or another trait.

You can use the following syntax to declare a trait named `TraitName`, where it can only be mixed into classes that extend a type named `SuperThing`. `SuperThing` may be a trait, class, or abstract class:

```
trait [TraitName] extends [SuperThing]
```

For instance, in the following example, `Starship` and `StarfleetWarpCore` both extend the common superclass `StarfleetComponent`, so the `StarfleetWarpCore` trait can be mixed into the `Starship` class:

```
class StarfleetComponent
trait StarfleetWarpCore extends StarfleetComponent
class Starship extends StarfleetComponent with StarfleetWarpCore
```
However, in the following example, the `Warbird` class can’t extend the `StarfleetWarpCore` trait, because Warbird and `StarfleetWarpCore` don’t share the same superclass:

```
class StarfleetComponent
trait StarfleetWarpCore extends StarfleetComponent
class RomulanStuff

// won't compile
class Warbird extends RomulanStuff with StarfleetWarpCore
```

Attempting to compile the second example yields this error:
```
error: illegal inheritance; superclass RomulanStuff
  is not a subclass of the superclass StarfleetComponent
  of the mixin trait StarfleetWarpCore
class Warbird extends RomulanStuff with StarfleetWarpCore
                                        ^
```
A trait inheriting from a class is not a common occurrence, Recipes 8.6 and 8.7 are more commonly used to limit the classes a trait can be mixed into.

As long as a class and a trait share the same superclass, the code will compile. However, if the superclasses are different, the code will not compile.

## 8.6 Marking Traits So They Can Only Be Used by Subclasses of a Certain Type
To make sure a trait named `MyTrait` can only be mixed into a class that is a subclass of a type named `BaseType`, begin your trait with a `this: BaseType =>` declaration, like this:
```
trait StarfleetWarpCore {
  this: Starship =>
}
```
This code will work:
```
class Enterprise extends Starship with StarfleetWarpCore
```
But this will fail:
```
class Warbird extends RomulanShip with StarfleetWarpCore
```
This approach is referred to as a *self type*.

## 8.7 Ensuring a Trait Can Only Be Added to a Type That Has a Specific Method
You only want to allow a trait to be mixed into a type that has a method with a given signature.

You can use a variation of the self-type syntax that lets you declare that any class that attempts to mix in the trait must implement the method you specify.

For example, the `WarpCore` trait requires that any class that attempt to mix it must have an `ejectWarpCore` method:

```
trait WarpCore {
  this: { def ejectWarpCore(password: String): Boolean } =>
}
```
It further states that the `ejectWarpCore` method must accept a `String` argument and return a `Boolean` value.

For a trait to require that a class have multiple methods, just add the additional method signatures inside the `this` block.

This approach is known as a *structural* type.

## 8.8 Adding a Trait to an Object Instance
Rather than add a trait to entire class, you just want to add a trait to an object instance when the object is created.

You can add the trait to the object when you construct it. For example:
```
class DavidBanner

trait Angry {
  println("Angry")
}

object Test extends App {
  val hulk = new DavidBanner with Angry
}
```

## 8.9 Extending a Java Interface Like a Trait
In your Scala application, you can simply use the `extends` and `with` keywords to implement your Java interfaces as though they were Scala traits.
