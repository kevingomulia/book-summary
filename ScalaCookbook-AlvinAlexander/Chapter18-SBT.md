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
