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
