---
title: "What Is a Unit Test Anyway"
date: 2021-12-28T10:42:34-05:00
draft: true
---

Is a unit big? Is a unit small? Is it a function? Or even a whole class? These questions and their kin plague many engineers who are starting to use tests in their work. But in my 10+ years teaching developers test-driven development focusing on the semantics of the word "unit" is a lost cause—even around those folks who themselves are experienced test writers. Instead, I would like to propose that we focus on the outcome we are trying to drive.

TK add the concept of "who writes a unit test to this lede"
TK make it clearer "what are unit tests used for"
TK the value of testing is to ship with more confidence 

As a developer, the point of a unit test is to communicate your expectations about code to other developers. These expectations should make the context, element being tested, and expected output as clear as possible. The context is the scenario in which code is being tested, including dependencies (a.k.a. collaborating objects) and state that represent the pre-conditions. The element is the piece of software that you want to test, it may be a function, it may be a method, or it may be a series of interactions with a function or group of methods that you wish to communicate to your collegues. Lastly, the expected output is code that verifies the observable effect of the code being tested has worked. It could be the result of a function or the mutation of state—depending on which methodology you are using to structure the code you are testing.   

# The parts of a unit test

For every test you write, you want to convey three things to your reader: the context, the software element, and the expected output. It is also important to try to convey those in as few parts as possible. In order to communicate the context, you will need to construct any dependencies for the software element—even the element itself. This is commonly referred to as "arrange".
Once the element is set up, it will need to invoke the part of the system under test (sut). You can refer to this as "act". Lastly, you will need to check the post-conditions are correct using assertions. This final step can be referred to as assert. The three parts, are referenced by arrange-act-assert on most teams, but some call it "setup-exercise-verify".

Although, most teams only discuss the three parts of the unit test, there is actually a forth, its name. The name is also an opportunity to emphasize the elements that you want to communicate to the developer who is reading your code.

Below are two examples of how to structure the elements of a unit test in both Java and JavaScript. The Java example uses JUnit 4 syntax and an assertion library known as AssertJ to communicate context (pre-conditions), the software element (sut), and the expected output (post-conditions) both via the content of the test and the test's name. Within the JavaScript example the same information is communicated via nested blocks—this nesting style is now possible in Java via JUnit 5. 

```
@Test
void returnsMapWithOneElement_whenListOnlyContainsDuplicates() {
  final String duplicateName = "duplicate";
  final Cart cart = new Cart();
  cart.add(List.of(makeProduct(duplicateName), makeProduct(duplicateName)));
  
  final Map<Product, Integer> result = cart.productsGroupedByName();
  
  assertThat(result)
    .containsKey("duplicateName")
    .contains(entry("duplicateName", makeProduct(duplicateName)));
}
```

```javascript
describe('.groupProductsByName', () => {
    context('when list contains duplicates', () => {
        it('returns set with one element', () => {
            const duplicateName = "duplicate";
            const cart = new Cart();
            cart.addProducts([makeProduct(duplicateName), makeProduct(duplicateName)]);

            const result = cart.productsGroupedByName();

            expect(result).toEqual(
                { [duplicateName]: makeProduct(duplicateName) }
            );
        });
    });    
});
```

The tradeoff between these, the nesting and helper method approaches, are in the number of tests. Nesting can be a beneficial way to isolate groupings of tests, much like a subheading in an article. However, nesting too deeply can make the context that the test is executing hard to change and fragile. Conversely, the Java code shows the helper method structuring where common setup methods are created to hide the noise of the setup away from the reader so they can quickly glean the context. The helper method style can suffer when there are a large number of tests; the reader will not be able to quickly determine which tests are related or similar scenarios.

Although these two styles are displayed separately, the easiest to read tests use a mix of both strategies. They organize the tests into logical groupings and utilize helpers methods to create dependencies that are common across related contexts.

# What should be tested in a unit test

Here is the moment where we talk about the much debated meaning of the word "unit". As I said in the opening of this article, it is important to do away with debates about approach and instead focus on what the desirable outcome looks like. We want to create tests that are give us confidence that all combination of pre- and post-conditions will work. 

scope of the test 
white box vs black box
test coupling 
