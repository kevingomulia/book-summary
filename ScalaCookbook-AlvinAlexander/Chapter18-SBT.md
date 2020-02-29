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
SBT. In this situation, if library *a.jar* depends on *b.jar*, and that library depends on *c.jar*, and those JAR files are kept in an Ivy/Maven repository along with this relationship information, then all you have to do is add a line to your *build.sbt* file stating that you want to use *a.jar*. The other JAR files will be downloaded and included into your project automatically.

There are two general forms for adding a managed dependency to a *build.sbt* file.
- `libraryDependencies += groupID % artifactID % revision`
- `libraryDependencies += groupID % artifactID % revision % configuration`

You can use either % or %% after the `groupID`:
- % is used to construct an Ivy Module ID from the strings you supply
- %% automatically adds your project's Scala version (such as `_2.10`) to the end of the artifact name.

In some cases, the string `test` is added after the `revision`:
```
"org.scalatest" % "scalatest_2.10" % "1.9.1" % "test"
```
In this case, the dependency you're defining will be added to the classpath for the Test configuration, and won't be added in the Compile configuration.

SBT uses the standard Maven2 repository by default, so it can locate most libraries when you add a `libraryDependencies` line to a *build.sbt* file. However, when a library is not in a standard repository, you can tell SBT where to look for it. It will be covered in **Recipe 18.11**.

## 18.5 Controlling Which Version of a Managed Dependency Is Used
The `revision` field in the `libraryDependencies` setting isn't limited to specifying a single, fixed version. As an example, rather than specifying version 1.8 of a `foobar` module:
`libraryDependencies += "org.foobar" %% "foobar" % "1.8"`
You can request the `latest.integration` version like this:
`libraryDependencies += "org.foobar" %% "foobar" % "latest.integration"`

The module developer will often tell you what versions are available or should be used. The Ivy "dependency" documentation states that these tags can be used:
- `latest.integration`
- `latest.[any status]`, e.g. `latest.milestone`
- You can end the revision with a `+` character. This selects the latest subrevision of the dependency module. For instance, if the dependency module exists in revisions `1.0.3`, `1.0.7`, and `1.1.2`, specifying `1.0.+` as your dependency will result in `1.0.7` being selected.
- You can use "version ranges", like so:
```
[1.0,2.0] matches all versions greater or equal to 1.0 and lower or equal to 2.0
[1.0,2.0[ matches all versions greater or equal to 1.0 and lower than 2.0
]1.0,2.0] matches all versions greater than 1.0 and lower or equal to 2.0
]1.0,2.0[ matches all versions greater than 1.0 and lower than 2.0
[1.0,) matches all versions greater or equal to 1.0
]1.0,) matches all versions greater than 1.0
(,2.0] matches all versions lower or equal to 2.0
(,2.0[ matches all versions lower than 2.0
```

## 18.6 Creating a Project with Subprojects
You want to configure SBT to work with a main project that depends on other subprojects.

Firstly, create your subproject as a regular SBT project, but **wthiout a project directory**. In your main project, define a *project/Build.scala* file that defines the dependencies the between the main project and subprojects.

Here's an example:
```
import sbt._
import Keys._

/**
* based on http://www.scala-sbt.org/release/docs/Getting-Started/Multi-Project
*/

object HelloBuild extends Build {
  // aggregate: running a task on the aggregate project will also run it
  // on the aggregated projects.
  // dependsOn: a project depends on code in another project.
  // without dependsOn, you'll get a compiler error: "object bar is not a
  // member of package com.alvinalexander".
  lazy val root = Project(id = "hello",
                          base = file(".")) aggregate(foo, bar) dependsOn(foo, bar)

  // sub-project in the Foo subdirectory
  lazy val foo = Project(id = "hello-foo",
                          base = file("Foo"))

  // sub-project in the Bar subdirectory
  lazy val bar = Project(id = "hello-bar",
                          base = file("Bar"))
}
```
You can also see the instructions in the SBT Multiproject documentation.

Creating a main project with subprojects is well documented on the SBT website, and the primary glue that defines the relationships between projects is the **project/Build.scala** file you create in the main project.

## 18.7 Using SBT with Eclipse

## 18.8 Generating Project API documentation
You have marked up your source code with Scaladoc comments, and want to generate the API documentation for your project.

