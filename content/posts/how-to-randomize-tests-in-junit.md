---
title: "How to Randomize Tests in JUnit with Gradle"
date: 2021-09-22T08:00:36-04:00
featured_image: "images/how_to_randomize_tests_in_junit.jpeg"
images: ["images/how_to_randomize_tests_in_junit.jpeg"]
tags: ['java', 'testing']
---

The ability to randomize the order of tests has always been a useful tool for diagnosing flaky tests as it exposes the interdependency between tests. However, until JUnit 5.8.0, you were only able to randomize the order of methods within a test class using [MethodOrder$Random](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/MethodOrderer.Random.html). Now you can also randomize the class order!

Lets setup version 5.8.0 of JUnit, and demonstrate how to run classes and methods randomly.

# 1. Setting up JUnit 5.8.0

Start the process by specify the version of JUnit you would like to use in your `build.gradle`.

```groovy
dependencies {
    // ...
    testImplementation: 'org.junit.jupiter:jupiter-api:5.8.0'
    // ...
}
```

With the version of JUnit specified, turn your attention towards logging out the names of each test. This is done by specifying the types of logging events you would like Gradle to display. Configure the [testLogging](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.logging.TestLogging.html) container to display failures, passes, and skips.

```groovy
test {
    useJUnitPlatform()

    testLogging {
        events "FAILED", "PASSED", "SKIPPED"
    }
}
```

Now each time you execute the tests you will be able to see the names of each test; This is helpful in verifying that the test are run randomly, and which ones have failed.

# 2. Randomizing methods and classes

Controlling the order of the test in JUnit is now easily accomplished by setting the order strategy for both methods and classes. 
Set the fully-qualified class name (FQN) of the `Random` static inner class for the properties for both types of ordering. 

```
junit.jupiter.testmethod.order.default=org.junit.jupiter.api.MethodOrderer$Random
junit.jupiter.testclass.order.default=org.junit.jupiter.api.ClassOrderer$Random
```

# 3. Logging the seeds

Before you are in a position to reproduce failing tests, you will need to print out the `CONFIG` level logs within JUnit. You will accomplish this by setting up configuration for the `java.util.logging` logger (commonly referred to as JUL) that JUnit uses. JUL is configured to not display `CONFIG` level logs by default.

Create a `logging.properties` file within the `test/resources` directory of your app. And add the following configuration:

```text
// test/resources/logging.properties
.level=CONFIG
java.util.logging.ConsoleHandler.level=CONFIG
org.junit.jupiter.api.ClassOrderer$Random.handlers=java.util.logging.ConsoleHandler
org.junit.jupiter.api.MethodOrderer$Random.handlers=java.util.logging.ConsoleHandler
```

**It is very important that you specify both the root log level with `.level=CONFIG` and the handler's level, or else no logs will appear**

The next step in this process is to make sure that JUL is configured to use the logging settings that you have specified. Include the logging file as a system property in the test configuration in your `build.gradle` file.
Set the `java.util.logging.config.file` property to the path of your logger settings file. Gradle's [file](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:file(java.lang.Object)) helps resolve relative paths into absolute ones.

```groovy
test {
    useJUnitPlatform()
    
    systemProperty 'java.util.logging.config.file', file('src/test/resources/logging.properties')

    testLogging {
        // ...
    }
}
```

For the last step in the setup process, is to run any gradle command in debug mode using the `--debug` argument. This will make sure that you can see the seeds in the logs.

```bash
$ ./gradlew clean test --debug
```

Within the logs you will see the `CONFIG` level output for the randomized tests.  
```
CONFIG: ClassOrderer.Random default seed: 45792142505802
Sep 24, 2021 8:43:25 AM org.junit.jupiter.api.ClassOrderer$Random lambda$getCustomSeed$3
CONFIG: Using custom seed for configuration parameter [junit.jupiter.execution.order.random.seed] with value [80732495619230].
Sep 24, 2021 8:43:25 AM org.junit.jupiter.api.MethodOrderer$Random <clinit>
CONFIG: MethodOrderer.Random default seed: 45792178997002
Sep 24, 2021 8:43:25 AM org.junit.jupiter.api.MethodOrderer$Random lambda$getCustomSeed$3
CONFIG: Using custom seed for configuration parameter [junit.jupiter.execution.order.random.seed] with value [80732495619230].
```

# 5. Reproducing failures

Once you detect a failing pair of seeds, you are ready to start the debugging process by utilizing another set of properties for JUnit platform. These will run the test in a consistent order using the seeds. Below are the two values you will have to set in your `junit-platform.properties` file.

```
// ...
junit.jupiter.execution.order.random.seed=<seed for method order>
junit.jupiter.execution.class.order.random.seed=<seed for class order>
```
