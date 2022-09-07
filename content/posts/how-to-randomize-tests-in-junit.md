---
title: "How to Randomize Tests in JUnit with Gradle and JUL"
date: 2021-09-22T08:00:36-04:00
featured_image: "images/how_to_randomize_tests_in_junit.jpeg"
images: ["images/how_to_randomize_tests_in_junit.jpeg"]
tags: ['java', 'testing']
---

The ability to randomize the order of tests has always been a useful tool for diagnosing flaky tests. However, until JUnit 5.8.0, developers were only able to randomize the order of methods within a test class using [MethodOrder$Random](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/MethodOrderer.Random.html). With the addition of [MethodOrderer$ClassOrderer](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/ClassOrderer.Random.html), it is now possible to randomize the test classes too.

For the rest of this article, I will walk you through how to enable test classes and methods to run in a random order with the following steps:
1. Set up JUnit 5.8.0
2. Configure Gradle to display test names
3. Randomize methods and classes
4. Log the seeds with JUL
5. Reproduce failures with the seeds

# 1. Set up JUnit 5.8.0

Start the process by specify the version of JUnit you would like to use in your `build.gradle`.

```groovy
// build.gradle
dependencies {
    // ...
    testImplementation: 'org.junit.jupiter:jupiter-api:5.8.0'
    // ...
}
```

# 2. Configure Gradle to display test names

Add a [testLogging](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.logging.TestLogging.html) container to the `test` block, and list the events you wish to see. I recommend displaying test failures, passes, and skips.

```groovy
// build.gradle
test {
    useJUnitPlatform()

    testLogging {
        events "FAILED", "PASSED", "SKIPPED"
    }
}
```

Now each time you execute the tests task, you will be able to see the names of each test. Having visibility into the order of the tests is helpful in verifying that the test are run randomly, and critical in debugging test pollution. Check out my article about [how to think through and debug test pollution](/posts/thinking-through-test-pollution/). 

# 3. Randomize methods and classes

Controlling the order of the test in JUnit is accomplished by setting the order strategy for both methods and classes. 
Set the fully-qualified class name (FQN) of the `Random` static inner class for the method and class order properties. 

```
// junit-platform.properties
junit.jupiter.testmethod.order.default=org.junit.jupiter.api.MethodOrderer$Random
junit.jupiter.testclass.order.default=org.junit.jupiter.api.ClassOrderer$Random
```

# 4. Log the seeds with JUL

Before you are in a position to reproduce failing tests, you will need to print out the seed values which are sent to the `CONFIG` level logs by JUnit. You will set up configuration for the `java.util.logging` logger (commonly referred to as JUL) below. 

**Please note: JUL is configured _not_ to display `CONFIG` level logs by default.**

Create a `logging.properties` file within the `test/resources` directory of your app and add the following contents to it:

```text
// test/resources/logging.properties
.level=CONFIG
java.util.logging.ConsoleHandler.level=CONFIG
org.junit.jupiter.api.ClassOrderer$Random.handlers=java.util.logging.ConsoleHandler
org.junit.jupiter.api.MethodOrderer$Random.handlers=java.util.logging.ConsoleHandler
```

It is very important that you specify both the root log level and the handler's level with `.level=CONFIG`, or else no logs will appear.

The next step in this process is to make sure that JUL is configured to use the logging settings that you have specified. Include the logging file as a system property in the test configuration of your `build.gradle` file.
Set the `java.util.logging.config.file` property to the path of the settings file that you just created. In the example, I use a relative path because Gradle's [file](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:file(java.lang.Object)) helper resolves relative paths into absolute ones.

```groovy
// build.gradle
test {
    useJUnitPlatform()
    
    systemProperty 'java.util.logging.config.file', file('src/test/resources/logging.properties')

    testLogging {
        // ...
    }
}
```

For the final step, run the Gradle `test` task in debug mode using the `--debug` flag. Then verify that you can see the seeds in the logs.

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

# 5. Reproduce failures with the seeds

Once you detect a failing pair of seeds, you are ready to start the debugging process by utilizing another set of properties for JUnit platform. The seeds will allow you to rerun the tests in a predictable order. Within the `junit-platform.properties` file, make the following changes:

```
// ...
junit.jupiter.execution.order.random.seed=<seed for method order>
junit.jupiter.execution.class.order.random.seed=<seed for class order>
```

{{< thank-you >}}