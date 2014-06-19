This project represents a template for what a LifeWay API project should look
like. We're striving for a common, consistent Gradle build, instead of each
project using different wrapper versions and tool.  This will still maintain
each project's isolation and keep the build as transparent as possible. While at
the same time we want to be able to push out build updates in a predictable
fashion.

## Features

The template features:
* Consolidated customizations into build.gradle, moving generic code into individual files in the gradle directory
* Jenkins friendly version of gradlew
* Able to publish to maven central
* Capable of also working with Artifactory
* Find Bugs, PMD and CheckStyle
* Build source jar and javadoc jar
* Cobertura for code coverage
* Release plugin to build, tag, and publish
* Header checking for headers

## Merging build files

Merging in the build should be a four step process: merge, customize, test and delete pom.xml. In general, this is only done once and while we work out the kinks, Engineering Tools is willing to do the initial port to Gradle and provide a pull request, just contact EngineeringTools@netflix.com

### Merging
This will pull in the template:

    git remote add --track $BRANCH build git@github.com:Netflix/gradle-template.git
    git fetch build
    git merge build/$BRANCH

### Customize

* Edit settings.gradle, set the name of the project and to list all the submodules you have.
* Edit gradle.properties, set the version to the *next* version with -SNAPSHOT on the end
* Edit build.gradle, customize ext.githubProjectName - Used to construct a few urls in the pom.
* Edit build.gradle, customize group - make sure it represents the group that you want to publish into maven/ivy with.

If in a multi-module project, edit the build.gradle and create a project block for each project, there are two by default which you override.

The dependencies will have to be changed to match the pom dependencies. In the single-project branch, simply follow the dependencies block. In the multi-project branch, there are a few dependencies sections. The first one, part of subprojects, is common to all projects. And then there's a dependency block for each project.

### Delete the pom.xml

    find . -name 'pom.xml' -exec git rm \{\} \;

## Updating Template

As updates are put into the gradle-template project, individual OSS Projects will need to merge in those changes. Any changes will be announced to the DL Open Source Software.  To perform that merge run this, using the branch you choose when you first setup the project:

    git fetch build
    git merge build/$BRANCH

## Artifactory

By itself the above build.gradle will pull from Maven Central and publish to Maven Central, to use Netflix infrastructure we provide an "init script" to orchestrate Gradle to use artifacts.netflix.com. This pulls in artifactory.gradle, which is sourced from //depot/Tools/nebula-boot/artifactory.gradle. It's job is to:

* Set ivy cache directory to a Jenkins friendly one
* Add jfrog build-into plugin to classpath, add it as a plugin and configure it to publish to {{libs-$\{project.status}-local}}
* Remove all repositories
* Add artifacts.netflix.com/ext-releases-local as the only repository

Two targets available to use artifactoryi:
* uploadArtifactory - Call out to compile against internal repository
* buildWithArtifactory - Build against internal repository

## Properties

Gradle has a few ways to inject properties into the project, which can then be used in the DSL.
It can done on the command line via the "_\-P_ option,

e.g.

    ./gradlew -Pmyproperty=myvalue_".

It can also be done via Java System Property as long as it's prefixed with ORG\_GRADLE\_PROJECT,

e.g.

    ./gradlew -DORG\_GRADLE\_PROJECT\_myproperty=myvalue

To avoid having to pass in some properties every time on the command line, they can
be stored in <em>$HOME/.gradle/gradle.properties</em>.
