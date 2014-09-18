

——

##[fit] Hello, I'm Lisa.
###[fit] I work on the Android team at Etsy

^ In addition to working on our app, I write and maintain our project Gradle scripts, 
and keep our CI servers up to date.  

---

![](android_team.jpg)

^  Here's the part of the team that went to Google IO this year, plus our Designer Paul and Doug, a PM at Etsy.
We're all excited about the #cardboard freebies.

---

## Why Gradle?

^ But enough about me. Let’s talk about Gradle.

^ But first off, why are we talking about Gradle at an Android conference? Well, mostly because Google (aka Xavier and the rest of the tools team at Google), said that it was a good idea.
Which is true, Gradle is powerful and super handy.  A lot handier than Ant, the build system that Android use previously.

---

## Why Gradle?
![](android_studio.png)

^ But yeah, basically the reason that we all use Gradle now is because it is THE build system for Android Studio. If you want to develop Android using Android Studio, 99% of you will have Gradle running your builds.


---

# Gradlin'

## __Nuts and bolts__

#### Lisa Neigut __DroidCon NYC__ Sept 21


^ Before we get started, please indulge my curiosity. By a show of hands, how many of you 
have worked on or written any Gradle scripts?

^[[ PAUSE TO LET PEOPLE RAISE THEIR HANDS ]]. Ok, thank you.  [[ MAKE COMMENT ABOUT % ]]

^ So drink up that coffee. Let's learn some Gradle.

---

# Nuts and Bolts
- Groovy
- Gradle
	- Tasks
	- Build Phases
	- Projects

^ Here’s a quick list of what I’m hoping to cover in the next … 35 minutes

^ With any talk on the nuts and bolts of Gradle, especially in a conference for Android devs who, I'm assuming,
mostly write Java, it seems wise to begin with a short primer on Groovy, the language that Gradle is built on top of.

^ If you already know Groovy backwards and forwards, please excuse my, hopefully short, digression.  I promise this will all 
be useful for understanding (and possibly writing) Gradle build scripts.

^ After Groovy, we'll get into the basics of gradle, specifically tasks, build phases and projects

^ So what is Groovy?  I wasn’t really sure, so I looked it up on Wikipedia.  This is the definition that I more or less pieced together from that:

---

## Groovy:
A dynamic, static, strongly, duck typed language that is imperative, object-oriented, functional, and scripting.
It runs on the Java Virtual Machine.

^ [[READ WHATS ON THE SCREEN]] Ok, let's break that down a bit.  It's... a turducken?

--- 

![](turducken.jpg)

---

## Groovy:
A dynamic, static, strongly, duck typed language that is imperative, object-oriented, functional, and scripting.
It runs on the Java Virtual Machine.

^ Ok, so what does this beast actually look like, on paper?

---

Java:

````java
for (String it : new String[] { "Erin", "Jan", "Taylor" }) {
	if (it.length() >= 4) {
		System.out.println(it);
	}
}
````

^ Let's start with a canonical example of printing an array Strings in Java
	
---

Groovy:

````java
["Erin", "Jan", "Taylor"].findAll{ it.size() >= 4 }.each{ println it }
````

^ That's a lot to parse all at once. Let's break it down, piece by piece.

---

Groovy:

```` java
	["Erin", "Jan", "Taylor"]
````

^ All the things I want to print.  
Note that this may look like an array, but the underlying type is a list.


---

Groovy: 

```` java
		.findAll{ it.size() >= 4 }
````

^ Acts like a 'filter'. For each element in the list, include the item in a new filtered list, if passes this test.

---

Groovy: 

````java
	.each{ println it }
````

^ For each item, that passes the previous filter (.findAll), print `it` out.

---

Groovy: 

````java
["Erin", "Jan", "Taylor"].findAll{ it.size() >= 4 }.each{ println it }
````

^ Here it is again all in one line.  [[ Pop this in your emulator, and run it!]]

---

Java:

