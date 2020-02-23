# Chapter 14 - Command Lines Tasks - Part I

We're going to discuss command line tools such as `scalac`, `fsc`, `scaladoc`, and others.

## 14.1 Getting Started
To start the Scala REPL, type `scala` in the command line.

Inside the REPL environment, you can try all sorts of different expertiments and expressions.

You can use the *tab completion* to see methods that are available on an object.

```
scala> "foo".[Tab]
```
This will show the common methods that are available to the `String` object. If you press the Tab key again, it will expand the list to show all the available methods.  
You can also limit the list of methods by typing the first part of a method name and then pressing the Tab key.

Although the REPL tab-completion feature is good, it currently doesn’t show methods that are available to an object that results from implicit conversions. For instance, when you invoke the tab-completion feature on a String instance, the REPL doesn’t show the methods that are available to the `String` that come from the implicit conversions defined in the `StringOps` class.

To see methods available from the `StringOps` class, you currently have
to do something like this:
```
scala> val s = new collection.immutable.StringOps("")
s: scala.collection.immutable.StringOps = s

scala> s.[Tab]
```

The REPL also doesn't show method signatures currently.

The REPL is a good place to test out experiments for your code.

### REPL command-line options
You can set Java properties when starting the Scala interpreter on Unix systems by doing:
```
$ env JAVA_OPTS="-Xmx512M -Xms64M" scala
```
You can also use the `-J` command-lime argument to set parameters:
```
$ scala -J-Xms256m -J-Xmx512m
```

## 14.2 Pasting and Loading Blocks of Code into the REPL
The REPL is "greedy" and consumes the first full statement you type in, so attempting to paste blocks of code into it can fail. To solve the problem, either use the `:paste` command to paste blocks of code into the REPL or the `:load` command to load the command from a file into the REPL.

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

if (true)
  print("that was true")
else
  print("false")

[Ctrl-D]

// Exiting paste mode, now interpreting.
that was true
```

### The :load command
Assume you have the following source code in a file named *Person.scala* in the same directory where you started the REPL:
```
case class Person(name: String)
```
You can load that source code into the REPL environment like:
```
scala> :load Person.scala
Loading /path/to/class/Person.scala...
defined class Person
```
Now you can create a new `Person` instance.

**Note**: If your source code has a package declaration, the `:load` command will fail.

You can't use packages in the REPL. For situations like this, you'll need to compile your file(s) and then include them on the classpath. This will be discussed in **Recipe 14.3**

## 14.3 Adding JAR files and Classes to the REPL Classpath
IF you know that you want to use code from a JAR file when you start the REPL session, use the `cp` or `-classpath` argument to the command when you start the session. For example, this shows how to load and use *DateUtils.jar* library:

```
scala -cp DateUtils.jar

scala> import com.kevingomulia.dateutils._
import com.kevingomulia.dateutils._

scala> DateUtils.getCurrentDate
res0: String = Friday, March 21
```
If you have already started a REPL session, you can add one dynamically with the `:cp` command:

```
scala> :cp DateUtils.jar
Added '/path/to/class/DateUtils.jar'
Your new classpath is:
".:/path/to/class/DateUtils.jar"
```
Compiled class files in the current directory `(*.class)` are automatically loaded into the REPL environment, so if a simple `Person.class` file is in the current directory when you start the REPL, you can create a new `Person` instance without requiring a classpath command.

However, if your class files are in a subdirectory, you can add them to the environment when you start the session, just as with JAR files. If all the class files are located in a subdirectory named classes, you can include them by starting your REPL session like this:
```
$ scala -cp classes
```

If the class files you want to include are in several different directories, you can add them all to your classpath:
```
$ scala -cp "../Project1/bin:../Project2/classes"
```

## 14.4 Running a Shell Command from the REPL
Run the command using the `:sh` REPL command. This shows how to run the Unix `ls -al` command:
```
scala> :sh ls -al
res0: scala.tools.nsc.interpreter.ProcessResult = `ls -al` (6 lines, exit 0)

