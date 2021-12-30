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

["Test everything that could possibly break," says the Extreme Programming Community](http://wiki.c2.com/?TestEverythingThatCouldPossiblyBreak). That might seem, well extreme, but let's try to understand what could possibly break. Using the lens of communicating intention that we discussed earlier, what do we feel it is important to communicate to our readers? For the example of grouping products by their name, it is important to communicate that empty input results in empty output, a single product is grouped by its name, duplicate products are grouped by the same key, and two different products are not grouped. Notice that I did not mention code coverage instead I focused on representing each behavior that I wanted to communicate to my fellow engineers, and I did it in an almost mathematical sense.   

Generally I try to write tests that describe how the code changes in each context. This requires an understanding of the different contexts that are possible for the code that I am trying to describe the intention of. 

But what about tests that are duplicate of the implementation? A well-structured unit test represents its verification in a way that is distinct from the implementation. This isn't just for pursuit of some mystical ideal, it has a very practical purpose. We want to be able to change the implementation and the tests not break. In order to achieve this goal, we need to minimize the concepts that the test knows about the implementation.

Now for the controversial, but pragmatic take, don't write tests that you and your team think are worthless. (TK maybe this can be in the lede?) You don't have to write a test for every method on every class; Focus on where there is meaningful business logic. But if the code ever does break, try to identify why, and then write a test.

```ruby
class ListNode
  # ruby functionality for instance variables and generated setters and getters
  attr_accessor :next, :prev, :value
end

# Tests that you don't need to write
describe ListNode do
  describe '.next' do
    it 'sets next' do
      tail = ListNode.new(nil)
      head = ListNode.new(nil)
      
      head.next(tail)
      
      expect(head.next).to eq(tail)
    end
  end
  
  describe '.value' do
    it 'sets value' do
      value = 10 
      head = ListNode.new(value)
      
      expect(head.value).to eq(value)
    end
  end
end 
```

There isn't really behavior in the above examples, but it may be beneficial to test the interaction between two elements. For instance, if when `next` is set on a `ListNode` then it would be good to verify that `prev` is invoked on the node. The code below now has a bit of logic that we want to communicate to follow engineers, `next` isn't just a setter now, it has some magic to it. We can even further emphasis this distinction by renaming `next` to a verb like `add_next`. Even this is on the edge of value but keep in mind that there is another case that has to exist in this code that should be handled, the scenario where there is already a next value.

```ruby
describe ListNode do
  describe 'when a node is passed to next' do
    it 'has prev set to the previous node' do
      tail = ListNode.new(nil)
      head = ListNode.new(nil)
      
      head.add_next(tail)
      
      expect(tail.prev).to eq(head)
      expect(head.next).to eq(tail)
    end
  end
end 

# Implementation
class ListNode
  def add_next(node)
    self.next = node
    node.prev = self
  end  
end
```
