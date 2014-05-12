# Contents

* [**Introduction**] (#introduction)
* [**Optimizing Gradle integration in your IDE and on the command line**] (#tips-to-speed-up-gradle-if-you-still-want-ide-integration)
 * [**Gradle Daemon**] (#gradle-daemon)
* [**Removing Gradle integration from your IDE**] (#how-to-remove-gradle-ide-integration-from-your-project)
 * [**Intellij IDEA**] (#creating-your-idea-project)
 * [**Eclipse**] (#creating-your-eclipse-project)
* [**LibGDX without Gradle**] (#libgdx-without-gradle)
* [**Why does LibGDX recommend Gradle**] (#why-does-libgdx-recommend-gradle)


## Introduction
Gradle is very powerful, and once you get used to it is a great tool to be versed in and to have in your arsenal, but when mixed with your IDE it can cause workflow problems.  This varies from project to project, and from person to person; here is an example of such an issue:

Having a multi-project, multi-flavour project, where you want to be running your desktop build very often to check your latest changes.  Due to how Gradle works (it allows the flexibility of accessing and changing any part of the build from any part of the project), it must configure all projects in a multi-project setup before any task is executed.  When you have a project with desktop only, this is usually very speedy, but when you add in an android project, html project the time starts to rack up.  This is especially noticeable in Intellij IDEA.

## Tips to speed up Gradle if you still want IDE integration

You can try a few things to get running more efficiently:

### Gradle daemon
The Gradle daemon aims to lower execution time and startup time of tasks especially where tasks are executed frequently. 

How to run with the Gradle daemon
You can run tasks adding the flag `--daemon`
You can also add the option to run on all tasks by editing a `gradle.properties` file that resides in the root directory of your project.
Add the line; `org.gradle.daemon=true`

_Further reading:_
_Daemon http://www.gradle.org/docs/current/userguide/gradle_daemon.html_



## How to remove Gradle IDE integration from your project

_Note: This only applies for Eclipse and Intellij IDEA users_

With the project that is generated from gdx-setup.jar, the Gradle IDEA, and Gradle Eclipse plugins are applied to all projects.
These plugins allow you to generate IDE specific files that you can then work from.

Doing this allows you to keep Gradle out of your IDE, so that when you want to launch any task, it wont configure your whole project and will be as quick as if you had a regular IDE generated project.  This does mean that you need to sort out support for the sub modules in the IDE yourself, android support, robovm ect.  (It is still recommended that you do all GWT/html stuff from the command line, as well as using the great packaging tasks that are supplied)

It also allows you to keep the power of Gradle at your disposal, to handle all your dependencies, packaging tasks, signing tasks, deployment and anything else you may want to implement by running from the command line.

### Creating your IDEA project
```groovy
gradlew idea
```

In IDEA:
File > OPEN > Locate the .ipr that the task above generates
### Creating your Eclipse Project
```groovy
gradlew eclipse
```

In Eclipse:
File > Import > Existing project into workspace > Locate the .project file and import

## Libgdx without Gradle

You are **never** forced to use Gradle, it is just recommended. If you want clarification on why it is recommended, check [here] (#why-does-libgdx-recommend-gradle)

If you don't want to use Gradle, you have the following choices.

* You can use the setup to create your project still and never touch Gradle again. Folow the guide from [here] (#how-to-remove-gradle-ide-integration-from-your-project) and never touch Gradle again.
* You can setup your project manually.
* You can maintain the old setup and use that to generate your projects

Doing either of these will mean: 
* You no longer have access to packaging tasks, you will have to package and deploy your projects yourself
* You will lose support for IDE's other than IDEA and Eclipse
* You install all the plugins required for your IDE to interpret each of the sub-projects
* You manually update your dependencies from now on, don't forget the platform specific natives too!

  
## Why does LibGDX recommend Gradle

#### Support
* Using Gradle, LibGDX is supported regardless of what IDE you use, even if you don't use one.  Your development environment doesn't support Android/RoboVM/HTML/Java? Don't worry, Gradle can compile/build/run/package your project no matter what you use to code with.
* Having one unified system of managing your project makes it easy to get support from other users. The more people that are using the same approach, the larger the support user base grows.

#### Dependency Management
One of the biggest pros of any build/dependency management system.
* Easy to understand
* Insanely quick and simple to stay up to date with the latest LibGDX, update one line-refresh-done. No more jar hunting.
* Transitive dependencies (Inherit dependencies from projects you depend on)
* Custom dependency definitions, depend on anything!
* Full compatibility with all repository types (Maven, Ivy, etc)

#### Flexibility
Gradle is built with Ant and Maven very much in mind, and aims to combine the two to make a very powerful and flexible build automation tool.
* Structure your build however you want, add custom tasks, use the rich API to completely customize your build to suit your needs
* Want to run your project on a CI server? Just throw it up there install the jenkins Gradle plugin and you're good to go
* The Gradle Wrapper allows you to be free of installing Gradle!

#### Maintainability
As LibGDX grows, extensions are created from scratch or by removing them from core where appropriate, making LibGDX more modular to avoid bloat when you don't need those extensions. (This happened with Box2D recently). This means **more** dependencies for you to add and therefore more jars to add.  If you are using a minimalist project with no extensions, if you were to manually manage your dependencies that would mean 1 core dependency, 1 desktop dependency, 4 android dependencies, 4 ios dependencies and 3 gwt dependencies.   Thats 13 jars you need to track down every time you want to update to the latest LibGDX with all the greatest new features.  If you use Box2D, Box2D lights, FreeType, this is going to get quickly very annoying to update and you will probably put off doing it. The setup has been built with this in mind, and makes it very quick to add these to your project and painless to update by changing one line with the power of Gradle.    

The setup itself is very simple, very transparent and maintainable by developers that didn't create it.  This lowers the bus factor for LibGDX and reduces strain when the setup requires an update/alteration.

As previously mentioned, the Gradle project that is created by **gdx-setup.jar** includes tasks to run and package/distribute each project.  This allows LibGDX to have a unified way of packaging your project, rather than x amount of ways for y amount of IDE's.  This saves a lot of time in terms of writing documentation, following what every IDE is doing, and providing support on the forums; this gives developers more time to work on great features for LibGDX to help make your project awesome.

#### It revolutionizes everything
Look down. Now look back up. Look down again. You're siting on a chair. Look back up. You aren't using Gradle, your life is OK but it could be better. You read this Article and you decide you want to use Gradle. You use the **gdx-setup.jar** to generate your project and you get tinkering with it.  You use the power of Gradle to create 500 flappy clones from the same source code. You look down again, your chair is now made of gold and there are an infinite amount of monkeys with keyboards to code for you. Your life is now better and you can now concentrate on this things that matter.
