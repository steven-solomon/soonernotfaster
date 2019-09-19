---
title: "Don't Repeat Yourself Is Misunderstood"
featured_image: "images/dont_repeat_yourself_is_misunderstood.jpeg"
date: 2018-05-06T08:23:47-04:00
---

In the words of Jimmy Two Times, "I'm gonna go get the papers, get the papers."

The Don’t Repeat Yourself (DRY) Principle is commonly interpreted to mean, code with no duplicate statements. I’d like to explore that thinking with you for a few minutes so that I can demonstrate how it is flawed.

I have written a small program that displays the name and job of randomly created people. Take a look at the :random_person function. At this moment it actually does two things; generates a random name and a random job ([Functions should do one thing](https://sites.google.com/site/unclebobconsultingllc/one-thing-extract-till-you-drop)) but it sets up the scenario I’d like to show you. Notice that both name and job are being looked up using the result of rand(4) as an index.

```ruby
class Person
  def initialize(name, job)
    @name = name
    @job = job
  end

  def to_s
    "Person named #{@name}, works as #{@job}"
  end
end

def random_person
  names = %w(Mike Zoe Adam Sarah)
  jobs = %w(Astronaut Engineer Painter Day-Trader)
  Person.new(names[rand(4)], jobs[rand(4)])
end

def main
  10
    .times
    .collect { random_person }
    .each { |p| puts p }
end

main
```

This fits our earlier definition of duplication, so let’s refactor. We use the [extract method refactoring](https://sourcemaking.com/refactoring/extract-method) to create a :random_index function.

```ruby
def random_index
  rand(4)
end

def random_person
  names = %w(Mike Zoe Adam Sarah)
  jobs = %w(Astronaut Engineer Painter Day-Trader)
  Person.new(names[random_index], jobs[random_index])
end
```

Hmm. Seems like there is also some duplication around looking up something from an Array. Let’s extract that too. Again, using the extract method refactoring to create :random_item.

```ruby
def random_item(list)
  list[random_index]
end

def random_person
  names = %w(Mike Zoe Adam Sarah)
  jobs = %w(Astronaut Engineer Painter Day-Trader)
  Person.new(random_item(names), random_item(jobs))
end
```

Now imagine we have to add a new job type, Theoretical-Physicist.

```ruby
def random_person
  names = %w(Mike Zoe Adam Sarah)
  jobs = %w(Astronaut Engineer Painter Day-Trader Theoretical-Physicist)
  Person.new(random_item(names), random_item(jobs))
end
```

All done right? Wrong, :random_index is coupled to the size of the Array. It assumes there are only 4 elements. So we will never see a theoretical physicist show up in a randomly created Person. So let’s change that function to instead use the size of the Array. Here we have to do two refactorings. First, we use the [add parameter refactoring](https://sourcemaking.com/refactoring/add-parameter) to pass the list to :random_index.

```ruby
def random_index(list)
  rand(4)
end

def random_item(list)
  list[random_index(list)]
end
```

Add Parameter Refactoring
Next, we change the implementation to use the size of the Array instead of the hard coded 4.

```ruby
def random_index(list)
  rand(list.size)
end

def random_item(list)
  list[random_index(list)]
end
```

Notice we changed the interface and the implementation at different times. This makes our lives easier.
Once again, our program is outputting random people, some of whom can now be a theoretical physicist. Now imagine the case where the data-structure for jobs has changed. It is now a Hash of sectors, each of which has an Array of jobs.

```ruby
def random_person
  names = %w(Mike Zoe Adam Sarah)
  jobs = {
    stem: %w(Astronaut Engineer Theoretical-Physicist),
    finance: %w(Day-Trader),
    arts: %w(Painter)
  }
  Person.new(random_item(names), random_item(jobs))
end
```

Now our :random_item function doesn’t work. It is coupled to the fact that jobs was an Array. We can put in a check for the type of data-structure, then map the Hash entries to their values. Giving us an Array of Arrays contain jobs. If we flatten that Array, we are left with all jobs.

```ruby
def random_item(list)
  if list.is_a?(Array)
    list[random_index(list)]
  else
    # Starting to get a bit hacky but okay
    jobs = list.map {|_, values| values}.flatten
    jobs[random_index(jobs)]
  end
end
```

Now our users want more random fields. This time they want a random pet. Pets is an Array containing Hashes, with type and name for each animal.

```ruby
pets = [
  { type: 'Dog', name: 'Fido' },
  { type: 'Cat', name: 'Whiskers' },
  { type: 'Horse', name: 'Thunder' }
]
```

We don’t want to display the name of the animal in each Person, just it’s type. So a random person could be Mike the Astronaut, who has a Cat. So we update Person and add a pet as an instance variable.

```ruby
class Person
  def initialize(name, job, pet)
    @name = name
    @job = job
    @pet = pet
  end

  def to_s
    "Person named #{@name}, works as #{@job}. They have a pet #{@pet}."
  end
end
# ...
```

Next we pass :random_item the pets data.

```ruby
def random_person
  # ...
  Person.new(random_item(names), random_item(jobs), random_item(pets))
end
```

Now let’s tweak that Array case to do a bit of type checking on the elements of the list…and Scene.

We were about to change how random names work to hack in the ability for random pets. We were changing unrelated parts of the system to add new behavior. Now that we have seen from experience that the previously stated interpretation of the DRY Principle can lead us to some really nasty code. I want to restate the original definition of the [DRY Principle](http://wiki.c2.com/?DontRepeatYourself).

**Dry Principle: Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.**

According to that definition, our original refactoring didn’t actually follow the DRY Principle at all. Instead of making sure each statement in our code exists once, we should make sure each idea exists once.

So how can we apply the DRY Principle as intended? We ask ourselves, [“Can these two pieces of code be reasoned about separately?”](https://m.facebook.com/notes/kent-beck/bits-clumps-and-just-right/792597974106402) Let’s try that with this situation. Can a random name be reasoned about separately from a random pet? Yes. Since they can, let’s not bind them together, as they will most likely change for different reasons. A block of code that changes for multiple reasons is giving off the code smell of [divergent change](https://refactoring.guru/smells/divergent-change), as it doesn’t follow the [Single Responsibility Principle](http://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html). If we put code that changes for different reasons together, bugs might be introduced in seemingly unrelated parts of the code. These bugs will surprise everyone; our team, our PM and even future us. In a real system, the cost of this type of mistake is much greater and much harder to recover from.

# Conclusion

Now we know from experience that the DRY Principle is actually about removing duplicate ideas and not duplicate code. We should remind each other of the following quote the next time we are about to refactor.

[Duplication is cheaper than the wrong abstraction — Sandi Metz](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction)

**If you enjoyed this post, please share it on your social media. Thanks!**
