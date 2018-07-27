# Gradle Parent Build Example Reference Project

An example reference project representing an artifact encapsulating a Gradle parent build file. This is used in conjunction with the 
[Pater plugin](https://github.com/boxheed/gradle-pater-build-plugin). The plugin works
by scanning the classpath for Gradle build files, then applying them in the order you declare 
them using - internally - the _apply from_ syntax. This means we could have several super build files,
not just one parent.

Regardless, the one is currently intended as the parent.

### Features
* Specifies a set of dependencies / versions to be used within your projects, and a task to list them.
* Sets sensibile defaults for repositories: mavenLocal(), jCenter(), mavenCentral(), 
and an example Company Artifactory
* Applies the following plugins to all projects: java, jacoco, and sonarqube
* Sets sendible defaults for Jacoco and Sonarqubue (though implementing projects can override)
* Adds two new test tasks: ___testUnit___ and ___testIntegration___, which both take advantage of the 
JUnit Category feature. 
    * testUnit looks for tests marked with the com.company-name.test.junit.UnitTest class
    * testIntegration looks for tests marked with the com.company-name.test.junit.IntegrationTest class
    * both classes can be found within the _company_name-test-junit_ module. (use in your project via a testCompile libraries.company-name.test)
* Enforces jacocoTestReport to execute after successful runs of the _test_ task
* Enforces _testUnit_ to run before the _jar_ task. This ensures that fast, non-integration tasks execute before a subproject
is built, say, during a _run_ execution.


### Usage

#### Importing the parent file
Usage is as simple as importing a buildscript dependency... however, you must also include the Pater plugin. So, the 
first few lines of your parent file like like:

    buildscript {
        repositories {
            mavenLocal()
            // add the company-name repo here. Actual config TBD after this is deployed
        }
    
        dependencies {
            classpath 'com.fizzpod:gradle-pater-build-plugin:1.1.1'
            // actual version of the parent build tbd
            classpath 'com.company-name:company-name-gradle-parent-build:0.0.1'
        }
    }
    
To ensure it worked, run `gradle tasks` or `gradle tasks --all`, you should see `testUnit`, `testIntegration`, and various other
tasks, like `jacocoTestReport`

#### Using dependencies
The plugin provides a set of libraries that can be referenced in your project, by referencing a name on an injected 'libraries'
object, like so:

    dependencies {
        compile libraries.rxjava2
        testCompile libraries.junit
        testCompile libraries.company-name.test
    }
    
To see what's available within `libraries`, you can run `gradle listCompanyNameLibraries` or `gradle lTL`.

#### Taking advantage of testUnit and testIntegration

This file adds two new tasks: `testUnit` and `testIntegration`. In addition, `testUnit` is executed whenever a project / module
is packaged in a jar. This typically happens during a build or during a run of some main module (e.g. 'app') that has 
dependencies on it's co-modules. The idea here is that testUnit is executed during this phase, letting us know if anything
has broken. Tests run during testUnit should be pure unit tests, executing quickly.

To mark tests as being _Unit_ or _Integration_, there exist two classes within the `libraries.company-name.test` 
dependency, _com.company-name.test.junit.UnitTest_ and _com.company-name.test.junit.IntegrationTest_. Mark
tests within your project using JUnit @Category and those interfaces to hook them into the tasks.

 