````java
for (String it : new String[] { "Erin", "Jan", "Taylor" }) {
	if (it.length() >= 4) {
		System.out.println(it);
	}
}
````

Groovy:

````java
["Erin", "Jan", "Taylor"].findAll{ it.size() >= 4 }.each{ println it }
````

^ So here they are side by side.  

^ A quick side note. In the Groovy, it is an abbreviation. We’ll get to this later.

---

# Groovy

---
 
## Fancy Groovy Features

- Syntax
- Null Checks
- Dynamic Typing
- GStrings
- Arrays
- Closures
- Objects


^ Note that all examples were, more or less, taken off of the Groovy Wikipedia page.  Thanks internet commons!


---

## Let's start with the good parts!

---

###[fit] ;

^ completely optional!!

---

# Syntax

---

Groovy

````
	take(coffee).with(sugar, milk).and(liquor)
````

^ you can also take a lot more out.

---

Groovy

````
	take(coffee).with(sugar, milk).and(liquor)

	take coffee with sugar, milk and liquor
````

^ Becomes this.  I like being explicit, but this is a way that you can do this.

^ Cool, so Groovy requires a smidge less typing that Java.

---

# Null Checks

---

Java

````java
	if (activity != null) { 
		activity.finish();
	}
````

^ chances are you’ve written some code that looks like this at some point

^ I’m happy to tell you that Groovy makes it a bit easier. instead of writing this out, youc an just do this.

---

Java

````java
	if (activity != null) { 
		activity.finish();
	}
````


Groovy

````java
	activity?.finish()
````

^ Put a Question Mark before the function call. The technical term for this is the safe navigation operator.  

^ I call it the Destroyer of NPEs. 

^ [[ QUICK DEMO ]]

---

# Dynamic Typing

---

## Use the `def` keyword!

Groovy:

`def variable`

^ I’m happy to say that you no longer need to know the type of your variables in Groovy.  You can just declare them as `def`.

---

## Use the `def` keyword!

Groovy:

`def variable`
`String variable`

^ If you want your IDE to help you out, it’s still ok to declare the type.  This is where the duck and static typing part of the turducken comes in.

---

# GStrings

^ Groovy Strings.  Sort of like templating languages (like Smarties if you do PHP)

---

Java:

````java
	int amount = 4563;
	String yourBill = "You owe me " + amount;
````

---

Java:

````java
	int amount = 4563;
	String yourBill = "You owe me " + amount;
	
	String.format("You owe me %d", amount);
````

^ or if you want to get fancy, you can do a format string.
This is totally reasonable for most things, but can get cumbersome quickly.

---

Java:

````java
	String.format("On %s the %s of %s %s paid $%d to %s", 
		dayOfWeek, day, month, payee, amountPaid, richMan);
````

^ this string was no fun to write

---

Groovy:

````java
    def amount = 4563
	def yourBill = "You owe me $amount"
````

^ Could also be written in Groovy, in line.
If you want to get super fancy you can also write it like this:

---

Groovy:

````java
    def amount = 4563
	def yourBill = "You owe me $amount"
	
	def fancyAmount = [4,5,6,3]
	def yourBill = "You owe me ${ fancyAmount.join() }"
````

^ super fancy, by joining a list of numbers

---

Groovy:

````java
    def amount = 4563
	def yourBill = "You owe me $amount"
	
	def fancyAmount = [4,5,6,3]
	def yourBill = "You owe me ${ fancyAmount.join() }"

	def yourBill = "You owe me \$${ fancyAmount.join() }"
````

^ if you want to get real fancy.

---

Groovy:

````java
String string = 
	“On $dayOfWeek the $day of $month” +
	“ $payee paid \$$amountPaid to $richMan”
````

^ And here’s that no fun format string from Java, in Groovy

—

# Arrays

---

^ Lists are written like arrays.

