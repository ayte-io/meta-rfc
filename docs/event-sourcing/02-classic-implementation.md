---
title: Classic Implementation
---

Simplest event sourcing implementations just drops entity reference,
mutation sequential number, its type and properties into storage:

| Entity                | Mutation Index | Type                        | Properties   |
|:---------------------:|:--------------:|:---------------------------:|:------------:|
| org.company.Account:1 | 0              | org.company.RefundEvent     | {amount: 10} |
| org.company.Account:1 | 1              | org.company.WithdrawEvent   | {amount: 20} |
| org.company.Account:2 | 0              | org.company.PromoBonusEvent | {amount: 50} |
| org.company.Account:1 | 2              | org.company.FreezeEvent     | {amount: 20} |

This is enough to restore entity history, but there's a caveat that
amount of records and fetched data grows proportional to history length.
To address this, concept of snapshots was introduced - if entity is
serialized and stored after mutation 50, then all mutations up to 50 are
not necessary to reapply to restore entity version 51 and later:

| Entity                | Mutation Index | Properties     |
|:---------------------:|:--------------:|:---------------|
| org.company.Account:1 | 50             | {balance: 200} |
| org.company.Account:2 | 50             | {balance: -30} |
| org.company.Account:1 | 100            | {balance: 350} |

Now to restore account with id 1 at version 57 it is enough to restore
snapshot at version 50 and apply mutations 51-57 without fetching first
50.

Such framework requires only two collections in storage and could be 
implemented in matter of hours. However, this implementation has 
numerous drawbacks that would be mentioned along with required 
countermeasures in following sections.