You can use any of the commands here, depending on your needs:
- doc - Creates Scaladoc API documentation from the Scala source code files located in */src/main/scala*
- test:doc - Creates Scaladoc API documentation from the Scala source code files located in *src/test/scala*
- package-doc - Creates a JAR file containing the API documentation created from the Scala source code in *src/main/scala*
- test:package-doc - Same as above, but files located in *src/test/scala*
- publish - Publishes artifacts to the repository defined by the `publish-to` setting. See **Recipe 18.15**
- publish-local - Publishes artifacts to the local Ivy repository as described

## 18.9 Specifying a Main Class to Run
What if you have multiple `main` methods in objects in your project, and you want to specify which `main` method should be run when you type `sbt run`, or when your project is packaged as a JAR file.

You can add a line like this to your *build.sbt* file:
```
mainClass in (Compile, run) := Some("com.alvinalexander.Foo")
```
This lass can either contain a `main` method, or extend the `App` trait.

To specify the class that will be added to the manifest when your application is packaged as a JAR file, add this to the *build.sbt* file:
```
mainClass in (Compile, packageBin) := Some("com.alvinalexander.Foo")
```

### Using run-main
When running your application with SBT, you can also use SBT's `run-main` command to specify the class to run. Invoke it like so in the command line:
```
$ sbt "run-main com.alvinalexander.Foo"
```

## 18.10 Using Github Projects as Project Dependencies
You can reference the GitHub project you want to include in your *project/Build.scala* file as a `RootProject`.

For example:
```
import sbt._

object MyBuild extends Build {
  lazy val root = Project("root", file(".") dependsOn(soundPlayerProject))
  lazy val soundPlayerProject =
        RootProject(uri("git://github.com/alvinj/SoundFilePlayer.git"))
}
```
Although this works for compiling and running your project, you can't package all of this code into a JAR file by just using the `sbt package` command. Unfortunately, SBT doesn't include the code from the GitHub project.

You can use a plug-in named **sbt-assembly** that lets you package all of this code together as a single fat JAR.

While the *build.sbt* file is used to define simple settings for your SBT project, the *project/Build.scala* file is used for "everything else". In this file, you write Scala code using the SBT API to accomplish any other task you want to achieve.

To use multiple GitHub projects as dependencies, add additional `RootProject` instances to your `project/Build.scala` file:
```
import sbt_.

object MyBuild extends Build {
  lazy val root = Project("root", file("."))
                    .dependsOn(soundPlayerProject)
                    .dependsOn(appleScriptUtils)

  lazy val soundPlayerProject =
        RootProject(uri("git://github.com/alvinj/SoundFilePlayer.git"))

  lazy val appleScriptUtils =
        RootProject(uri("git://github.com/alvinj/AppleScriptUtils.git"))

}
```

## 18.11 Telling SBT How to Find a Repository (Working with Resolvers)
Use the `resolvers` key in the *build.sbt* file to add any unknown Ivy repositories. Use this syntax to add one resolver:
```
resolvers += "Java.net Maven2 Repository" at "http://download.java.net/maven/2"
// multiple Resolvers

// resolvers ++= Seq(
  "Typesafe" at "http://repo.typesafe.com/typesafe/releases/",
  "Java.net Maven2 Repository" at "http://download.java.net/maven/2/"
)
```
This is the general format:
```
resolvers += "repository name" at "location"
```

## 18.12 Resolving Problems by Getting an SBT Stack Trace
When an SBT command silently fails (typically with a nonzero exit code message), but you can't tell why, run your command from within the SBT shell, then use the `last run` command after the command that failed.

This will show you the stack trace so that you can identify your problem easily. Another approach is to set the SBT logging level. More details in **Recipe 18.13**.

## 18.13 Setting the SBT Log Level
You can set the SBT logging level in your *build.sbt* file with this:
```
logLevel := Level.Debug
```
or from the SBT CLI:
```
> set logLevel := Level.Debug
> set logLevel := Level.Info
> set logLevel := Level.Warning
> set logLevel := Level.Error
```

