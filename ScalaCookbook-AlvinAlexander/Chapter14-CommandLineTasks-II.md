# Chapter 14 - Command Lines Tasks - Part II

## 14.10 Using Scala as a Scripting Language
Save your Scala code to a text file, making sure the first three lines of the script contain the lines shown, which will execute the script using the `scala` interpreter:
```
#!/bin/sh
exec scala "$0" "$@"
!#

println("hello, world")
```
To test this, save the code to a file named *hello.sh*, make it executable, and then run it.

- The `#!` in the first line is the usual way to start a Unix shell script. It invokes a Unix Bourne shell.
- The `exec` command is a shell built-in. `$0` expands to the name of the shell script, and `$@` expands to the positional parameters.
- The `!#` characters as the third line of the script is how the header section is closed

A great thing about using Scala in your scripts is that you can use all of Scala's advanced features, such as the ability to create and use classes in your scripts.

### Using the App trait or main method
Start the script with the usual first three header lines, and then create an object that extends the `App` trait:
```
// three lines

object Hello extends App {
  println("hello, world")
}

Hello.main(args)
```
The last line shows how to pass the script's command-line arguments to the implicit `main` method in the `Hello` object. As usual, in an `App` trait object, the arguments are available via a variable named `args`.

You can also define an object with a `main` method instead of using the `App` trait.

### Building the classpath
If your shell scripts need to rely on external dependencies (like JAR files), add them to your script's classpath like so:

```
#!/bin/sh
exec scala -classpath "lib/htmlcleaner-2.2.jar:lib/scalaemail_2.10.0-1.0.jar:lib/stockutils_2.10.0-1.0.jar" "$0" "$@"
!#
```
You can then import these classes into your code as usual.

## 14.11 Accessing Command-Line Arguments from a Script
Use the same script syntax as shown in **Recipe 14.8**, and then access the command-line arguments using `args`, which is a `List[String]` that is implicitly made available
```
#!/bin/sh
exec scala "$0" "$@"
!#

args.foreach(println)
```
Save this code to a file named *args.sh*, make the file executable, and run it like this:
```
$ ./args.sh a b c
a
b
c
```
Because the implicit field `args` is a `List[String]`, you can perform all the usual operations on it, including getting its size, and accessing elements with the usual syntax.

In a more “real-world” example, normally you’ll check for the number of command-line arguments, and then assign those arguments to values.

## 14.12 Prompting for Input from a Scala Shell Script
Use the `readLine`, `print`, `printf`, and `Console.read` methods to read user input, as demonstrated in the following script:

```
#!/bin/sh
exec scala "$0" "$@"
!#

// write some text out to the user with Console.println
Console.println("Hello")

// Console is imported by default, so it's not really needed, just use println
println("World")

// readLine lets you prompt the user and read their input as a String
val name = readLine("What's your name? ")

// readInt lets you read an Int, but you have to prompt the user manually
print("How old are you? ")
val age = readInt()

// you can also print output with printf
println(s"Your name is $name and you are $age years old.")
```
The `readLine` method lets you prompt a user for input, but the other `read*` methods don't, so you need to prompt the user manually with `print`, `println`, or `printf`.

You can also fall back and read input with the Java `Scanner` class.

### Reading Multiple Values from one line
There are several approaches to the problem, but I prefer to use the Java `Scanner` class:
```
import java.util.Scanner

// simulated input
val input = "Joe 33 200.0"

val line = new Scanner(input)
val name = line.next
val age = line.nextInt
val weight = line.nextDouble
```
To use this approach in a shell script, replace the `input` line with: `val input = readLine()`

Of course, if the input doesn't match what you expect, an error would be thrown, so you'll want to wrap this code in a `try/catch` block.

Another approach is to use one of the `readf` methods on the `Console` object. Unfortunately, they return their types as `Any`, so you have to cast them to the desired type.

Using the same example, you read the three values with the `readf3` method like this:
```
val(a, b, c) = readf3("{0} {1,number} {2,number}")
```
Variables `a`, `b` and `c` are of type `Any`, so you have to cast them like this:
```
val name = a
val age = b.asInstanceOf[Long]
val weight = c.asInstanceOf[Double]
```
or convert them:
```
val name = a.toString
val age = b.toString.toInt
val weight = c.toString.toDouble
```

A third approach is to read the values in as a `String`, and then split them into tokens.
```
scala> val input = "Michael 54 250.0"
input: String = Michael 54 250.0

scala> val tokens = input.split(" ")
tokens: Array[String] = Array(Michael, 54, 250.0)
```
A```
val name = tokens(0)
val age = tokens(1).toInt
val weight = tokens(2).toDouble
```

A fourth way is to specify a regular expression to match the input you expect to receive. You will receive each variable as a `String`, so you have to cast it to the desired type.

## 14.13 Make Your Scala Scripts Run Faster
Use the `savecompiled` argument of the Scala interpreter to save a compiled version of your script.

Change the first three header lines of your scripts to:
```
#!/bin/sh
exec scala -savecompiled "$0" "$@"
!#
```
When you run your script for the first time, Scala generates a JAR file that matches the name of your script. After running the script, on subsequent runs, Scala sees that there's a JAR file associated with the script. If the script hasn't been modified, it runs the precompiled code from the JAR file.
