---
title: "When Should You Refactor?"
featured_image: "images/when_should_you_refactor.jpeg"
images: ["images/when_should_you_refactor.jpeg"]
date: 2018-07-09T20:32:25-04:00
---

I find that there are two moments that best suit using refactoring; practicing TDD and adding new features.

As part of the TDD flow, we restructure the code we wrote in the last ten minutes, after the red and green stages. We iterate until we find a design that is the best we can think of at the moment, for the features we have now. The best designs follow the [Four Simple Rules Of Design](https://www.martinfowler.com/bliki/BeckDesignRules.html). We don’t concern ourselves with future features and guesses at optimizations as [we aren’t gonna need it](http://wiki.c2.com/?YouArentGonnaNeedIt).

When adding a new feature, we use refactoring differently. We use it to make the feature easy to add. Sometimes this means moving multiple pieces of code closer, to prevent shotgun surgery. In other cases, it is creating a layer of indirection, so an new algorithm can be added. This follows the mantra; [make the change easy, make then easy change](https://www.facebook.com/notes/kent-beck/runright-and-vice-versa/566483323384536).

# Permission Not Needed

In all cases refactoring is part of the user story, not a separate thing that needs to be advocated for by the engineering team. Engineers should be empowered to refactor as they need in order to deliver value. However, if they don’t need to change code to deliver the new feature, then refactoring is just over processing (adding more quality than is valuable). While delivering software in an agile fashion it is important to build just enough code with just enough quality that users need. Attempting to guess about the future features will yield unnecessary complexity.

# An Exception

There is one other scenario where refactoring is beneficial, it is when we refactor multiple parts of a system to be more uniform in structure. Not having shared patterns across multiple sections of the code creates cognitive overhead for engineers, reducing the overall readability. This refactoring is different than the other two types, it should require engineers and product managers to agree on time being spent. Engineers must make a sound case for why this change should happen now, and product managers must make sure that the value stream isn’t being interrupted too much. I am not a big fan of this third refactoring use case, it feels YAGNI but it serves another important purpose. Engineers need to feel like they own the code, [the best way to own something is to have the autonomy to change it](https://www.researchgate.net/publication/301612260_Practice_and_Perception_of_Team_Code_Ownership).

# Conclusion

There are two types of refactoring that are part of user stories and one that isn’t. Refactoring as part of the TDD flow and when adding new features, is part of a user story. Whereas, refactoring to bring structural uniformity across the system, must be done with agreement between engineers and product managers. I recommend leaning on the types that are part of user stories and doing [technical retrospectives](/posts/technical-retrospective-what-why-how/) to incrementally bring structural uniformity.

**If you enjoyed this post, please share it on your social media. Thanks!**