## 18.14 Deploying a Single, Executable JAR File
The `sbt package` command creates a JAR file that includes the class files it compiles from your source code, along with the resources in your project (from *src/main/resources*), but there are two things it doesn't include:
1. Your project dependencies (JAR files in your project's *lib* folder, or managed dependencies declared in *build.sbt*)
2. Libraries from the Scala distribution that are needed to execute the JAR file with the `java` command.

There are 3 things you can do solve this problem:
1. Distribute all the JAR files necessary with a script that builds the classpath and executes the JAR file with the `scala` command. This requires that Scala be installed on client systems.
2. Distribute all the JAR files necessary (including Scala libraries) with a script that builds the classpath and executes the JAR file with the `java` command. This requires that Java is installed on client systems.
3. Use an SBT plug-in such as **sbt-assembly** to build a single, far JAR file that can be executed with a simple `java` command. Again, Java has to be installed on client systems.

### Using sbt-assembly
First, add these lines to a *plugins.sbt* file (create it if it doesn't exist) in the *project* directory of your SBT project:
```
resolvers += Resolver.url("artifactory", url("http://scalasbt.artifactoryonline.com/scalasbt/sbt-plugin-releases")) (Resolver.ivyStylePatterns)

addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.8.4")
```
Then add these to the top of your *build.sbt* file:
```
import AssemblyKeys._

// sbt-assembly
assemblySettings
```
Now you can run `sbt assembly` to create your single fat JAR.

sbt-assembly works by copying the class files from your source code, the class files from your dependencies, and the class files from the Scala library into one single JAR.

### SBT, Scala interpreter, and Java interpreter
A JAR file created by SBT can be run by the Scala interpreter, but not the Java interpreter. This is because class files in the JAR file created by `sbt package` have dependencies on Scala class files, which aren't included in the JAR file SBT generates.

For the Java interpreter to run your JAR file, it needs the *scala-library.jar* file from your Scala installation to be on its classpath. This is the part of the work that sbt-assembly performs for you.

## 18.15 Publishing Your Library
You can either use `sbt publish-local` to publish to your local Ivy repository or define your repository information, and then publish your project with `sbt publish`.

You can add your repository information in the *build.sbt* file like:
```
publishTo :=Some(Resolver.file("file", new File("/Users/al/tmp")))
```

## 18.16 Using Build.scala instead of build.sbt
The recommended approach when using SBT is to define all your simple settings (key/value pairs) in the *build.sbt* file, and handle all the other work, such as build logic in the *project/Build.scala* file.

Let's learn more about how the *Build.scala* file works.

First, don't create a *build.sbt* file in your project, but create a *Build.scala* file in the project subdirectory by extending the SBT `Build` object:

```
import sbt._
import Keys._

object ExampleBuild extends Build {
  val dependecies = Seq(
    "org.scalatest" %% "scalatest" % "1.9.1" % "test"
  )

  lazy val exampleProject = Project("SbtExample", file(".")) settings(
    version       := "0.2",
    scalaVersion  := "2.10.0",
    scalacOptions := Seq("-deprecation"),
    libraryDependencies ++= dependencies
  )
}
```

With just this *Build.scala* file, you can now run all the usual SBT commands in your project, including `compile`, `run`, `package`, etc.

That file is equivalent to the following *build.sbt* file:
```
name := "SbtExample"
version := "0.2"
scalaVersion := "2.10.0"
scalacOptions += "-deprecation"
libraryDependencies += "org.scalatest" %% "scalatest" % "1.9.1" % "test"
```
You can give your build file any legal Scala filename, as long as you place the file in the project directory with a *.scala* suffix.

## 18.17 Using a Maven Repository Library with SBT
You want to use a Java library that's in a Maven repository, but the library doesn't include information about how to use it with Scala and SBT.

Firstly, translate the Maven `groupId`, `artifactId` and `version` fields into an SBT `libraryDependencies` setting. Then, add this line to the *build.sbt* file, then run `sbt compile`.

As mentioned in other recipes, because SBT and Maven both use Apache Ivy under the covers, and SBT also uses the standard Maven2 repository as a default resolver, SBT users can easily use Java libraries that are packaged for Maven

## 18.18 Building a Scala Project with Ant
Assuming you have a Maven- and SBT-like project directory structure, create the following Ant *build.xml* file in the root directory of your project:

```
get from http://www.scala-lang.org/node/98
```
