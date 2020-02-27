# Chapter 18 - SBT

SBT is the de facto build tool for Scala applications. SBT uses the same directory structure as Maven, and it also uses a "convention over configuration" approach. Under the covers, SBT's dependency management system is handled by Apache Ivy. This means that Java projects that have been created and packaged for use with Maven can easily be used by SBT.

Other JAR files not in an Ivy/Maven repository can simply be placed in your project's *lib* folder, and SBT will automatically find them.  

Examples shown here are tested with SBT version 0.12.3.

## 18.1 Creating a Project Directory Structure for SBT
SBT doesn't include a command to create a new project.

You can use either a shell script or a tool like *Giter8* to create your project's directory structure.

### Use a shell script
Since SBT use the same directory structure as Maven, for simple needs you can generate a compatible structure using a shell script. For example, the following Unix shell script creates the initial set of files and directories you'll want for most projects:
```
#!/bin/sh
mkdir -p src/{main,test}/{java,resources,scala}
mkdir lib project target

# create an initial build sbt file
echo 'name := "MyProject"

version := "1.0"

scalaVersion := "2.10.0"' > build.sbt
```
Save that code, make it executable, and run it inside a new project directory to create all the subdirectories SBT needs, as well as an initial *build.sbt* file.

### Using Giter8
Giter8 is based on a template system, and the templates usually contain everything you need to create a skeleton SBT project that's preconfigured to use one or more Scala tools, e.g. ScalaTest, Scalatra, etc.

As a demonstration, the following example shows how you to use the `scalatra/scalatra-sbt` template.
To create a project named "NewApp", Giter8 prompts you with a series of question, and then creates a *newapp* folder for you project.

To demonstrate this, move to a directory where you normally create your projects, then start the Giter8 with the `g8` command:
```
/Users/Al/Projects> g8 scalatra/scalatra-sbt

organization [com.example]: com.alvinalexander
package [com.example.app]: com.alvinalexander.newapp
name [My Scalatra Web App]: NewApp
scalatra_version [2.2.0]:
servlet_name [MyScalatraServlet]: NewAppServlet
scala_version [2.10.0]:
version [0.1.0-SNAPSHOT]:

Template applied in ./newapp
```
In this example, Giter8 creates all the configuration files and Scalatra stub files you need to have a skeleton Scalatra project up and running.

Giter8 works by downloading project templates from GitHub. Giter8 requires SBT and Conscript to work.

In case you face an error `Unable to find github repository: ...` when running g8, simply upgrade Conscript and Giter8 to their latest versions.

## 18.2 Compiling, Running, and Packaging a Scala Project with SBT
Create a directory layout to match what SBT expects, then run `sbt compile` to compile your project, `sbt run` to run your project, and `sbt package` to package your project as a JAR file.

Unlike Java, in SCala, the file's package name doesn't have to match the directory name. From the root directory of the project, you can compile, run, and package the project.

Description of the most common SBT commands:
|Command|Description|
|---|---|
|clean|Removes all generated files from the *target* directory|
|compile|Compiles source code that are in *src/main/scala*, *src/main/java*, and the root directory of the project|
|~ compile|Automatically recompiles source code files while you're running SBT in interactive mode|
|console|Compiles the source code files in the project, puts them on the classpath, and starts the Scala REPL|
|doc|Generates API documentation from your Scala source code using `scaladoc`|
|help <command>|Issued by itself, the `help` command lists the common commands that are currently available. When given a command, `help` provides a description of that command.|
|inspect <setting>|Displays information about <setting>. For instance, `inspect library-dependencies` displays information about the `libraryDependencies` setting.|
|package|Creates a JAR file (or WAR file for web projects) containing the files in *src/main/scala*, *src/main/java*, and resources in *src/main/resources*|
|package-doc|Creates a JAR file containing documentation generated from your Scala source code|
|publish|Publishes your project to a remote repository|
|publish-local|Publishes your project to a local Ivy repository|
|reload|Reloads the build definition files (build.sbt, project/*.scala, and project/*.sbt), which is necessary if you change them while you’re in an interactive SBT session.|
|run|Compiles your code, and runs the `main` class from your project.|
|test|Compiles and run all tests|
|update|Updates external dependencies|

### The last command
The `last` command prints logging info for the last command that was executed. This can help you understand what’s happening, including understanding why something is being recompiled over and over when using incremental compilation.

## 18.3 Running Tests with SBT and ScalaTest
Create a new SBT project directory structure as shown in **Recipe 18.1**, and then add the ScalaTest library dependency to your *build.sbt* file.

Add your Scala source code under the *src/main/scala* folder, add your tests under the *src/test/scala* folder, and then run the tests with the SBT `test` command.

Here's an example of a test file named *HelloTests.scala* in the *src/test/scala* directory of your project:
```
package com.alvinalexander.testproject

import org.scalatest.FunSuite

class HelloTests extends FunSuite {
  test("the name is set correctly in constructor") {
    val p = Person("Barney Rubble")
    assert(p.name == "Barney Rubble")
  }

  test("a Person's name can be changed") {
    val p = Person("Chad Johnson")
    p.name = "Ochocinco"
    assert(p.name == "Ochocinco")
  }
}
```

## 18.4 Managing Dependencies with SBT
Let's say you want to use one or more external libraries in your Scala/SBT projects.

You can use both managed and unmanaged dependencies in your SBT projects.

If you have JAR files (*unmanaged* dependencies) that you want to use in your project, simply copy them to the *lib* folder in the root directory and SBT will find them automatically. If those JARs depend on other JAR files, you’ll have to download those other JAR files and copy them to the lib directory as well.

If you have a single *managed* dependency, add a `libraryDependencies` line to your *build.sbt* file.
To add multiple managed dependencies to your project, define them as a `Seq` in your *build.sbt* file.

A *managed dependency* is a dependency that’s managed by your build tool, in this case,
SBT. In this situation, if library a.jar depends on b.jar, and that library depends on
c.jar, and those JAR files are kept in an Ivy/Maven repository along with this relationship
information, then all you have to do is add a line to your build.sbt file stating that you
want to use a.jar. The other JAR files will be downloaded and included into your project
automatically.
