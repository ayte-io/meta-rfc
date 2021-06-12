---
title: Schema Evolution
---

As been stated before, event sourcing concept implies immutability of
persisted records. This introduces another problem: it is likely hat 
over time mutation structures will change significantly in a 
backward-incompatible way. A synthetic example is a refactoring where
team tries to get rid of old ugly enum like `{ST_OPEN, ST_DLVRD}` and
start using more meaningful and self-explanatory names. The problem is,
storage is full of those old values that are there forever. Developers 
are then facing the choice of either continue using

TODO
