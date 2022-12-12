---
layout: default
title: EuroPython
parent: Conference
grand_parent: Python
nav_order: 1
permalink: /docs/python/meta-programming/euro-python/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Clean Architectures in Python

| Lecturer        | Leonardo Giordani                                            |
| --------------- | ------------------------------------------------------------ |
| Time            | 2022/07/15                                                   |
| Link            | [https://www.youtube.com/watch?v=C7MRkqP5NRI](https://www.youtube.com/watch?v=C7MRkqP5NRI) |
| Takeaways       | - OOP is supposed to be objects exchanging the messages, it's distributed system<br/>- Clean should be for each component, we know **where** it is, **what** it is and **why** it is in the system<br/>- A component can see only what have been defined in the same/inner layers, but not allowed to use anything defined in the outer layers. ref: [clean architecture]([A quick introduction to clean architecture (freecodecamp.org)](https://www.freecodecamp.org/news/a-quick-introduction-to-clean-architecture-990c014448d2/)) model predates Robert Martin<br/>- The golden rule: Talk inward with **simple structures**, talk outwards through **interfaces**<br/>- Dependency injection, use interfaces to reduce coupling<br/>- A clean architecture can be **tested** very well, for example to test all the use cases in a isolated environment (more coupled system, more difficult to write tests)<br/>- Tests should cover all the layers in the architecture to make sure everything functions as expected<br/>- Migrations happen one step at a time, isolate a part of code, refactor it and so on<br/>- There is no definitive clean architecture, always a balance between generality and speciality |
| Recommendations | - (Book) **Design Patterns** by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides<br/>- (Book) **Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions** by Gregor Hohpe, Bobby Woolf<br/>- (Book) **Clean Architectures in Python** by Leonardo Giordani |