scala> res0.show
total 24
drwxr-xr-x 5 Al staff 170 Jul 14 17:14 .
drwxr-xr-x 29 Al staff 986 Jul 14 15:27 ..
-rw-r--r-- 1 Al staff 108 Jul 14 15:34 finance.csv
-rw-r--r-- 1 Al staff 469 Jul 14 15:38 process.scala
-rw-r--r-- 1 Al staff 412 Jul 14 16:24 process2.scala
```
Alternatively you can import the `scala.sys.process` package, and then use the normal `Process` and `ProcessBuilder` commands described in **Recipe 12.10**
```
scala> import sys.process._
import sys.process._

scala> "ls -al" !
total 24
drwxr-xr-x 5 Al staff 170 Jul 14 17:14 .
drwxr-xr-x 29 Al staff 986 Jul 14 15:27 ..
-rw-r--r-- 1 Al staff 108 Jul 14 15:34 finance.csv
-rw-r--r-- 1 Al staff 469 Jul 14 15:38 process.scala
-rw-r--r-- 1 Al staff 412 Jul 14 16:24 process2.scala
res0: Int = 0
```

### Scala's -i option
You can improve the REPL command loading by loading your own custom code when you start the Scala interpreter. For example. I always start the REPL in the `/Users/kevin/tmp` directory, and I have a file in that directory named *repl-commands* with these contents:
```
import sys.process._

def clear = "clear".!
def cmd(cmd: String) = cmd.!!
def ls(dir: String) { println(cmd(s"ls -al $dir")) }
def help {
  println("\n=== MY CONFIG ===")
  "cat /Users/kevin/tmp/repl-commands".!
}

case class Person(name: String)
val nums = List(1, 2, 3)
val strings = List("sundance", "rocky", "indigo")

// lets me easily see the methods from StringOps
// with tab completion
val so = new collection.immutable.StringOps("")
```

With this setup, I start the Scala interpreter with the `-i` argument, telling it to load this file when it starts:
```
$ scala -i repl-commands
```
This makes those pieces of code available to me inside the REPL.

This approach is similar to using a startup file to initialize a Unix login session, like a *.bash_profile* file for Bash users.

To make this even easier, I created the following Unix alias and put it in my *.bash_profile* file:
```
alias repl="scala -i /Users/kevin/tmp/repl-commands
```

## 14.5 Compiling with scalac and Running with scala
Though you normally use the SBT to build Scala applications, you may want to use more basic tools to compile and run small test programs, similar to using `javac` and `java` with small Java applications.

Compile programs with `scalac` and run them with `scala`.
```
$ scalac Hello.scala

