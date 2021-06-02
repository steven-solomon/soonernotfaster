---
title: "How to Stop Past Coworkers From Calling You"
featured_image: "images/how_to_stop_past_coworkers_from_calling_you.jpeg"
images: ["images/how_to_stop_past_coworkers_from_calling_you.jpeg"]
description: “I Have To Tell You About The Future”
date: 2018-07-05T20:15:32-04:00
tags: ['design', 'frontend']
---

Are you currently the only one on your engineering team who can read and understand your code? If you left for a new job, how often would your teammates need to call you in order to maintain the code you left them? Let’s talk about how to not get calls after you leave a job.

# Visual Hierarchies

Visual hierarchies are a powerful tool. As readers they allow us to scan, giving us a sense of the main flow. A high level flow helps us ignore information that we don’t care about. Diving into a chapter, section and subsection allows us to get closer and closer to what we are looking. Since, software spends most of its life being read, we should optimize our code’s readability.

# Relaying Out The Code

Take a look at this code, it is long and full of detail. What are the import parts? What can you ignore? We can’t tell, rather we must spend time reading and understanding all the code in order to find out what is happening.

```
const fs = require('fs')

fs.readFileSync('customers.csv', 'utf8')
    .split('\n')
    .map(line => line.split(','))
    .map(parts => ({
        name: parts[0],
        numPurchases: parts[1]
    }))
    .filter(customer => customer.numPurchases > 2)
    .map(customer => customer.name)
    .forEach(x => console.log('each:', x))
```

Compare that with the following code. Where I have extracted each behavior into a section (function) and gave it a clear name. Notice how you can ignore the details of how behaviors are preformed. You can just skim the bolded text just like an article, getting a sense of what is happening. If you really want more detail, you can read the function that interests you.

```
const fs = require('fs')

function customersList() {
    return fs.readFileSync('customers.csv', 'utf8')
        .split('\n')
        .map(line => line.split(','))
        .map(parts => ({
            name: parts[0],
            numPurchases: parts[1]
    }))
}

function namesWithMoreThanTwoPurchases(customers) {
    return customers.filter(customer => customer.numPurchases > 2)
            .map(customer => customer.name)
}

function printAll(names) {
    names.forEach(n => console.log('each:', n))
}

// high level policy is clearer
printAll(
    namesWithMoreThanTwoPurchases(
        customersList()
    )
)

```

Sure, it is not as pretty as chained functions but it does something more important, makes what the code is doing clear.

# Conclusion

There has a been a big push lately to use function chaining in order to eliminate imperative code. While I believe that this is mostly a good thing, I have seen engineers become tempted to collapse large swaths of a system into a giant method chain. While it is true that you can do this, it is probably not a good idea. Fewer lines of code doesn’t mean the code is easier to understand. Your coworkers probably won’t be able to read it a few months from today and maybe you won’t be able to either. Readability for our team should be the most important design constraint (in most cases).

{{< thank-you >}}

*Code example taken from a great Javascript Youtube channel, [Fun Fun Function](https://www.youtube.com/watch?v=UD2dZw9iHCc&list=PL0zVEGEvSaeEd9hlmCXrk5yUyqUag-n84&index=11)*

*Clean Code by Robert Martin, Chapter 3, Functions*

*Implementation Patterns by Kent Beck, Chapter 7, page 64, Main Flow Pattern*