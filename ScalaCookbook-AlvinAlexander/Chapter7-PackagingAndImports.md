# Chapter 7 - Packaging and Imports

In Scala, two packages are implicitly imported for you:
1. `java.lang._`
2. `scala._`
3. All members from the `scala.Predef` object

## 7.1 Packaging with the Curly Braces Style Notation
You can wrap one or more classes in a set of curly braces with a package name, as shown:
```
package com.acme.store {
  class Foo { override def toString = "com.acme.store.Foo" }
}
```
You can create Scala packages with the usual Java practice of declaring a package name at the top of the file.

## 7.2 Importing One or More Members
This is the syntax for importing one class:
```
import java.io.File
```
You can import multiple classes the Java way:
```
import java.io.File
import java.io.IOException
import java.io.FileNotFoundException
```
or the Scala way:
```
import java.io.{File, IOException, FileNotFoundException}
```
Syntactically, the two big differences between Java and Scala are: the curly brace syntax (import selector clause, more at 7.3 and 7.4), and the use of the `_` wildcard instead of Java's `*`.

Also, with Scala, you can:
- Place import statements anywhere
- Import classes, packages, or objects
- Hide and rename members when you import them

### Placing import statements anywhere
In Scala you can place import statements anywhere. For instance, because Scala makes it easy to include multiple classes in the same file, you may want to separate your import statements so the common imports are declared at the top of the file, and the imports specific to each class are within each class specification:
```
package foo
import java.io.File
import java.io.PrintWriter

class Foo {
  import javax.swing.JFrame // only visible in this class
  // ...
}

class Bar {
  import scala.util.Random // only visible in this class
  // ...
}
```

## 7.3 Renaming Members on Import
You want to rename members when you import them to help avoid namespace collisions or confusion.

You can give the class you're importing a new name when you import it like this:
```
import java.util.{ArrayList => JavaList}
                  original  => renamed
```
Then, you can refer to the class like:
```
val list = new JavaList[String]
val list = new ArrayList[String] // This doesn't work anymore
```

## 7.4 Hiding a Class During the Import Process
Use the renaming syntax shown in 7.3, but rename it to the `_` wildcard character.

This ability to hide members on import is useful when you need many members from one package, and therefore want to use the _ wildcard syntax, but you also want to hide one or more members during the import process, typically due to naming conflicts.

## 7.5 Using Static Imports
You want to import members in a way similar to the Java static import approach, so you can refer to the member names directly, without having to prefix them with their class name.

You can use the wildcard syntax to do this. For example:
```
import java.lang.Math._

// Now you can call PI without using Math.PI
val a = cos(PI)
```

Enumerations are another great candidate for this technique.

## 7.6 Using Import Statements Anywhere
You can place an import statement almost anywhere inside a program to limit the scope of an import. You can:
- import members inside a class
- place import inside a method
- place import statement inside a block

Import statements are read in the order of the file, so where you place them in a file is important.
