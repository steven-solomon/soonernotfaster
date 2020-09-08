---
title: "Writing Maybe in Ruby: Lessons Learned"
featured_image: "images/writing_maybe_in_ruby_lessons_learned.jpeg"
images: ["images/writing_maybe_in_ruby_lessons_learned.jpeg"]
date: 2018-07-28T20:46:03-04:00
---

I built a [small framework](https://github.com/steven-solomon/maybe), which retaught me why it is important to understand a user’s problem, ship early and iterate. Yes, even for frameworks.

It kicked off with a tweet, which expressed disdain for Ruby’s nil. The author yearned for a maybe implementation. Having just finished the Bowling Kata in Elm, a language which has a built in Maybe type, I decided I could create a quick implementation. I didn’t anticipate how long quick was…

# The Design

My first iteration was rather simple. I would model my implementation off of SmallTalk’s `ifTrue` and `ifFalse` methods.

I let RubyMine generate a new Gem and I started writing tests. First I would monkey-patch Object with a method `maybe`, that would execute a block and return self.

```ruby
Object.new.maybe { puts 'success' }
# prints 'success'
# returns #<Object:0x00007fc7290347e0>
class Object
  def maybe
    yield
    self
  end
end
```

With maybe returning itself, I could add another method `else`. Now I could call both methods in sequence. Soon I it would allow client code to not know which type it received.

```ruby
Object.new.maybe { puts 'success' }.else { puts 'false' }
# prints 'success'
# returns #<Object:0x00007fc7290347e0>
class Object
  ..
  def else
    self
  end
end
```

Now I would implement the corresponding behavior for `NilClass`. It would have the same methods but only execute the else block. I didn’t have to return self as methods already return nil but default.

```ruby
nil.maybe { puts 'success' }.else { puts 'false' }
# prints 'false'
# returns nil
class NilClass
  def maybe
  end
  
  def else
    yield
  end
end
```

# Shipping Early

I briefly celebrated my turn around time by walking to my favorite coffee shop and ordering a large coldbrew. When I returned to my apartment, I proudly tweeted out my Github link and I thought I was done… Oh was I wrong.

What followed was a few hour back and forth as a real person attempted to use my “framework” for the first time. I relearned the lesson, **writing shareable code is hard.**

# Incorrect Documentation

The first thing I forgot to update was the default usage documentation that RubyMine generates. As I didn’t do a proper Gem release, it’s recommendation to `bundle install maybe` resulted in another library from being pulled in. So, I updated the documentation and again thought I was finished.

I also realized that it might be valuable to pass the Object instance into the maybe block so that methods could be called on it.

```ruby
class Object 
  def maybe
    yield self
    self
  end
end
```

# Iterating

A short time later, my single customer reported some unexpected behavior. He expected a `maybe/else` chain on an object instance to return the result of the `maybe` block and not the object.

```ruby
6.maybe { 7 }.else { 8 }
# returns 6
# expected 7
```

Well at least I had a test case. So I wrote a new test around the object, this time using both the maybe and else together. Due to the way some objects are frozen in ruby, I had to create a new `Else` class to contain the else behavior, returning the result of the `maybe` block and ignoring the else block.

```ruby
class Else
  def initialize(value)
    @value = value
  end

  def else
    @value
  end
end

class Object
  def maybe
    Else.new(yield self)
  end
end
```

Finally, I had delivered some value. My customer could now use my makeshift framework as desired. Was it helpful, I’m not sure but this exchange *retaught* me a few things that I want to highlight for you.

# Take time to understand the user’s problem

Just because I am writing a framework doesn’t mean that I shouldn’t take the time to understand what users want. Software engineers are people too. When I didn’t take the time to understand the scenario I was solving, I created more problems. This isn’t to say that thinking harder could prevent future iterations, rather deep insights only come from shipping early and responding to feedback.

# Ship it & Respond To Feedback

**Software is never finished**, it is very important to listen to user pains and respond. Maybe the first prototype got them some of the way there and they got stuck. Maybe they wouldn’t actually use that in practice. Even in the best case, where you deliversd value, the users probably want more. Shipping and listening to feedback is the best way to know.

# Conclusion

Building software is challenging. Even something as simple as 15 lines of code and 9 tests can require a lot of craft and thoughtfulness, less you make peoples’ lives worse. Hopefully you will learn from my failures and take the time to understand the problems customers are trying to solve, ship early and iterate.

### Special Thanks
Thanks to [J.B Rainsberger](https://twitter.com/jbrains) for participating.

{{< thank-you >}}
