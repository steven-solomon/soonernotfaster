---
title: "What Is a Unit Test Anyway"
date: 2021-12-28T10:42:34-05:00
draft: true
---

They say one unit test per function; You say one unit test per idea. Or maybe it’s the reverse. Since I started teaching developers test-driven development in 2010, I have rarely found a space where the definition of the word "unit" didn’t yield a giant debate—even around those folks who themselves are experienced TDD practitioners. Rather than retread the arguments around categories, I would like to focus on the outcome that having unit tests can create.

The most important outcome of unit tests is that the intention for part of the system is communicated between software developers. Each test suite is a living document that makes it easy for engineers to see the element(s) being tested, the context, and the expectations for some functionality of the system. And its accuracy is verified when the tests pass. With the purpose of a unit test defined, we can talk about different strategies to make this intention clear: the parts of a unit test, how to structure a suite of tests, and how to reduce the test’s coupling to the implementation.


# What is a unit

A unit test can contain any number of classes or functions that represent the test subject. They should follow the constraints laid out by Robert Martin in his book Clean Code, namely that they should be Fast, Independent, Repeatable, Self-validating, and Timely. These criteria vastly limit what can be present in the context of a unit test. For instance a slow database query can't be within the execution path of the test subject or the test will not be fast. And if multiple tests interact with an external system that provides both read or write capabilities like a REST or GraphQL API then the tests won't be independent, making them likely to be flaky, flickering green and then red, with no change to the application code. 

So in general, a unit test does not include any elements that are outside the system being tested, this includes a database, external HTTP based API, interactions with the file system. However, a unit test can contain multiple collaborating objects if the test author wants to create tests that still have  

# What are the parts of a unit test

Typically, you will hear engineers talk about three parts of a unit test—there are actually four which I will discuss in a moment. The three most discussed parts are the setup, the exercise of the system under test (SUT), and the verification. Teams will likely refer to this trio as “arrange-act-assert”.

The setup (or arrange) part of the test is an opportunity to demonstrate the context necessary for the code being tested to demonstrate some interesting behavior. The context is can be communicated by demonstrating a set of [preconditions](https://en.wikipedia.org/wiki/Precondition) that must be met for the code to have the desired behavior, or the demonstration of how the SUT acts when the preconditions are not met.

The exercise (or act) step is the simplest of the parts of a unit test as it is just the interaction with the code that will trigger interesting behavior. The act is usually a method call when using theObject Oriented (OO) paradigm, or a function call when taking a Functional approach. This step helps communicate the subject of the test.

The verification (assert) step is where the interesting behavior is checked. When using OO to design your SUT it is likely that you will verify the desired behavior through the use of test doubles, but there are scenarios where you will verify that state has been mutated through a change of behavior. Behavior changes infer that some internal state on the object exists causing it to now do something else. This behavior is observed through its interaction with peer objects or through the return value of a method. In the case of Functional programming, you will verify that the return value meets the desired criteria. In both styles of programming, it is useful to build up [custom assertions](http://xunitpatterns.com/Custom%20Assertion.html) which can make the intention clearer to the reader of the test. Although most teams use a matcher library such as Hamcrest, custom assertions are often simpler to read when constructed from helper methods within the test as it can emphasize "the why" in a way that is more comprehensible to your team.

The fourth part of a test is its name. The name of a test is another opportunity to try to communicate part of your intention (the context, subject, and expectations) to your reader. Depending on the testing library that you are using the test name may also be made up of a `describe`, and/or `context` block, in addition to the string that appears right before the test scenario. The test name can contain the expectation, subject, or aspect of the context that you want to share. For instance, `returnsEmptyList_whenAllInputsAreOdd` can communicate that there is some filtering behavior present in the SUT. 
