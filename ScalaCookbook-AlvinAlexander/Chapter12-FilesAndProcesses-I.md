# Chapter 12 - Files and Processes

## 12.1 How to Open and Read a Text File
There are two primary ways to open and read a text file:
1. Use a concise, one-line syntax. This has the side effect of leaving the file open, but can be useful in short-lived programs, like shell scripts
2. Use a slightly longer approach that properly closes the file.

### Use the concise syntax
To handle each line in the file as it's read, use this approach:
```
import scala.io.Source

val filename = "file.scala"
for (line <- Source.fromFile(filename).getLines) {
  println(line)
}
```
The `fromFile` method returns a `BufferedSource`, and its `getLines` method treats any of `\r\n`, `\r`, or `\n` as a line separator.

To get all of the lines from the file as one `String`:
```
val fileContents = Source.fromFile(filename).getLines.mkString
```
This approach has the side effect of leaving the file open as long as the JVM is running, but for short-lived shell scripts, this shouldn’t be an issue; the file is closed when the JVM shuts down.

### Properly closing the file
```
val bufferedSource = Source.fromFile("example.txt")
for (line <- bufferedSource.getLines) {
  println(line.toUpperCase)
}
bufferedSource.close // to properly close it
```

The `getLines` method of the `Source` class returns a `scala.collection.Iterator`. The iterator returns each line without any newline characters. An iterator has many methods for working with a collection, and it works well with the `for loop`, as shown.

When working with files and other resources that need to be properly closed, it's best to use the "Loan Pattern".

Loan Pattern as the name suggests would loan a resource to your function. So if you break out the sentence. It would:

- Create a resource which you can use
- Loan the resources to the function which would use it
- This function would be passed by the caller
- The resource would be destroyed

In Scala, you can achieve this with a `try/finally` clause:
```
def using[A](r : Resource)(f : Resource => A) : A =
  try {
    f(r)
  } finally {
    r.dispose()
  }
```

You can generate exceptions any time you try to open a file, and if you want to handle your exceptions, use the `try/catch` syntax:
```
import scala.io.Source
import java.io.{FileNotFoundException, IOException}

val filename = "no-such-file.scala"
try {
  for (line <- Source.fromFile(filename).getLines) {
    println(line)
  }
} catch {
  case e: FileNotFoundException => println("Couldn't find that file.")
  case e: IOException => println("Got an IOException!")
}
```
## 12.2 Writing Text Files
Scala doesn't offer any special file writing capability, so we need to use the Java `PrintWriter` or `FileWriter` approaches.
```
// PrintWriter
import java.io._
val pw = new PrintWriter(new File("hello.txt" ))
pw.write("Hello, world")
pw.close

// FileWriter
val file = new File(canonicalFilename)
val bw = new BufferedWriter(new FileWriter(file))
bw.write(text)
bw.close()
```
There are a few differences between the two Writer classes. For example, `FileWriter` throws `IOException`, whereas `PrintWriter` doesn't throw exceptions, and instead sets Boolean flags that can be checked.

Check their Javadoc for more information

## 12.3 Reading and Writing Binary Files
Scala doesn't offer any special conveniences for reading or writing binary files, so use the Java `FileInputStream` and `FileOutputStream` classes.

To demonstrate this, the following code is a Scala translation of the `CopyBytes` class:
```
import java.io._

object CopyBytes extends App {

  var in = None: Option[FileInputStream]
  var out = None: Option[FileOutputStream]

  try {
    in = Some(new FileInputStream("/tmp/Test.class"))
    out = Some(new FileOutputStream("/tmp/Test.class.copy"))
    var c = 0
    while ({c = in.get.read; c != −1}) {
      out.get.write(c)
    }
  } catch {
    case e: IOException => e.printStackTrace
  } finally {
    println("entered finally ...")
    if (in.isDefined) in.get.close
    if (out.isDefined) out.get.close
  }
}
```
In this code, in and out are populated in the `try` clause. It’s safe to call `in.get` and `out.get` in the while loop, because if an exception had occurred, flow control would have switched to the `catch` clause, and then the `finally` clause before leaving the method.

## 12.4 How to Process Every Character in a Text File
If performance isn't a concern:
```
val source = io.Source.fromFile("/Users/Al/.bash_profile")
for (char <- source) {
  println(char.toUpper) // OR DO OTHER STUFF HERE
}
source.close
```
This code may be slow on large files. The time taken can be significantly reduced by using the `getLines` method to retrieve one line at a time, and then working through the characters in each line. Like so:

