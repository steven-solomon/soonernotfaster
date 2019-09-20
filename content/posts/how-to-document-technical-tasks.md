---
title: "How to Document Technical Tasks"
featured_image: "/images/technical_tasks.jpeg"
images: ["/images/technical_tasks.jpeg"]
date: 2019-05-10T14:46:55-04:00
---

I want to briefly speak to you about the strategies that I use when writing technical documentation on my teams.

You may be wondering how you can write a document about the steps to deploy your application to your staging environment. In this case, I feel you should favor automation over writing documentation — as it is valuable to deploy an application or services in an automated fashion at a regular cadence.

If you are looking for help with documenting environment setup, or performing tasks that don’t happen frequently, then documents are a better choice. Eventually it makes sense to invest in automation if a task happens frequently enough. The mantra of “once, twice, automate” is a good rule of thumb.

When generating a document, I like to do it in iterations. I will step through some process, and write down a high level description of what I am trying to accomplish. Then I follow it with details about how to do that step. I prefer to link out to the vendor documentation whenever possible — my goal is not be to write documentation for technology that I don’t own, unless it is poorly written.

Refining the document comes with each subsequent usage. Next time I do the task described in the document, I make sure to only follow the steps in the document, this will make sure to expose holes in what was written, as well any hidden assumptions.

When possible I prefer to have someone who didn’t participate in writing the document follow it’s step. Fresh eyes allow for maximizing mutual benefit; the reader learns the new technique, and their questions highlight potential improvements in the document.

Lastly, I make sure to delete sections or documents that have outlived their usefulness. Documents are great, but they come with a maintenance cost. If they are out of date or never used, they only serve to hurt the team. They can slow you down by giving them false information, or causing them to waste precious execution time updating a document that no one will read.

As with all things there are trade-offs. It is up to you to appropriately apply these ideas in your context.

**TL;DR**

- For deployments: prefer automation to documentation
- Write a document in iterations, then follow mantra: “once, twice, automate”
- Don’t maintain documents you don’t need

**If you enjoyed this post, please share it on your social media. Thanks!**