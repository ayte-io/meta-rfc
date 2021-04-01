---
title: Isolation
---

One of pain points in every software maintenance process is dependency
conflicts. Configuration management is not an exception, and it's quite
often to enter a stalemate of outdated dependencies without an option of
forceful resolution, even if no backward-incompatible features are used.

This can be addressed with a very simple solution, which is separating
complete flow into units and forcing every unit to run in a separate 
process.

CM may even enforce every step to be ran in a separate process, 
providing absolute isolation, preparing new processes in advance to keep
time overhead as low as possible.