```
def countLines2(source: io.Source): Long = {
  val NEWLINE = 10
  var newlineCount = 0L
  for {
    line <- source.getLines
    c <- line
    if c.toByte == NEWLINE
  } newlineCount += 1
  newlineCount
}
```

## 12.5 How to Process a CSV File
Given a CSV like this named *finance.csv*:
```
January, 10000.00, 9000.00, 1000.00
February, 11000.00, 9500.00, 1500.00
March, 12000.00, 10000.00, 2000.00
```

You can process the lines in the file with the following:
```
object CSVDemo extends App {

  println("Month, Income, Expenses, Profit")
  val bufferedSource = io.Source.fromFile("/tmp/finance.csv")
  for (line <- bufferedSource.getLines) {
    val cols = line.split(",").map(_.trim)
    // do whatever you want with the columns here
    println(s"${cols(0)}|${cols(1)}|${cols(2)}|${cols(3)}")
  }
  bufferedSource.close
}
```
The magic is in this line:
```
val cols = line.split(",").map(_.trim)
```
The resulting output:
```
January|10000.00|9000.00|1000.00
February|11000.00|9500.00|1500.00
March|12000.00|10000.00|2000.00
```

If you prefer named variables, you can change the loop to look like this:
```
for (line <- bufferedSource.getLines) {
  val Array(month, revenue, expenses, profit) = line.split(",").map(_.trim)
  println(s"$month $revenue $expenses $profit")
}
```
There are a few other ways to deal with a CSV file as shown in the book. Check it out for more details.

## 12.6 Pretending that String Is a File
This is more for the purpose of testing.

Because `Scala.fromFile` and `Scala.fromString` both extend `scala.io.Source`, they are easily interchangeable. As long as your method takes a `Source` reference, you can pass it the `BufferedSource` you get from calling `Source.fromFile`, or the `Source` you get from calling `Source.fromString`.

For example, this takes a `Source` object and prints the lines it contains:
```
import io.Source

def printLines(source: Source) {
  for (line <- source.getLines) {
    println(line)
  }
}
```
It can be called when the source is constructed from a String:
```
val s = Source.fromString("foo\nbar\n")
printLines(s)
```
It can also be called when the source is a file:
```
val f = Source.fromFile("/Users/Al/.bash_profile")
printLines(f)
```

## 12.7 Using Serialization
To make a Scala class serializable, extend the `Serializable` trait and add the `@SerialVersionUID` annotation to the class:
```
@SerialVersionUID(100L)
class Stock(var symbol: String, var price: BigDecimal) extends Serializable {
  // code here
}
```
After marking the class serializable, use the same techniques to write and read the objects as you did in Java, including the Java “deep copy” technique that uses serialization.


## 12.8 Listing Files in a Directory
Scala doesn't offer any different methods for working with directories, so use the `listFiles` method of the Java `File` class.

This method creates a list of all files in a directory:
```
def getListOfFiles(dir: String):List[File] = {
  val d = new File(dir)
  if (d.exists && d.isDirectory) {
    d.listFiles.filter(_.isFile).toList
  } else {
    List[File]()
  }
}
```
If you want to limit the list of files based on their extension, in Java, you'd implement a `FileFilter` with an `accept` method. In Scala, you can write the equivalent code without requiring a `FileFilter`.

Assuming the `file` you're given represents a directory that is known to exist, the following method shows how to filter a set of files based on their extension:

```
import java.io.File

def getListOfFiles(dir: File, extensions: List[String]): List[File] =
{
  dir.listFiles.filter(_.isFile).toList.filter { file =>
    extensions.exists(file.getName.endsWith(_))
  }
}
```
You can call this method to list all WAV and MP3 files in a given directory like this:
```
val okFileExtensions = List("wav", "mp3")
val files = getListOfFiles(new File("/tmp"), okFileExtensions)
```

## 12.9 List Subdirectories in a Directory
Use a combination of the Java `File` class and Scala collection methods:
```
def getListOfSubDirectories(dir: File): List[String] =
  dir.listFiles
     .filter(_.isDirectory)
     .map(_.getName)
     .toList
```
This algorithm does the following:
- Uses the `listFiles` method of the `File` class to list all the files in the given directory as an `Array[File].`
- The `filter` method trims that list to contain only directories
- `map` calls `getName` on each file to return an array of directory names (instead of `File` instances)
- `toList` converts that to a `List[String]`

This problem provides a good way to demonstrate the differences between writing code in a functional style versus in an imperative style.
