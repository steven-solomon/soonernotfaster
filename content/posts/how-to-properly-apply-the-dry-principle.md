---
title: "How to Properly Apply the Dry Principle"
featured_image: "images/how_to_properly_apply_the_dry_principle.jpeg"
date: 2018-05-17T08:15:47-04:00
---

DRY is commonly interpreted to mean that code should not have duplicate structures. For instance two array indexes that both use `rand(4)` would be refactored away. Let’s look at an example of code that does exactly that.

```ruby
Person.new(names[rand(4)], jobs[rand(4)])
```

At some point, this is perceived as duplication and an abstraction is put in place. This abstraction is not logically consistent with the domain logic.

```ruby
Person.new(random_item(names), random_item(jobs))
```

At first this inconsistency is not inherently dangerous. The danger comes later, when functionality is built on top of it. The abstraction is cemented in place, making it exponentially harder to remove. Eventually, a behavior is requested that violates the original assumptions of the abstraction. Rather than replacing the now incorrect abstraction, engineers simply tweak the abstraction to work. This causes bugs to start cropping up in seemingly unrelated parts of the app.

Take a look at the code in :random_item, it has been modified to handle different data structures. Resulting in code that mixes two different concepts together. It is now possible to introduce a bug in the Array processing when modifying the Hash processing, since the method changes when either type of data changes.

```ruby
def random_item(data)
  if data.is_a?(Array)
    data[random_index(data)]
  else 
    jobs = data.map {|_, values| values}.flatten
    jobs[random_index(jobs)]
  end
end
```

Let’s refresh our understanding of the original definition of DRY.

**DRY Principle: Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.**

Rather than seek to remove code that has similar structures, we should focus on each idea existing once. In our example random jobs is one idea and random names is another. Code for different ideas changes at different times, so it’s best to keep them separate.

Here are some tips to see if duplication is representing the same idea:
- Ask your Product Manager (or Stakeholder) would they expect the behaviors to affect one another
- Try to reason about the ideas separately. Would they change at the same time? If they would, they are duplicates

Lastly, if you are unsure how to remove the duplication, its okay to leave it. [[Duplication is cheaper than the wrong abstraction — Sandi Metz](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction)

# TLDR;
The DRY Principle is misunderstood. This misunderstanding leads to invalid abstractions, which change for many reasons, causing bugs in unrelated parts of the system.

**If you enjoyed this post, please share it on your social media. Thanks!**

*Photo by Oliver Hale on Unsplash*