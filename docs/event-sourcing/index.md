---
title: Event Sourcing Framework
---

This RFC is dedicated to event sourcing pattern and its caveats. While
pattern itself is simple and quite easy to implement, it is expected 
that some hard-to-fix issues will arise during prolonged usage.

This RFC starts with basic explanation and expands concepts and patterns
article-over-article.

Necessary remark to be made is that while event sourcing usually goes 
hand-in-hand with CQRS, those two concepts can be used and should be 
inspected separately oof each other.

1. [Understanding](understanding) - basic concepts behind event sourcing
paradigm and this framework in particular.
2. [Using](using) - How to use this framework.
3. [Fighting](fighting) - What to do with edge cases.
4. [Extending](extending) - How to write your own adapters.
