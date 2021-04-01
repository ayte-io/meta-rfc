---
title: Ayte : Event Sourcing : Understanding : Introduction
---

This article is created to explain whole event sourcing concept, which 
is not limited to this particular framework.

Bird-view takes just a single paragraph. Idea of event sourcing is just 
to capture events that mutate entity state (which will be further called 
aggregate root): that is, if somebody top-ups theirs account balance,
persist "Top-up: $100" event, and that's it. It is obvious that such 
approach makes persisting and reading entity history tremendously easy
(you automatically get a sequential stream of events that have happened) 
but hey, it's the smallest of all benefits - we've just started.

Event sourcing implies not only capturing those events, but persisting
and recovering actual entity state using them. Imagine you have
following history:

- Account created
- Registration bonus: $10
- DLC bought: $7
- Account top-up: $25

You can easily recover entity state using this history just applying
events one-by-one to initial entity state (null):

- null -> Account (balance=0)
- Account(balance=0) -> Account(balance=10)
- Account(balance=10) -> Account(balance=3)
- Account(balance=3) -> Account(balance=28)

Moreover, you are able not only restore current entity state - you are
able to restore entity state at *any* given point in time.

So when using event sourcing, you don't need traditional "persist state
as-is" approach at all (some restrictions apply, though).

Other nice benefits to mention is that you have out-of-the box update 
atomicity and that storage requirements are so primitive that event 
sourcing can be built on top of nearly every storage, be it SQL, NoSQL,
or even filesystem, if you're feeling eccentric.

### Ok, that's pretty simple, but how will my code look like?

The events themselves are usually represented by classes:

```java
class AccountCreated {}

@Data
class AccountBonusIssued {
    @NonNull
    private final BigDecimal amount;
}

@Data
class OrderPlaced {
    @NonNull
    private final Map<UUID, Integer> items;
    @NonNull
    private final BigDecimal cost;
}

@Data
class AccountTopUpped {
    @NonNull
    private final BigDecimal amount;
}
```

Those events are applied to entity. Traditional approach is exposing
methods on aggregate root that accept event classes, which are
completely anemic:

```
class Account {
    public void accept(AccountCreated event) {}
    public void accept(AccountBonusIssued event) {}
    public void accept(OrderPlaced event) {}
    public void accept(AccountTopUpped event) {}
}
```

**Please note that this framework uses it's own approach.** This article 
is about concept as it usually described on the internets, not about 
particular implementation, so I'm describing commonly accepted style
(I guess lots of folks haven't made a solid decision about using this
particular framework at this moment, so it's better to introduce them
what's usually happening rather what is happening in particular engine).
You can check this framework's and see motivation behind taking 
non-industry-standard way in [Approach](approach) article.

Persistence and recovery are damn simple:

```java
class Storage {
    public boolean persist(String entityId, Object event) {
        long nextEventIndex = getLatestEventIndex(entityId);
        Wrapper wrapper = new Wrapper(event, event.getClass());
        return keyValueStorage.getCollection(entityId).insert(nextEventIndex, serialize(wrapper));
    }

    public List<Object> getEvents(String entityId) {
        List<Wrapper> wrappers = keyValueStorage.getCollection(entityId).getAll();
        return wrappers.stream()
            .map(wrapper -> wrapper.deserialize(wrapper.getEvent(), wrapper.getEventClass()))
            .collect(Collectors.toList());
    }
}

class AccountController() {
    public Account get(String id) {
        return storage.getEvents(entityId).stream()
            .reduce(new Account(), (account, event) -> account.accept(event));
    }

    public boolean topUp(String id, BigDecimal amount) {
        for (int i = 0; i < 8; i++) {
            if (storage.persist(entityId, new AccountTopUpped(amount))) {
                return true;
            }
        } 
        // much contention, very concurrent access
        return false;
    }
}
```

Code is very simplified for ease of grasping the concepts, but it 
doesn't somewhat harder to understand in production.  

### How does it look from storage side?

Implementation of basic event sourcing storage is as simple as code.
Every event is stored as a record with following attributes:

- Entity type (e.g. `account`)
- Entity id (e.g. some random uuid)
- Sequential event number (1, 2, 3 and so on)
- Event type (e.g. `AccountCreated`)
- Event payload (usually just a json blob)

Primary key consists of entity type, entity id and sequential event 
number. In case of partitioned storage partition key is entity type and
entity id, while event number is a clustering key. Whenever new event
is about to be added, first free sequential event number is computed,
and client code tries to add entity with such id.

### Restrictions

Event sourcing works very well when application scope is limited by a
single entity. Problems arise when

- Entity listing is involved (e.g. standard pagination of existing 
entities). Or, even worse, entity search.
- Several entities require transactional processing (i.e. all-or-nothing
operation execution).
- Some other cases I can't come up with at this very moment.

Those are not unbeatable, however, solving those issues is trickier in
order of magnitude. First one is solved through [Projections](projections),
the only way to address second one I'm aware about is described in
[Transactional Processing](../fighting/transactional-processing). 
However, those are only examples that come up first, the idea behind
this section is just get you prepared that classic operations are not so
simple anymore.

TBD: nothing can be changed after it's in database

### Conclusion

If something is still unclear for you, that's okay - I've never been 
good at explanations. You can check some classic resources which may be 
more understandable for you:

- [Martin Fowler's article](https://martinfowler.com/eaaDev/EventSourcing.html)
- [GitHub user ookami86 presentation](https://ookami86.github.io/event-sourcing-in-practice/)

And, obviously, there's [Google feed](http://google.com/search?q=event+sourcing)
that accumulates new articles every once in a while.

### Related

#### Snapshots

#### CQRS (not covered by this framework)

#### Projections (not covered by this framework)