^ run some fun things to prove this in your emulator. Samples include
["Erin", "Jan", "Taylor" ]
["Erin", "Jan", "Taylor" ].size()
["Erin", "Jan", "Taylor"].length()
["Erin", "Jan", "Taylor" ].get(1)
["Erin", "Jan", "Taylor" ][1]
for (String string : [ "Erin", "Jan", "Taylor" ]) { println string.length() }

---

# Closures

^ Let's go over closures really fast (I promise that this will come back up later. Maybe to haunt you forever.)
Especially since as Java programmers, most of us don't ever have to deal with closures.
Unless you secretly do iOS programming on the side. In which case you're probably familiar with 
block syntax, which is essentially the same thing, just with a different name.

---

Java:

````java
new MyFancyNewObjectThatsReallyJustAMethodEnclosure().doSomething();
````

^ So in regular, plain old Java, if there's a method that you want to be called, you have to have an object to call 
that method on

^In Groovy, You can pass code around in a 'closure'.  The code inside the closure
won't be executed until it's called, but it's no longer encapsulated inside of a function call.  

^Here's an example:

---

Groovy:

```` java
list = [1, 2, 3, 4, 5, 6, 7, 8, 9]

def odds = list.findAll { it % 2 }

assert odds == [1, 3, 5, 7, 9]

````

^ The closure here is { it % 2 }.  That's an in line declared method.

^Closures aren’t executed until they’re called, so they can be lazily evaluated. Let’s take a look at some lazily evaluated code.

^[[ Run below in emulator ]]

^Integer amount = 10
BigDecimal tax = 0.15
text = "Your account balance is ${account - account * tax}"
deferredText = "Your account balance is ${-> account - account * tax}"

^println text
println deferredText

^tax = 0.10

^println text
println deferredText

---

Groovy:

````java
\\ Abbreviated with it
["Erin", "Jan", "Taylor"]
    .findAll{ it.size() >= 4 }
    .each{ println it }
````

