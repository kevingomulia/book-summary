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
Use the `!!` method to execute the command and get the STDOUT from the resulting process as a `String`. The `!!` returns the STDOUT from the command rather than the exit code of the command. This returns a multiline string, which you can process in your application.

```
val result = "ls -al" !!  // stores the entire result of ls -al
```
You can do the same thing with a `Process` or `Seq` instead of a `String`:
```
val result = Process("ls -al").!!
val result = Seq("ls -al").!!
```
You can use `Seq` to execute a system command that requires arguments, as shown above.

Beware that attempting to get the standard output from a command exposes you to exceptions that can occur. If your command throws an exception, the `result` is not assigned.

You might get unexpected newline characters when running the command. In that case, just trim the result.

### Using the lines_! method
You may want to check to see whether an executable program is available on your system. For instance, suppose you wanted to know whether the hadoop2 executable is available on a Unix-based system. A simple way to handle this situation is to use the Unix `which` command with the `!` method, where a nonzero exit code indicates that the command isn’t available:
```
scala> val executable = "which hadoop2".!
executable: Int = 1
```
If the value is nonzero, you know that the executable is not available on the current system. More accurately, it may be on the system, but it’s not on the PATH.

Another way to handle this is to use the `lines_!` method. This can be used to return a `Some` or `None`, depending on whether the `hadoop` command is found by `which`.

```
// Not found
scala> val executable = "which hadoop2".lines_!.headOption
executable: Option[String] = None

// Found
scala> val executable = "which ls".lines_!.headOption
executable: Option[String] = Some(/bin/ls)
```
You call the `headOption` method because the `lines_!` method returns a `Stream`, but you want the `Option` immediately.

## 12.12 Handling STDOUT and STDERR for External Commands
The simplest way to do this is to run your commands as shown in previous recipes, and then capture the output with a `ProcessLogger`.

```
#!/bin/sh
exec scala "$0" "$@"
!#

import sys.process._
val stdout = new StringBuilder
val stderr = new StringBuilder

val status = "ls -al FRED" ! ProcessLogger(stdout append _, stderr append _)

println(status)
println("stdout: " + stdout)
println("stderr: " + stderr)
```
When this script is run, the `status` variable contains the exit status of the command. The `stdout` variable contains the STDOUT if the command is successful, and `stderr` contains the STDERR if there are problems.

## 12.13 Building a Pipeline of Commands
You can use the `#|` method to pipe the output from one command into the input stream of another command. When doing this, use `!` at the end of the pipeline if you want the exit code of the pipeline, or `!!` if you want the output.

For example, this code pipes the output from the `ps` command into a line count command `(wc -l)`:
```
val javaProcs = ("ps auxw" #| "grep java").!!.trim
```
Note that attempting to pipe commands together inside a `String` and then executing it with `!` won't work:
```
// won't work
val result = "ls -al | grep Foo").!!
```
To execute a series of commands in a shell, such as the `Bourne` shell, use a `Seq` with multiple parameters, like so:
```
val r = Seq("/bin/sh", "-c", "ls | gre .scala").!!
```
This approach runs the `ls | grep .scala` command inside a Bourne shell instance. However, when using `!!`, you'll get an exception if there are no *.scala* files in the directory.

It is best to wrap commands executed with `!!` in a `try/catch` block.

## 12.14 Redirecting the STDOUT and STDIN of External Commands
Use `#>` to redirect `STDOUT` and `#<` to redirect `STDIN`.

```
import sys.process._
import java.io.File

("ls -al" #> new File("files.txt")).!
("ps aux" #> new File("processes.txt")).!
```
You can also pipe commands together and then write the resulting output to a file:
```
("ps aux" #| "grep http" #> new File("http-processes.out")).!
```
The Scala process commands are consistent with the standard Unix redirection symbols, so you can also append to a file with the `#>>` method.

## 12.15 Using AND (&&) and OR (||) with Processes
Again, use the Scala operators `#&&` and `#!!`.

