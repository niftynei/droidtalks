# Gradlin'

## __Pluggin' it in for Build Success__

#### Lisa Neigut __DroidCon UK__ Oct 31

^ Hello. Good afternoon.  This session we're talking about some strategies for organizing your Gradle build logic (and building plugins!).  
It turns out that that's a lot of ground to cover in 40 minutes.

^ Drink up that coffee. Let's learn some Gradle.

---

# Quick Overview of Gradle Lifecycles (??)

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
````and

Once the projects are created, each of them will have a set of tasks created for them in the
gradle task console.  To reference tasks on a specific project, you can prepend the project name, like so

`:app:assemble`

---

Setting up and configuring multi-project builds

The projects(:projectName) block
the allProjects() block

---

# Plugins

---

- Build Script
- `buildSrc` project
- Standalone project / jar

---

"tasks" you can do with plugins:
- Add a new task 
- Add a method or information about a build

---

How to get more input from the build itself...

Use an extension object.



