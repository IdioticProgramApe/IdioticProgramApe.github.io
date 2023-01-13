---
layout: default
title: Factory Method
parent: Creational Patterns
grand_parent: Design Patterns
nav_order: 1
permalink: /docs/design-patterns/creational-patterns/factory-method/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Pattern Structure

![FactoryMethodStructure](./assets/factory_method_structure.png)

This structure can be divided into 2 parts in a simple way, product part and creator part:

- **Product**: this part of the design take over the actual work, normally, in order to make it polymorphic, it requires:
  - Interface: defines all the virtual methods that its each specialized instance needs to have
  - Instance: carries on the specific work/logic, this is where we implement our code for a certain scenario
- **Creator**: this part of the design usually exposed to the end users, let them to create a specific product to handle their needs:
  - Interface: same as before
  - Instance: needs to create a new instance of a specific product