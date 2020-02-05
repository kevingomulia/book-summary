## 12.10 Executing External Commands
Let's say we are not concerned about the output from the command, but are interested in the exit code.

To execute external commands, use the methods of the `scala.sys.process` package.

There are 3 primary ways to execute external commands:
- Use the `!` method to execute the command and get its exit status
- Use the `!!` method to execute the command and get its output
- Use the `lines` method to execute the command in the background and get its result as a `Stream`

To execute a command and get its exit status, import the necessary members and run the desired command with the `!` method:
```
scala> import sys.process._

scala> "ls -al".!
...
res0: Int = 0    <--- this is the exit code
```

In a Scala application, it can be called like this:
```
def playSoundFile(filename: String): Int = {
  val cmd = "afplay " + filename
  val exitCode = cmd.!
  exitCode
}
```
To execute system commands I generally just use ! after a `String`, but the `Seq` approach is also useful. The first element in the `Seq` should be the name of the command you want to run, and subsequent elements are considered to be arguments to it.
```
val exitCode = Seq("ls", "-a", "-l").!
val exitCode = Seq("ls", "-a", "-l", "/tmp").!
```
You can also create a Process object to execute an external command, if you prefer:
```
val exitCode = Process("ls").!
```
When running these commands, be aware of whitespace around your command and arguments.

### Using the lines method
The `lines` method is an alternative to the `!` and `!!` commands. With `lines`, you can execute a command in the background.
```
val process = Process("find / -print").lines
```
The `lines` method throws an exception if the exit status of the command is nonzero. If you want to retrieve the stderr from the command, use the `lines_!` method instead.

## 12.11 Executing External Commands and Using STDOUT
