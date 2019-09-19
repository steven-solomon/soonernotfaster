---
title: "Which Type of Tests Should You Write?"
featured_image: "images/which_type_of_tests_should_you_write.jpeg"
date: 2018-09-10T19:55:33-04:00
---

When test driving a system, it can be tricky to know which type of test to use and when. Rather than talk about the [testing pyramid](https://martinfowler.com/bliki/TestPyramid.html), I want to focus on the questions we want answers to.

Below I have organized feature tests, integration tests and unit tests by the questions that they answer.

# Is It Wired Together?

Whenever I start working on a new feature for a system, I begin by writing a failing feature test. Feature tests usually travel a “happy path” through the entire system. Focusing on the flow of the system, rather than testing out edge cases within each layer.

The reason that I start with this type of test is that I want to know the answer to the question *“Is the System Wired Together Correctly?”*. While completing future stories, I may not even change this test.

Feature tests are usually slower than other types of tests, this is due to the fact that they use the entire system and most of the time a web browser.

Web browsers weren’t designed to make automated testing easy (though that is changing). So leaning on them to make sure every aspect of the application works will result in slow tests — that we probably won’t want to run as often. Running tests less often means that it takes more time to find out if something is broken.

# Does It Work Like I Expect?

While making the feature test pass, I eventually have to integrate with an external system, such as an API, or a database. Leading me to ask *“Is my code integrated with the external system?”*. This brings the need for an Integration test.

I usually keep the integration layer as thin as possible, because integrations can be flaky or painfully slow. When integrations begin to make my tests flaky — not consistently passing or failing — then I use a contract test to verify behavior.

# Does It Work Well?

With my feature tests verifying that I have wired the system together, and integration tests showing that I can work with external systems. All that is left is answering the question “Does it work well?”. Said another way, *“Can my system do all the things it needs to and handle edge cases?”* This is best answered by unit tests.

A unit test can be at any level of granularity we like. Large grained tests work with multiple objects and functions, whereas small grained tests interact directly with one object or function.

When deciding on test size, I recommend starting large at the highest level of abstraction available and letting the test grow over time. Once the test is too hard to reason about, or you and your pair feel that the design of the system is unlikely to change, then break up the test to be smaller grained.

# Conclusion

We have discussed the outcomes each type of test can bring to a codebase. Focusing on outcomes is beneficial to many aspects of a project. In this case, it will lead us to a better-designed test suite, and more confidence as we ship early and often.

*Photo by Viswanath V Pai on Unsplash*

**If you enjoyed this post, please share it on your social media. Thanks!**
