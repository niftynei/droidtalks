
# Taking Gradle to Task

## Intro to the Gradle Task

#### Lisa Neigut | New Circle Training
#### 16 April 2015

——

##[fit] Hello, *I’m Lisa*
###[fit] I work on Android at Electric Objects
![](android_team.jpg)

^ Prior to working at Electric Objects, I worked at Etsy for two and a half years. While at Etsy I worked on the apps, as well as I writing and maintaining Gradle plugins, and the Android testing infrastructure.

^ This is the Etsy Android team at Google IO last year.  I'm the one with the zebra shirt.

^ We're all excited about getting cardboard freebies.

---

[[ TODO: Pictures of me hiking & of Janet. ]]

^ Other than writing software, I'm an avid hiker and barefoot runner, a proud Vespa owner, and very active Twitterer.

^ I live and work in NYC.

---

## Gradle Tasks

- Recognize the 3 phases of a Gradle build
- Understand how a Task interacts with build phases
- Be able to start writing your own Gradle tasks to customize your release process

^ Today I want to take you through the process of writing a Gradle Task.

^ By the end of today's training, my goal is that you'll... ( read from slide )

---

## Test Yourself

- What language is Gradle written in?
- What are the 3 phases of a Gradle build?
- What does the `<<` notation stand for in a Gradle build script?
- `doLast` is a property on a Task. What is its type?
- How do you run a Gradle task from the command line?
- Name 2 types of tasks included in the Android Gradle plugin.

^ By the end of this short presentation, you should be able to answer these questions.

---

## Why Gradle?

^ But first off, why are we talking about Gradle? 

^ We're Java developers, what do we care about a Groovy language build system?

---

## Why Gradle?
![](android_studio.png)

^ The that we all use Gradle now is because it is THE build system for Android Studio. 

^ If you want to develop Android using Android Studio, 99% of you will have Gradle running your builds.

---

## Gradle is Groovy

- Gradle is a Groovy DSL for managing build dependencies, configurations, and build related tasks
- Replaces and / or supplements Maven, the Java package manager
- Completely replaces Ant (XML)
- Script-like language for automating build tasks
- Builds a dependency graph for all build inputs and build steps

^ Gradle is Groovy. Here's why.

---

## Tasks

A Gradle build is composed of a set of Tasks.

^ A gradle build is a task, or a series of tasks, that get executed.

^ If you've ever written a task for Ant, this may look familiar.

---

- Build an Android Project

```` bash
	$ ./gradlew  assemble
````

^ For example, a task is what you call in order to build an Android project.
^ Tasks can be invoked from the command line.  
^ Here's an example of calling the task to build an Android project

---

- Show me all the Tasks in a project

````bash
	$ ./gradlew tasks
````

^ Gradle even includes a task that will show you all the tasks that are declared for a project.

^ talk about meta. :D

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

^ Here's what this looks like.  Now, when you run this on an Android Studio project, you'll see a lot of tasks that are included by the android plugin.

---


`apply plugin: 'com.android.application'`

^ If you've ever looked through the gradle scripts that are included in your Android project, that's this line.
^ Understanding plugins is beyond the scope of what we're talking about today.

---

## Android Plugin Tasks

- Android tasks: Android project introspection
- Build tasks: Build Android projects
- Install tasks: Calls ADB actions for installing / uninstalling APKs
- Verification tasks: Run instrumentation tests and Android lint

^ These have changed recently, but very briefly, here are a few of the groups of tasks that are included in the Android Gradle plugin.

---

Groovy / Gradle:

````java
	task hello {
    	doLast {
        	println "Hello NewCircle!"
	    }
	}
````

^ Let's write a task to say hello NewCircle

---

Groovy / Gradle:

````java
	task hello {
    	doLast {
        	println "Hello NewCircle!"
	    }
	}
````

Run:

````bash
	$ ./gradlew hello
````

---

And we get...

````java
♤ ./gradlew hello
:app:hello
Hello NewCircle!

BUILD SUCCESSFUL

Total time: 11.075 secs
````

^ cool. let's try some other things. 
[[ GO TO TERMINAL HERE ]]

---

Gradle:

````java
♤ ./gradlew earlyHello
Hello, very early
:app:earlyHello UP-TO-DATE

BUILD SUCCESSFUL

Total time: 10.184 secs

^ what's going on?

````

Gradle uses closures to build task logic that will be executed at runtime.

^ any logic that isn't wrapped in the doLast{} block gets run during the 'configuration' stage.

^ ok, so with this in mind, let's talk about the phases of a gradle build

---

## Closures

^ Do string closure example.  (lazily eval'd?)

---

# Gradle Build Phases

- Initialization: which projects are being built
- Configuration: Resolves dependency graph, builds a task execution plan
- Execution: Runs the task execution plan

^ [[read slide]]

^ Let's watch this run one more time. See if you can spot where it's switching between the different phases.
	
---

````bash
♤ ./gradlew hello
> Configuring > 2/2 projects

:app:helloAgain
Hello Again, NewCircle!

BUILD SUCCESSFUL
````

---

Groovy:

````java
task hello << {
    println "Hello NewCircle!"
}

task earlyHello {
    println "Hello, very early"
}
````

^ Now let's go back to that broken task.  [[ RUN brokenHello ]]

^ What's happening here?

---

#[fit] Closure magic!

![150%|right](closure_magic.gif)

^ Closure magic!!

^ Gradle takes advantage of Groovy closures to delay execution of task codes until 
the task invoked by either the task execution plan, or from the command line.  Any code 
in a build script that is not wrapped inside of a task's closure block (the magical doLast{ })
will be executed in the configuration phase, not at execution.

---

# Task Actions

- Tasks are a set of “actions”, which are closures
- Actions are run during the Execution phase
- Three ways to add an Action (closure) to a task:
	- doFirst {}
	- doLast {}
	- leftShift {} / <<

^ if we want to get more specific about what a task is, it's a set of actions...
^ all "actions" are run during the Execution phase.

---

## Test Yourself

- What language is Gradle written in?
- What are the 3 phases of a Gradle build?
- What does the `<<` symbol stand for in a Gradle build script?
- `doLast` is a property on a Task. What is its type?
- How do you run a Gradle task from the command line?
- Name 2 types of tasks included in the Android Gradle plugin.

^ Let's go back to the questions that we had at the very beginning.

^ [[ Pause briefly ]].  How did we do?

^ /Users/lisaneigut/Knaps/Android/AndroidTalks/gradle/GradlePractice/gradle/wrapper/gradle-wrapper.properties

---

## Q&A

^ Questions?