```
val result = ("ls temp" #&& "rm temp" #|| "echo 'temp' not found").!!.trim
```
> Run the ls command on the file temp, and if it's found, remove it. Otherwise, print the "not found" message

## 12.16 Handling Wildcard Characters in External Commands
Run your command while invoking a Unix shell, otherwise the command will fail.
```
scala> val status = Seq("/bin/sh", "-c", "ls *.scala").!
status: Int = 0
```
This won't work:
```
scala> "ls *.scala".!
res0: Int = 1
```
Putting a shell wildcard character like `*` into a command doesn't work because the `*` needs to be interpreted and expanded by a shell.

An important part of this recipe is using the `-c` argument of the */bin/sh* command. The `-c` option causes the commands to be read from string.

As an exception to this general rule, the `-name` option of the `find` command may work because it treats the `*` character as a wildcard character itself.

## 12.17 How to Run a Process in a Different Directory
Use one of the `Process` factory methods, setting your command and the desired directory, then running the process with the usual `!` or `!!` commands. The following example runs the `ls` command with the `-al` arguments in the */var/tmp* directory:

```
import sys.process._
import java.io.File

object Test extends App {
  val output = Process("ls -al", new File("/tmp")).!!
  println(output)
}
```

## 12.18 Setting Environment Variables When Running Commands
Specify the environment variables when calling a `Process` factory method (an `apply` method in the `Process` object).

The following example shows how to run a shell script in a directory named */home/al/bin* while also setting the `PATH` environment variable:

```
val p = Process("runFoo.sh",
        new File("/Users/Al/bin"),
        "PATH" -> ".:/usr/bin:/opt/scala/bin")

val output = p.!!
```
To set multiple environment variables at once, keep adding them at the end of the `Process` constructor:

```
val output = Process("env",
                      None,
                      "VAR1" -> "foo",
                      "VAR2" -> "bar")
```

## 12.19 An Index of Methods to Execute External Commands
In summary, the following tables list the methods of the `scala.sys.process` package that you can use when running external (system) commands.

This table lists the methods that you can use to execute system commands.

| Method | Description |
| --- | --- |
| ! | Runs the command and return its exit code. Blocks until all external commands exit. If used in a chain, returns the exit code of the last command in the chain |
| !! | Runs the command (or command pipe/chain) and returns the output from the command as a `String`. Blocks until all external commands exit. Throws exceptions when the command's exit status in non-0 |
| run | Return a `Process`  object immediately while running the process in the background. The `Process` can't currently be polled to see if it has completed |
| lines | Returns immediately, while running the process in the background. The output that's generated is provided through a `Stream[String]`. Getting the next element of the `Stream` may block until it becomes available. Throws an exception if the return code is non-0. |
| lines_! | Like the `lines` method, but STDERR output is sent to the `ProcessLogger` you provide. |

This table lists the methods you can use to redirect STDIN and STDOUT when external commands are executed

| Method | Description |
| --- | --- |
| #< | Read from STDIN |
| #> | Write to STDOUT |
| #>> | Append to STDOUT |

And this table lists the methods that you can use to combine (pipe) external commands.

| Method | Description |
| --- | --- |
| cmd1 #\| cmd2 | The output of the first command is used as input to the second command, like a Unix shell pipe. |
| cmd1 ### cmd2 | cmd1 and cmd2 will be executed in sequence, one after the other. This is like the Unix ; operator, but ; is a reserved keyword in Scala. |
| cmd1 #> cmd2 | Normally used to write to STDOUT but can be used like #\| to chain commands together |
| cmd1 #&& cmd2 | Run cmd2 if cmd1 runs successfully (i.e., it has an exit status of 0). |
| cmd1 #\|\| cmd2 | Run cmd2 if cmd1 has an unsuccessful (nonzero) exit status. |
| cmd1 #&& cmd2 #\|\| cmd3 | Run cmd2 is cmd1 has a successful exit status, otherwise, run cmd3 |