$ scala Hello
Hello, world
```
Compiling and executing classes is basically the same as Java, including concepts like the classpath. For instance, if you have a class named `A` in a file named *A.scala*, it may depend on class B, in a file named *B.scala* in a directory named *classes*.

You can then compile *A* like this:
```
$ scalac -classpath classes A.scala
```

### More complicated example
you may have your source code in subdirectories under a *src* folder, one or more JAR files in a *lib* directory, and you want to compile your output class files to a *classes* folder. In this case, your files and directories will look like this:
```
./classes
./lib/DateUtils.jar
./src/com/alvinalexander/pizza/Main.scala
./src/com/alvinalexander/pizza/Pizza.scala
./src/com/alvinalexander/pizza/Topping.scala
```
The *Main.scala*, *Pizza.scala*, and *Topping.scala* files will also have package declarations corresponding to the directories they are located in, i.e.:
```
package com.alvinalexander.pizza
```
Given this config, to compile your source code files to the *classes* directory, use:
```
$ scalac -classpath lib/DateUtils.jar -d classes src/com/alvinalexander/pizza/*
```

Assuming *Main.scala* is an object that extends `App`, *Pizza.scala* is a regular class file, and *Topping.scala* is a case class, your *classes* directory will contain these files after your `scalac` command:
```
./classes/com/alvinalexander/pizza/Main$.class
./classes/com/alvinalexander/pizza/Main$delayedInit$body.class
./classes/com/alvinalexander/pizza/Main.class
./classes/com/alvinalexander/pizza/Pizza.class
./classes/com/alvinalexander/pizza/Topping$.class
./classes/com/alvinalexander/pizza/Topping.class
```

Now, you can run the application like this:
```
$ scala -classpath classes:lib/DateUtils.jar com.alvinalexander.pizza.Main
```
As you can see, the more classes and libraries you have, it is more recommended to use a tool like SBT, Maven, or Ant.

## 14.6 Disassembling and Decompiling Scala Code
You can use several different approaches to see how your Scala source code is translated:
- Use the `javap` command to disassemble a *.class* file to look at its signature
- Use `scalac` options to see how the compiler converts your Scala source code to Java code
- Use a decompiler to convert your classes back to Java source code

### Using javap
Since Scala code is compiled into regular Java class files , you can use the `javap` command to disassemble them.

```
// Person.scala file
class Person (var name: String, var age: Int)

// Compile Person.scala
$ scalac Person.scala

$ javap Person
Compiled from "Person.scala"
public class Person extends java.lang.Object implements scala.ScalaObject{
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public int age();
  public void age_$eq(int);
  public Person(java.lang.String, int);
}
```
This shows the signature of the `Person` class, which is its public API, or interface. You can see the Scala compiler created methods like `name()`, `name_$eq`, `age()`, and `age_$eq` methods for you.

### Using scalac print options
You can also use the "print" options available with the `scalac` command. These are demonstrated in detail in **Recipe 3.1**

### Using a decompiler
You can use a tool like JAD or Java Decompiler Project to see how your Scala source code is converted into Java source code.

Disassembling class files can be helpful to understand how Scala works.

## 14.7 Finding Scala Libraries
In Scala, once you've found a tool you want to use, you usually add it as a dependency to your project with SBT. For instance, to include libraries into your project, such as ScalaTest or Mockito, add these lines to your SBT *build.sbt* file:
```
resolvers += "Typesafe Repository" at
  "http://repo.typesafe.com/typesafe/releases/"

libraryDependencies ++= Seq(
  "org.scalatest" %% "scalatest" % "1.8" % "test",
  "org.mockito" % "mockito-core" % "1.9.0" % "test"
)
```

## 14.8 Generating Documentation with scaladoc
You have annotated your Scala code with Scaladoc and now you want to generate developer documentation for your API.

To generate Scaladoc API documentation, document your code using Scaladoc tags, and then create the documentation using an SBT task or the `scaladoc` command. You can mark up your source code using Scaladoc tags, as well as a wiki-like syntax.

[Refer to the wiki or book for examples]

You can generate the Scaladoc documentation with the Scaladoc command:
```
$ scaladoc Person.scala
```
This generates a root *index.html* file and other related files for your API documentation.

If you are using SBT, run this command in the **root directory** of your project:
```
$ sbt doc
```
This generates the same API documentation, and places it under the *target* directory of your SBT project. With Scala 2.10 and SBT 0.12.3, the root file is located at *target/scala-2.10/api/index.html*.

Here are some common Scaladoc tags
- `@author`: Author of the class - Multiple tags are allowed
- `@constructor`: Documentation for the constructor - Only one tag
- `@example`: Provide an example of how to use a method/constructor - Multiple tags allowed
- `@note`: Document pre- and post- conditions, and other requirements - Multiple tags
- `@param`: Document a method/constructor parameter - One per param
- `@return`: Document the return value of a method - One tag
- `@see`: Describe other sources of related information - Multiple tags
- `@since`: Used to indicate that a member has been available since a certain version release - One tag
- `@todo`: Document any to-do items for a method/class - Multiple
- `@throws`: Document an exception type that can be thrown by a method/constructor - Multiple
- `@version`: The version of a class - One tag

There are other formatting markup and other Scaladoc tags and annotations that you can find in more detail in the Scala wiki, as well as the Wiki markup tags.

## 14.9 Faster Command-Line Compiling with fsc
Let's say you are making changes to a project and recompiling it with `scalac`, and you'd like to reduce the compile time.

You can use the `fsc` command instead of `scalac` to compile your code:
```
$ fsc *.scala
```
The `fsc` command works by starting a compilation daemon and also maintains a cache, so compilation attempts after the first attempt run much faster than `scalac`.

Beware of a few caveats of the `fsc` command:
- “The tool is especially effective when repeatedly compiling with the same class paths, because the compilation daemon can reuse a compiler instance.”
- “The compilation daemon is smart enough to flush its cached compiler when the class path changes. However, if the contents of the class path change, for example due to upgrading a library, then the daemon should be explicitly shut down with `-shutdown`.

As an example of the second point, if the JAR files on the classpath have changed, you should shut down the daemon, and then reissue your `fsc` command:
```
$ fsc -shutdown
[compile server exited]

$ fsc *.scala
```