```java
\\ Long form
["Erin", "Jan", "Taylor"]
	.findAll{ name -> name.size() >= 4 }
	.each{ filteredName -> println filteredName }
````

^ It is a shortcut way of saying "the current item in the list that you're iterating over!"

---

# Groovy Objects

---

Groovy:

````java
class AGroovyObject {
	String color
}

def myGroovyObject = new AGroovyObject()

myGroovyObject.setColor('blue')
assert myGroovyObject.getColor() == 'blue'

myGroovyObject.color = 'fuschia'
assert myGroovyObject.color == 'fuschia'

````

^ You can declare classes in Groovy.  Properties on those classes become insta-setters/getters.

^Be careful that you're not clobbering some property declared on a class.
Ok, that's all well and good. What if I wanted to add some dynamic properties to that class?


---

Groovy: 

````java
class ExpandoClass {

    def propertyMissing(String name) {
        println "Missing property $name"
    }

    def backingMap = [:]

    Object getProperty( String property ) {
      if( backingMap[ property ] == null ) {
        propertyMissing( property )
      }
      else {
        backingMap[ property ]
      }
    }

    void setProperty( String property, Object value ) {
      backingMap[ property ] = value
    }
}
````

^ [[ POP THIS INTO YOUR TERMINAL ]]. Thank you StackOverflow.

^Cool, now we've got an automagically expanding class.  And that about wraps up our basics 
discussion.

--- 
 
![right|fit](groovy_logo.png)

## And that's Groovy

- Syntax
- Null Checks
- Dynamic Typing
- GStrings
- Arrays
- Closures
- Objects

^ Just to quickly re-iterate. 
So now that we've got a handle on some Groovy Basics, let's take a look at Gradle

---

#Gradle:

Gradle is a project automation tool that builds upon the concepts of Apache Ant and Apache Maven and introduces a Groovy-based domain-specific language (DSL) instead of the more traditional XML form of declaring the project configuration.

^ What is it?

^ [[ READ DESC ]]

^ I went ahead and made up my own definition for this

---

#Gradle:
Java builds without XML.

^ Hooray!

—

## Structure of a Gradle Project

---


^ There are 3 files that compose a gradle build.

---

- build.gradle

^ The build file where your build configuration and logic go.  In Android apps, there are 2 of these, and we'll go over them.

---

- build.gradle
- settings.gradle

^ The 'settings' file that holds your project structure. This is where you declare multiple Gradle projects under a single build


---

- build.gradle
- settings.gradle
- gradle.properties

^ Where you put different build properties.  

^ Gradle has some configuration options, you can also put build specific flags into here
As long as you’re not checking this into your source control, it’s a great place for hiding project secrets

---

^ [[ GO through these files quickly in the IDE ]]

^ there are two build.gradle files : one in the project root. you can access every project from the root.
^ any project specific build files.  (the :app one)

^ show off adding more projects to settings.gradle
^ include ':app', ':app1'
^ include ':app', ':nested:app1'
^ run ./gradlew projects to show off how you set up different structures for these

---

gradle.properties

````
org.gradle.daemon=true
org.gradle.java.home
# org.gradle.jvmargs
org.gradle.parallel
org.gradle.configureondemand
````

^ Just as a quick aside, here's a few gradle configs that can speed up your builds.

^ these can also be passed into your build via the command line.  I find it easier to keep them here.

^ the daemon is a long running process. This should be turned off on CI builds, but enabled locally

^ your java home directory.  the documentation claims that a 'reasonable default' is used if this isn't provided. we don't provide it.

^ the jvmarg settings. these are included by default in any new Android Studio project, you can uncomment them if you need more memory

^ Builds different projects in parallel.  Don't use this if you're using projects as dependencies in other projects.

^ Configure on Demand.  This speeds up multiproject builds. (It'll have no effect on single projects, like my demo project.)  More details on this when we talk about configuration phase in a bit.

^ finally, you can add your own properties to these, and they'll available as a property on the Root Project in your build files.

^ Great. So now that we've got the basic project set up under our belt, let's take a look at some of the core
pieces of a Gradle build

---

## Gradle Core Components

- Tasks
- Build Phases
- Projects

^ Just as in Groovy, there are a few basic building blocks that Gradle is built off of.
Let's go through them.  We'll start with tasks.

---

# Tasks

---

The task is the core building block of a build action.  

^ All build logic is built as a task, or a series of tasks, that get run.

---

The task is the core building block of a build action. 

Build an Android Project

```` bash
	$ ./gradlew  assemble
````

^ Tasks are invoked from the command line.  here's the task that gets run when you build an Android project

---

The task is the core building block of a build action.

Build an Android Project

```` bash
	$ ./gradlew  assemble
````

Show me all the Tasks in a project

````bash
	$ ./gradlew tasks
````

^ Tasks are invoked from the command line.  here's the task that gets run when you build an Android project

---

````
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Android tasks
-------------
androidDependencies - Displays the Android dependencies of the project
signingReport - Displays the signing info for each variant

Build tasks
-----------
assemble - Assembles all variants of all applications and secondary packages.
assembleDebug - Assembles all Debug builds
assembleDebugTest - Assembles the Test build for the Debug build
assembleRelease - Assembles all Release builds
build - Assembles and tests this project.
...
````

^ [[ Switch to terminal to go through this ]]

^Android tasks
Android project introspection tasks, added by Android plugin

^Build tasks
Tasks to build Android projects, added by the Android plugin

^Build setup tasks
Gradle wrapper tasks. These are for setting up a new gradle wrapper

^Help tasks
Gradle tasks for inspecting build system properties

^Install tasks
ADB actions for installing / uninstalling APKs; added by Android plugin

^Verification tasks 
Running instrumentation tests and Android lint; added by Android plugin

^ tasks are split up into different sections.  In vanilla android installation, we've got 
^ if we run it again with the --all flag, we see all of the dependent tasks that are built as a result of
calling the header task.

---

^ ok, so let's write our own task

---

Groovy / Gradle:

````java
	task hello {
    	doLast {
        	println "Hello DroidConNYC!"
	    }
	}
````

^ Let's write a task to commemorate DroidConNYC

---

Groovy / Gradle:

````java
	task hello {
    	doLast {
        	println "Hello DroidConNYC!"
	    }
	}
````

Run:

````bash
	$ ./gradlew hello
````

---

And we get...

````
♤ ./gradlew hello
:app:hello
Hello DroidConNYC!

BUILD SUCCESSFUL

Total time: 11.075 secs
````

^ cool. let's try some other things. 

---

Gradle:

````java
task hello {
    doLast {
        println "Hello DroidConNYC!"
    }
}

task newHello << {
    println "Hello Again!"
}

task brokenHello {
    println "Hello -- Too Soon!!"
}
````

---

Gradle:

````java
♤ ./gradlew newHello
:app:newHello
Hello Again!

BUILD SUCCESSFUL

Total time: 11.1 secs

````

---

Gradle:

````java
♤ ./gradlew newHello
:app:newHello
Hello Again!

BUILD SUCCESSFUL

Total time: 11.1 secs

♤ ./gradlew brokenHello
Hello -- Too Soon!!
:app:brokenHello UP-TO-DATE

BUILD SUCCESSFUL

Total time: 10.184 secs

````

^ what's going on?
Gradle uses closures to build task logic that will be executed at runtime.

^ any logic that isn't wrapped in the doLast{} block gets run during the 'configuration' stage.

^ ok, so with this in mind, let's talk about the phases of a gradle build

---

# Build Phases

---

^ There are three phases to any Gradle build

---

- Initialization

^ The first phase is initialization. The build script starts up, and determines which projects are going to take part in the build, creates a Project object for every project.

---

- Initialization
- Configuration

^ Next is configuration, when all the project objects are 'hydrated'. When you run a task from the command line, you'll see the
"configuring" label pop up.  Gradle will go through all of the projects that you have listed in settings.gradle and 
actually figure out the dependency and task graphs for all of them.

---

- Initialization
- Configuration*

gradle.properties

````
org.gradle.configureondemand=true
````

^ if you're only interested in a subset of projects though, it's probably good to turn on the configure on demand 
flag.  This will help speed up multi-project builds, as configuration happens EVERY time that you run a gradle command.

---

- Initialization
- Configuration
	- Resolves Dependency Graph

	
^ So what happens during a configuraion phase?  The project objects are built.  including
Resolving the dependency graph (making sure that all the libraries you've said are required aren't circular, and that they're available.)


---

- Initialization
- Configuration
	- Resolves Dependency Graph
	- Task Execution Plan
	
^ this also includes comes up with the task execution plan - what tasks must be executed before the task you requested
can be run.


---

- Initialization
- Configuration
	- Resolves Dependency Graph
	- Task Execution Plan	
- Execution
	
^ Next, Gradle goes ahead and executes the task plan.  At this point, all your task code runs.

^ So let's watch this run, see if you can spot where it's switching between the different phases.
	
---

````bash
♤ ./gradlew hello
> Configuring > 2/2 projects

:app:newHello
Hello Again!

BUILD SUCCESSFUL
````

---

Groovy:

````java
task newHello << {
    println "Hello Again!"
}

task brokenHello {
    println "Hello -- Too Soon!!"
}
````

^ Now let's go back to that broken task.  What's happening here?

---

#[fit] Closure magic!

![150%|right](closure_magic.gif)

^ Closure magic!!

^ Gradle takes advantage of Groovy closures to delay execution of task codes until 
the task invoked by either the task execution plan, or from the command line.  Any code 
in a build script that is not wrapped inside of a task's closure block (the magical doLast{ })
will be executed in the configuration phase, not at execution.

---

Debugging Tasks

````bash
	./gradlew <command> --stacktrace --debug --info
````

^ Quick aside, to help you out when writing tasks, there are a few flags you can throw on the command line
that will make debugging a lot easier.  Stacktrace and info, I find, are the most useful.
[[ RUN THIS ]]

---

# The Project

^ also known as the build.gradle file

---

Project Components

- Tasks
- Dependencies
- Configurations
- Properties
- Plugins

^ Here are the components of a project.  

---

Project Components

- _Tasks_
- Dependencies
- Configurations
- Properties
- Plugins*

^ We've already covered Tasks, and plugins isn't going to make it into this talk today.
So let's move along to dependencies and configurations

---

^ [[ GO TO ANDROID STUDIO TO DO THIS ]]

^ [[ SHOW OFF THE AUTO ADDED DEPENDENCY ]]
Dependencies are simply other projects or libraries that are needed to compile a project.
Here's an example of a dependency that is the libs directory, only including .jar files. By default, Android Studio adds 
this line to a new Gradle project build file for you.

^ [[show off all the dependencies and configuations for the :app project]]
^ ./gradlew dependencies (comes back null)
^ ./gradlew :app:dependencies  (shows all dependencies for :app)
Dependencies are specified for a specific configuration.

^ [[ Add a second project, and then add it as a dependency in the first ]]
^ So let's add another project, and set it as a dependency for my Android project.

^ [[ Forget to add it to settings.gradle, run ./gradlew tasks ]]. should fail.
^ it failed because it's not listed as a project in settings.gradle, so let's add it there. 

^[[ add second project to settings.gradle.]]
[[ Re run gradle task. Should succeed. ]]
Cool, so now it's a dependency.

^ let's add a configuration to specify dependencies for.
^ ./gradlew :app2:dependencies  (shows all dependencies for :app2)
^ app2 has no configurations. Let's add one.
^ [[ Add libraries configuration for app2, show that it now picks up in the list ]]

^ ok, that's cool. but what is it useful for?
Imagine that you've got libraries that you want to be included both in your app build, and in your test compilations.
Let's create a single configuration to specify dependencies for.
^ configurations {
    libraries {}
    compile {
        extendsFrom libraries
    }
    androidTestCompile {
        extendsFrom libraries
    }
}

^ and let's add a dependency to those configurations
 libraries 'com.etsy.android.grid:library:1.0.5'

---

````java
dependencies {
  //for dependencies found in artifact repositories you can use
  //the group:name:version notation
  compile 'commons-lang:commons-lang:2.6'
  testCompile 'org.mockito:mockito:1.9.0-rc1'

  //map-style notation:
  compile group: 'com.google.code.guice', name: 'guice', version: '1.0'

  //declaring arbitrary files as dependencies
  compile files('hibernate.jar', 'libs/spring.jar')

  //putting all jars from 'libs' onto compile classpath
  compile fileTree('libs')
  
  compile project(:app2)
}

//taken from http://www.gradle.org/docs/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html
````

^ Here's some examples of different ways to add dependencies.  Note that the simple string declarations 
are from artifact repositories.

---

## Properties

^ Cool. the last thing to go over is Gradle properties.  Remember the Expando Groovy example, of a class
where you could add properties on the fly?

---

Groovy: 

````java
class ExpandoClass {

    def propertyMissing(String name) {
        println "Missing property $name"
    }

    def backingMap = [:]

    Object getProperty( String property ) {
      if( backingMap[ property ] == null ) {
        propertyMissing( property )
      }
      else {
        backingMap[ property ]
      }
    }

    void setProperty( String property, Object value ) {
      backingMap[ property ] = value
    }
}
````

^ This is what it looked like, in case you've forgotten. Gradle Projects work very similarly, except you access
the Expando class through the extension ext.

---

Properties

````java
project.ext.prop1 = "foo"
task doStuff {
    ext.prop2 = "bar"
}

````

^ Here's some examples of how to add properties to a project.

---

Properties

````java
project.ext.prop1 = "foo"
task doStuff {
    ext.prop2 = "bar"
}

project.hasProperty['prop1']

````

^ You can also check for a property, before calling it.

---

^ let's explore.
[[ ADD ON app2 ]]
project.ext.prop1 = "Property Number One"
./gradlew :app2:properties

^ you can also add properties from the command line, or in gradle.properties
task props << {
    if (project.hasProperty("ZBestPropertyToHave")) {
        println "You've got lots of $project.ZBestPropertyToHave"
    }
    else {
        println "Hurry up and find yourself some properties!"
    }
}
./gradlew props
./gradlew props -PZBestPropertyToHave=time


^ And that just about wraps up our 30,000 foot dive into Gradle.

---

# Things not covered in this talk:
- Task dependencies
- Manipulating Files
- Build Graph Hooks
- The Android Plugin
- Multiproject Build Logic
- Plugins

^ There were a lot of things that we didn't have time to jump into today.
Here's a short list of those things.  As an aside, I'm presenting on Gradle Plugins
at DroidCon London at the end of October.  It'll be a bit more in depth into the Gradle
API than today.

---

## Further Reading:
- Gradle Beyond the Basics
	http://chimera.labs.oreilly.com/books/1234000001741/ch01.html
- Extensive Gradle API documentation at gradle.org
- Gradle Plugin User Guide - Android Tools Project Site 
	http://tools.android.com/tech-docs/new-build-system/user-guide

^ There is a *great* book on getting farther into Gradle tasks and files from O'Reilly media.  If you enjoyed this talk and want to dig further, I highly suggest you start here.
^ Gradle.org is a good source of information on their API
^ Finally, if you want to dig into the Android Plugin and configuration, the Android Tools Project Site is the place to go.
It's not as well written as the frist two, but it's the best place to learn about the Android Plugin

---

# Gradlin'

## __Nuts and bolts__

#### Lisa Neigut __DroidCon NYC__ Sept 21

^ Thank you.

---

---

# Hooks

---

## Task Hooks

^ Task hooks are very cool, and are very worth talking about.

---

When the Task Graph has been configured

````java
\\ Called after configuration, but before execution
gradle.taskGraph.whenReady { taskGraph ->
	if (taskGraph.hasTask(assembleRelease)) {
		version = '1.0'
	}
	else {
		version = '1.0.0.Beta'
	}
}
````

^ Lets you do some inspection on the tasks that have been configured for a project build.  This will include 
all of the tasks that have been configured.

---

Start and End of Task Execution.

````java
gradle.taskGraph.beforeTask { Task task, TaskState state ->

}

gradle.taskGraph.afterTask { Task task, TaskState state ->
	
}
````

^ These methods are called for each task that is executed, as they're executed.

---

When Tasks are Added to a Project

````java
tasks.whenTaskAdded { task ->
	task.ext.srcDir = "src/main/java"
}

task a

println "source dir is $a.srcDir"
````

^ Here's an example that adds a property to the task called scrDir.

---

## Project Hooks

---

Project Hooks

````java
beforeEvaluate { project ->
	
}

afterEvaluate { project -> 
	if (project.hasTests) {
		println "Adding tests to $project"
		project.task('test') << {
			println "Running tests for $project"
		}
	}
}

````

^ These listeners are set on the project.

---

Hooks for All Projects

````java
gradle.afterProject {project, projectState ->
    if (projectState.failure) {
        println "Evaluation of $project FAILED"
    } else {
        println "Evaluation of $project succeeded"
    }
}
````

^ Prints out the state of the evalution


---

# Multi Project Builds

---

Multi-project builds depend on an extra file called `settings.gradle`.

The location of the file in the build tree is where the build file for that project goes.

````
:app
:secondApp
````

Once the projects are created, each of them will have a set of tasks created for them in the
gradle task console.  To reference tasks on a specific project, you can prepend the project name, like so

`:app:assemble`

---

Setting up and configuring multi-project builds

The projects(:projectName) block
the allProjects() block

---

- androidTestCompile includes compile
- include an example of an @aar
- what have i used gradle for?
- Changes the version number (use real life examples)
- more examples from real life
- more syntactic examples in the
- play with the display set up