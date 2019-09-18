---
title: "Patterns of Sharing Work Between Teams"
featured_image: "images/patterns_of_sharing_work.jpeg"
date: 2018-08-01T07:59:41-04:00
---

While I was researching different ways for teams to increase their throughput, I stumbled upon a distinction that is often not made. Framework is often used to refer to both the framework pattern and subsystem pattern. Here is a short description of those two patterns and their tradeoffs.

# Frameworks

Frameworks are skeletal structures of programs that must be fleshed out to build a complete application — Rebecca Wirfs-Brock

## Examples

Spring and Rails are examples of frameworks. However, your business may have a series of very similar applications that may fit into a framework.

## Motivation

Whenever there is a cross cutting concerns that can be applied to many applications, create a framework. You may have a number of applications that all follow a POS flow but the products sold, hardware and payment methods accepted vary. A framework can encapsulate the overall flow and plugins can easily allow for the system to add the specific behavior for the variations.

## Risks

Frameworks are not the right pattern if all the applications don’t need the overarching flow. For example a framework for all games probably doesn’t have much use.

# Subsystems

A subsystem is a set of classes (and possibly other subsystems) collaborating to fulfill a set of responsibilities. Although subsystems do not exist as the software executes, they are useful conceptual entities. — Rebecca Wirfs-Brock

## Examples

Subsystems are about an isolated and specific behavior. They are tools that can be used to accomplish a finite task. Examples are Moment JS or Hibernate.

## Motivation

It may be desirable to share common functionality between teams or as an open source system. The shared behavior is discrete.

## Risks

Maintaining the project can become a full time activity.

It can be difficult to define a well formed interface to your subsystem.

Coordinating between teams can become expensive. With each new client, it becomes harder to change the subsystem.

Teams can become blocked until the new features are delivered.

# Conclusion

Sharing work can bog down teams. Frameworks and subsystems are two patterns of sharing work but they should be used with caution. Sometimes it is faster if each team builds smaller versions of frameworks or subsystems.

**If you enjoyed this post, please share it on your social media. Thanks!**

*Photo by Elaine Casap on Unsplash*