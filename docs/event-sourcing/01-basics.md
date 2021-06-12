---
title: Basics
---

Event sourcing is a pattern that perceives entity as a series of 
mutations over initial (empty) state, in contrast to classic 
approach that stores complete entities in their actual state. The most
common benefit from event sourcing is automatic history of changes, 
while storing entity as a whole requires manual work and don't tell how 
exactly entity traversed from state A to state B.

Classic approach is to simply store entity in its latest representation.
Persistence layer operates over instances of entity type, requiring 
passing fully initialized entity for inserting and updating. This way
requires client code to retrieve entity in its latest state, perform
some mutations and return it back to persistence layer in complete form,
i.e. not an "list of new fields values: a -> b, c -> d" but an object
with all properties set to corresponding values. There is nothing wrong
with this approach, it is explained for clarity.

Event sourcing, in contrast, works with mutations of entity state. The
most common bank account can be represented via its history of changes:

- Initial state with balance = 0, hold = 0 and state = open
- ATM cash income: $100
- Hold: $20
- Cancel hold: $20
- Freeze account

Such history allows recovering entity state after arbitrary number of 
changes. Applying "ATM cash income" over initial state will update
entity, setting balance to 100. Such mutations are easily represented
using in-language structures:

```
struct ATMBalanceOperation {
  decimal change;
}

struct Hold {
  decimal amount;
}

struct CancelHold {
  decimal amount;
}

struct Freeze {}

history = [
  new ATMBalanceOperation(100),
  new Hold(20),
  new CancelHold(20),
  new Freeze()
]
```

Just as an entity as a whole in classic approach, these structures could
be persisted in storage and recovered later:

```
INSERT INTO events VALUES (account = 1, index = 0, mutation = {type: ATMBalanceOperation, properties = {amount: 100}};
INSERT INTO events VALUES (account = 1, index = 1, mutation = {type: Hold, properties = {amount: 20}};
INSERT INTO events VALUES (account = 1, index = 2, mutation = {type: CancelHold, properties = {amount: 20}};
INSERT INTO events VALUES (account = 1, index = 3, mutation = {type: Freeze, properties = {}};
```

Once mutations are fetched from storage, they can be applied over 
initial entity state to get required version. Applying all mutations
will result in most recent version, but applying events 0..N where 
N < total events will restore entity state in past. This is one of major 
benefits of event sourcing, since it allows inspecting historical state
without any additional infrastructure and also allows to inspect which
mutations lead to such state. Using classical approach and storing 
entity as a whole would not tell _how_ account balance increased from N 
to M, while it could be refund, transfer or manual top up. Event 
sourcing explicitly tells what happened via mutation type removing 
possible ambiguity.

Event sourcing also implies immutability of stored records. Once created
and persisted, any record preserves it's state until purged (due to
complete parent entity removal or post-retention-interval removal). In
case any event has to be reversed, so-called retroactive event should be
issued instead of altering persisted - and this actually follows real
world processes, if you've been delivered other sneakers than you've 
ordered, then someone will come to pick them up and issue a refund, not
to wipe your memory so you'd forget the chain of events. There would be
one debatable exception mentioned in next sections, the so-called
upgrade read repair which converts from legacy format into new one and
overwrites the record with the same effect represented in a different 
structure.

While term used above is _mutation_, event sourcing name comes from
storing stream of events that happened over entity. Those two terms are
very close, but what's actually stored is what has changed in entity,
not what happened.
