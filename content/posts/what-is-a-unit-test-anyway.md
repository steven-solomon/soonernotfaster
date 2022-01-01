---
title: "What Is a Unit Test Anyway"
date: 2021-12-28T10:42:34-05:00
draft: true
---

# What are the parts of a unit test

Typically, you will hear engineers talk about three parts of a unit test—there are actually four which I will discuss in a moment. The three most discussed parts are the setup, the exercise of the system under test (SUT), and the verification. Teams will likely refer to this trio as “arrange-act-assert”.

The setup (or arrange) part of the test is an opportunity to demonstrate the context necessary for the code being tested to demonstrate some interesting behavior. The context is can be communicated by demonstrating a set of [preconditions](https://en.wikipedia.org/wiki/Precondition) that must be met for the code to have the desired behavior, or the demonstration of how the SUT acts when the preconditions are not met.

The exercise (or act) step is the simplest of the parts of a unit test as it is just the interaction with the code that will trigger interesting behavior. The act is usually a method call when using theObject Oriented (OO) paradigm, or a function call when taking a Functional approach. This step helps communicate the subject of the test.

The verification (assert) step is where the interesting behavior is checked. When using OO to design your SUT it is likely that you will verify the desired behavior through the use of test doubles, but there are scenarios where you will verify that state has been mutated through a change of behavior. Behavior changes infer that some internal state on the object exists causing it to now do something else. This behavior is observed through its interaction with peer objects or through the return value of a method. In the case of Functional programming, you will verify that the return value meets the desired criteria. In both styles of programming, it is useful to build up [custom assertions](http://xunitpatterns.com/Custom%20Assertion.html) which can make the intention clearer to the reader of the test. Although most teams use a matcher library such as Hamcrest, custom assertions are often simpler to read when constructed from helper methods within the test as it can emphasize "the why" in a way that is more comprehensible to your team.

The fourth part of a test is its name. The name of a test is another opportunity to try to communicate part of your intention (the context, subject, and expectations) to your reader. Depending on the testing library that you are using the test name may also be made up of a `describe`, and/or `context` block, in addition to the string that appears right before the test scenario. The test name can contain the expectation, subject, or aspect of the context that you want to share. For instance, `returnsEmptyList_whenAllInputsAreOdd` can communicate that there is some filtering behavior present in the SUT. 
