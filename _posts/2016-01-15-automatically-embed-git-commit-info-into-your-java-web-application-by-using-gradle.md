---
layout: post
title: "Automatically Embed Git Commit Info Into Your Java Web Application by Using Gradle"
modified:
categories: 
excerpt: "This tutorial will help you to extend your Gradle script, and automatically generate and embed a version.properties file into your .war file."
tags: [gradle, git, java, web, application, war, version, properties, commit, id, revision]
comments: true
image:
  feature:
date: 2016-01-15T22:36:42+01:00
---

This tutorial will help you to extend your Gradle script, and automatically generate and embed a `version.properties` file into your .war file. The great thing about this approach is that it's quite simple, requires no additional plugins (except `java` and `war`), and you can easily adjust it to suit your needs. Additionally, working Gradle is all that you need - there's no need for external installations (not even a Git command line tool).

### 1. Add a Build Dependency

The first thing we have to do is to include the [Grgit](https://github.com/ajoberstar/grgit) library as a buildscript dependency. Grgit provides an easy way to interact with Git from Java or Groovy code.

```
buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath 'org.ajoberstar:grgit:1.4.1'
    }
}
```

### 2. Add a Task

Now we're going to define a custom Gradle task:

```
task generateVersionProperties << {
    def git = org.ajoberstar.grgit.Grgit.open(file('.'))
    def commit = git.head()
    
    def commitId = commit.id
    def commitDate = commit.getDate()
    def buildDate = new Date()

    File resourcesDir = new File(project.getBuildDir(), 'resources/main')
    File propertiesFile = new File(resourcesDir, 'version.properties')

    if(!propertiesFile.exists()) {
        resourcesDir.mkdirs()
        propertiesFile.createNewFile()
    }
    
    propertiesFile.text = """git.commit.id=${commitId}
git.commit.time=${commitDate.format('dd.MM.yyyy HH:mm:ss')}
build.time=${buildDate.format('dd.MM.yyyy HH:mm:ss')}
"""
}
```

1. In the first line, we specify a path to the local Git repository. In our case, `gradle.build` script is in the repository root folder, so we use `.` as a relative path. As a result, a [Grgit object](https://github.com/ajoberstar/grgit/blob/master/src/main/groovy/org/ajoberstar/grgit/Grgit.groovy) is created.
2. Method _head()_ will give us a [Commit object](https://github.com/ajoberstar/grgit/blob/master/src/main/groovy/org/ajoberstar/grgit/Commit.groovy) pointing to the current HEAD of the repository.
3. Now we can access Commit object's fields like _id_, _author_, _fullMessage_... And methods like _getDate()_ and _getAbbreviatedId()_.
4. A `version.properties` file is going to be created within a `<build_dir>/resources/main` directory every time our custom task is executed. Typically, `<build_dir>` isn't under the version control system, so there's no fear of affecting the source code every time the task is triggered.
5. Finally, the `version.properties` file is going to be populated with some useful version control and build info. We want to store a Git commit ID, commit date and application build date. We chose a .properties file format because it's convenient for later usage in Java code.

### 3. Set the Dependency for the 'war' Task

```
project.tasks.war.dependsOn('generateVersionProperties')
```
Now, when you build your .war file by executing the `gradle war` command, the `version.properties` file is going to be embedded into `<your_app>/WEB-INF/classes/` directory.

### Summary

A very simple, but complete Java web application example **source code**, based on this tutorial, can be found on [https://github.com/skomarica/gradle-git-version-webapp](https://github.com/skomarica/gradle-git-version-webapp). A version file is going to be created during the build. Simple servlet will load the file and return a response in a JSON format. 

1. Clone the repository
2. Run `gradle war` (or `gradle jettyRunWar` if you use the `jetty` plugin). A version file is going to be created and embedded into a .war file, and it can be found on the path `ggv/WEB-INF/classes/version.properies`
3. Access the application version URL: `http://<host>:<port>/ggv/version`. Simple AngularJS frontend will fetch and display data from the backend Version Servlet.

As you can see, it's fairly easy to automate your versioning and build process by combining such a great tools like Gradle and Git!
