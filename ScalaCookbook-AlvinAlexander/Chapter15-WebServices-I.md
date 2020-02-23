# Chapter 15 - Web Services - Part I

When it comes to creating your own RESTful web services, you can use the lightweight frameworks like *Scalatra* or *Unfiltered*. You can also use the *Play Framework*, *Lift Framework*, or other Scala libraries, as well as all of the previously available Java web service libraries.

In **Chapter 16**, we will discuss about Scala's support for *MongoDB*, and this chapter demonstrates how to provide a complete web services solution using Scalatra and MongoDB.

## 15.1 Creating a JSON String from a Scala Object
If you're using the Play Framework, you can use its library to work with JSON (**Recipes 15.13, 15.14**). Outside of Play, you can use these libraries that are available for Scala and Java:
- Lift-JSON
- The Google Gson library (Java)
- Json4s
- spray-json

### Lift-JSON solution
Create an empty SBT project, an configure your *build.sbt* like this:
```
name := "Lift-JSON Demo"
version := "1.0"
scalaVersion := "2.10.0"
libraryDependencies += "net.liftweb" %% "lift-json" % "2.5+"
```
In the root directory of your project, create a file named *LiftJsonTest.scala*:
```
import scala.collection.mutable._
import net.liftweb.json._
import net.liftweb.json.Serialization.write

case class Person(name: String, address: Address)
case class Address(city: String, state: String)

object LiftJsonTest extends App {

  val p = Person("Alvin Alexander", Address("Talkeetna", "AK"))

  // create a JSON string from the Person, then print it
  implicit val formats = DefaultFormats
  val jsonString = write(p)
  println(jsonString)
}
```
When you run the project with the `sbt run` command, you'll see the following JSON output:
```
{"name":"Alvin Alexander","address":{"city":"Talkeetna","state":"AK"}}
```

### Gson solution
Create an empty SBT test project, then download the Gson JAR file and place it in your project's *lib* directory.

In the root directory of your project, create a file named *GsonTest.scala*:
```
import com.google.gson.Gson

case class Person(name: String, address: Address)
case class Address(city: String, state: String)

object GsonTest extends App {
  val p = Person("Alvin Alexander", Address("Talkeetna", "AK"))

  // create a JSON string from the Person, then print it
  val gson = new Gson
  val jsonString = gson.toJson(p)
  println(jsonString)
}
```
When running `sbt run`, you'll see the same output as before.

## 15.2 Creating a JSON String from Classes That Have Collections
What if you want to generate a JSON representation of a Scala object that contains one or more collections?

In this situation, try to use the Lift-JSON domain-specific library (DSL) because it is easier.

### Lift-JSON version 1
The Lift-JSON library uses its own DSL for generating JSON output from Scala objects. The benefit of this approach is that you have complete control over the JSON that is generated.

This is an example to show how to generate a JSON string for a `Person` class that has a `friends` field defined as `List[Person]`:
```
import net.liftweb.json._
import net.liftweb.json.JsonDSL._

case class Person(name: String, address: Address) {
  var friends = List[Person]()
}

case class Address(city: String, state: String)

object LiftJsonListsVersion1 extends App {

  //import net.liftweb.json.JsonParser._
  implicit val formats = DefaultFormats

  val merc = Person("Mercedes", Address("Somewhere", "KY"))
  val mel = Person("Mel", Address("Lake Zurich", "IL"))
  val friends = List(merc, mel)
  val p = Person("Alvin Alexander", Address("Talkeetna", "AK"))
  p.friends = friends

  // define the json output
  val json =
    ("person" ->
      ("name" -> p.name) ~
      ("address" ->
        ("city" -> p.address.city) ~
        ("state" -> p.address.state)) ~
      ("friends" ->
        friends.map { f =>
          ("name" -> f.name) ~
          ("address" ->
            ("city" -> f.address.city) ~
            ("state" -> f.address.state))
        })
    )
  println(pretty(render(json)))
}
```
Lift uses a custom DSL to let you generate the JSON, and also have control over how the JSON is generated. 
