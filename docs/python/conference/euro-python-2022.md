---
layout: default
title: EuroPython 2022
parent: Conference
grand_parent: Python
nav_order: 1
permalink: /docs/python/meta-programming/euro-python-2022/
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

## Demystifying Python’s Internals

| Lecturer        | Sebastiaan Zeeff                                             |
| --------------- | ------------------------------------------------------------ |
| Time            | 2022/07/15                                                   |
| Link            | [https://www.youtube.com/watch?v=f8nTJp_k7U8](https://www.youtube.com/watch?v=f8nTJp_k7U8) |
| Takeaways       | - Convert the source code into a Magic, it will go from Source code to **Abstract Syntax Tree** as the first step and from from AST to Magic (Execution) as second step<br/>- Convert from raw text to a stream of tokens, some value tokens, name tokens and some symbol tokens. ref: Grammar/Tokens<br/>- Use **Parsing Expression Grammar** (PEG) syntax to add a grammar rule, ref: Grammar/python.gram<br/>- Add new AST Nodes to convert from the grammar rule to an AST, ref: Parser/Python.asdl<br/>- Each unique instruction has its own "opcode" (operation code), ref: Lib/opcode.py<br/>- Make the compile use the new "opcode", ref: Python/compile.c, `compiler_visit_expr`, `stack_effect`(for value stack)<br/>- The evaluation loop in python is quite massive, ref: Python/ceval.c |
| Recommendations | - (Link) [https://github.com/SebastiaanZ/pypethon](https://github.com/SebastiaanZ/pypethon)<br/>- (Book) **CPython Internals** by Anthony Shaw & *Real Python* |

## Write Docs Devs Love: Ten Tricks To Level Up Your Tech Writing

| Lecturer        | Mason Egger                                                  |
| --------------- | ------------------------------------------------------------ |
| Time            | 2022/07/15                                                   |
| Link            | [https://www.youtube.com/watch?v=EUhilSOppc0](https://www.youtube.com/watch?v=EUhilSOppc0) |
| Takeaways       | - Practice!<br/>- Verify the instructions and test them with a fresh environment<br/>- Make your content scannable/navigable/consistent, readers can easily find a single piece of information<br/>- Don't make your reader leave your docs, avoid too many external sites/links<br/>- Use meaningful code samples and variable names, it sometimes can save a lot of writing work (code should be self documenting as well)<br/>- Avoid memes/idioms and regional languages, or use it based on your audience<br/>- Define ALL acronyms, write out the full name of the acronym when first introduced, can even create a lookup table<br/>- Limit technical jargon, use it based on your audience<br/>- Use inclusive language, use more gender neutral pronouns and avoid internet slangs, subjective words<br/>- Don't be overly verbose, always assume the readers don't speak the same language as you<br/>- Make the end goal clear |
| Recommendations | - Write documentation at work<br/>- Start a blog<br/>- Contribute to Open Source Projects |

## `typing.Protocol`: type hints as Guido intended

| Lecturer        | Luciano Ramalho                                              |
| --------------- | ------------------------------------------------------------ |
| Time            | 2022-07-15                                                   |
| Link            | [https://www.youtube.com/watch?v=0_IQoxBFepw](https://www.youtube.com/watch?v=0_IQoxBFepw) |
| Takeaways       | - Type is a set of values and a set of functions that one can apply to these values (PEP 483 - The Theory of Type Hints)<br/>- Types are defined by *interfaces* and ***Protocol*** is a synonym for *interface*<br/>- Using `typing.Protocol` can preserve the flexibility of duck typing, support static analysis and reduce the code coupling, make tests easier (mock tests)<br/>- Python is always a dynamic typing language even we provide the type hint prior to the runtime<br/>- **Duck typing** is about *structural types* get checked during runtime, supported in Python<br/>- **Static typing** is about *nominal types* get checked statically, supported in Python 3.5 or later <br/>- **Goose typing** is about *nominal types* get checked during runtime, as `collections.abc` in Python 2.6 or later<br/>- **Static duck typing** is about *structural types* get checked statically, as `typing.Protocol` in Python 3.8 or later<br/>- Follow the Interface Segregation Principle, client code should not be forced to depend on methods it does not use<br/>- The narrow protocols are preferred, single-method protocols are the most common ones, 2-methods protocols are lot less<br/>- Being optional on type hint in Python is the feature that gives programmers the power to cope with the inderent complexities, anooyances, flaws, and limitations of static types<br/>- Usually, 95% test coverage is pretty good |
| Recommendations | - (Link) Some built-in typing protocols: [https://docs.python.org/3/library/typing.html?highlight=typing#protocols](https://docs.python.org/3/library/typing.html?highlight=typing#protocols)<br/>- (Link) [Abstract Base Classes and Protocols](https://jellis18.github.io/post/2022-01-11-abc-vs-protocol/)<br/>- Use `typing.TypeVar` to create a new type and bound it with a class which has the desired methods/variables |

> Guido Van Rossum is the father of Python programming language, ref: [Guido van Rossum - Wikipedia](https://en.wikipedia.org/wiki/Guido_van_Rossum)

