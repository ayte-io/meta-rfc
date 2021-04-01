---
title: Ayte : Event Sourcing : Understanding : Framework Requirements
---

[Introduction](./introduction) should have given a brief overview of
what event sourcing is and what is the basic minimum of code / storage
requirements needed. However, it's not enough for production systems,
and here are few things that related frameworks try to solve.

### Snapshots

This is quite obvious and usually implemented in all frameworks, but 
just nobody forgets that: there is no event sourcing without 
snapshotting. Snapshots are just entity states taken at event N. Given
request for entity state at `N + X` framework shouldn't read all events
from `0` to `N + X`, instead it should recover entity from snapshot `N`
and read and apply only events from `N` to `N + X`. 

### Listeners

This is usual requirement as well: framework should provide interface
to plug in user-defined listeners, so their application could react on
specific events, be it simple read, event insertion, snapshot creation,
or something else.

### Identifiers of different types

First of all, entities identifier type should not be imposed by 
framework. If you look at other existing solutions, they are often bound
to specific ID type, usually UUID, but it's not necessary that end user
wants to or even can use UUIDs. I bet that there are more systems that
have migrated to event sourcing rather than ones that were using it from
the start, which means that a lot of users (if not most) need to migrate
from good old auto-increment int / long identifiers, but they can't use
such framework without reimporting all data from scratch. That's not
only lots of extra work, but also very error-prone.

If you think it just ends here, then it doesn't: there are entities that 
have composite IDs. Therefore framework should never assume that ID can 
be represented as string (that would end up with complex and error-prone
escaping of concatenated string representations), instead if should 
target byte arrays and serialize IDs using user-provided serializer. 
Users, in turn, have to ensure that properties of composite key are 
serialized in consistent way, otherwise they may end up with several 
event streams instead of one. 

### Entity interfaces

**Don't** force user to implement **any** interface on their entity. If
you need specific values, like ID extraction, force user to provide
specific entity-accepting functions, but don't force user to do anything
with their entity. This always ends bad, don't obstruct user's domain.

### Event flood

Most tutorials will instruct you to keep events anemic and expose
methods on aggregate root that accept events, while most frameworks will 
*force* to do the same. At first, it seems OK:

```java
class EventA {}
class EventB {}
class EventC {}

class AggregateRoot {
    void accept(EventA event) {}
    void accept(EventB event) {}
    void accept(EventC event) {}
}
```

So far so good. But in one year any production system will end with
tens event types per entity, and one will have to support up to hundred
methods *in one class*. At that point code smells as bad as the one it 
has to migrate from, and developers are pulling their hair away.

The obvious solution is to attach actual processing to event itself,
and manage it with event class:

```java
interface Event<T> {
    T apply(T entity);
}
```

Or even simpler

```java
interface Event<T> extends UnaryOperator<T> {}
```

It also makes more sense: it's not entity mutating itself, it's external
event that forces changes in entity state.

### Difference between "event occurred at" and "event acknowledged at"

Usually it is implied that event date/time equals to time when event was 
registered in the system. However, most events originate from external
providers, be it automatic systems or user input, which means that it 
has happened earlier than it was accounted for. Because of that decent
framework should recognize two date/times - when event has happened 
(occurred at) and when it was processed by system (acknowledged at). 
While "acknowledged at" should monotonically increase, "occurred at" may
be out of order (however, acknowledged at is not guaranteed to be in 
order - if different hosts may register events, then one or more hosts
may experience time skew). Generally, both date/times should not be 
relied on for anything rather than providing debug data, because actual 
order of events processed by system is represented in event numbers, 
which are guaranteed to monotonically increase.

### Storing event date/time with zones

Given previous section, this is really important. Framework really has
to care about timezones. Users emitting events will be doing it from
different countries. Companies using framework will have servers on both
coasts, be it America or Eurasia. It's not an option to ignore timezones
or using unix timestamps, since it conceals real state of things. It's
better to serialize date as string rather cut off it's timezone using 
MySQL's datetime type.

### Event / mutation separation

Another usual thing to do is to imply that event and it's effect is the
same, while I believe they are somewhat different. Effect (or mutation)
is an actual state change, while event is actually a reference of
particular mutation that has been applied to specific entity at specific
time. Frameworks this document covers make this distinction, stating 
that event is just a wrapper that encloses entity type / id, event 
number, mutation, event date/times (occurred at / acknowledged at) and 
some metadata that application decided to attach (e.g. host that 
performed insert), and mutation is an instance of specific class that 
a) holds all the necessary context and b) can be applied over entity to 
change it's state:

```java
interface MutationContext<E, ID> {
    Class<E> getEntityClass();
    ID getEntityId();
    long getEventNumber();
    Optional<ZonedDateTime> getOccurredAt();
    ZoneDateTime getAcknowledgedAt();
    // ...
}

interface Mutation<E, ID> implements BiFunction<E, MutationContext<E, ID>, E> {}

@Data
class AccountCharged implements Mutation<Account, UUID> {
    private BigDecimal amount;

    public Account apply(Account entity, MutationContext<Account, UUID> context) {
        return entity.widthdraw(amount);
    }
}
``` 


In fact, event is a context that exists for each mutation that has taken
place, so it's semantically another entity rather than mutation.

### Mutation should not change entity

One other thing to consider is that events should produce new instance
of entity rather than modifying current one. First of all, it allows to
end user to have immutable entities, which is good by itself (and if end
user doesn't think so, nobody forces they to do so), second, it allows
to speculatively apply mutations to see what result will be *if*
mutation will be recorded. This may be particularly important for 
checking invariants before event persistence - instead of writing 
per-event validators that analyze current entity state and mutation 
context (which actually shouldn't be visible to anyone except for 
mutation itself) one can write single validator that validates only 
entity, e.g. that account's balance is more than zero.

### Actual types abstraction layer

Usually mutations are stored in database in a very simple way:

```yaml
entity: org.company.model.Account
entity_id: 1
event_id: 1
mutation_type: org.company.event.account.AccountCreated
mutation: <serialized json>
```

The moment one does that, they are seriously fucked:

- They can't move entity and mutation classes around anymore
- They can't migrate to other language, which a lot of applications
does

Because of that, another level of abstraction should be introduced: 
store not concrete classes, but semantic types instead:


```yaml
entity: account
entity_id: 1
event_id: 1
mutation_type: created
mutation: <serialized json>
```

And keep simple mapping in application layer:

```yaml
entities:
  account:
    type: org.company.model.Account
    mutations:
      created: org.company.event.account.AccountCreated
```

Moreover, nicely behaving framework should allow end user to specify 
aliases for mutations (e.g. `AccountCreated`: main id `created`, aliases
`legacyCreatedV1`, `crtd`) , so mutations previously stored under other 
name would still be readable. The same may be done for entities as well, 
but this will result in multiplying storage queries.

### Schema evolution

That's a very painful one. When application is long gone in production,
there's *always* a necessity to change one of the mutations, usually in 
a way that is not backwards compatible. There is ugly solution for 
this - AccountCreatedV2, AccountCreatedV3 - and a nice one, which is 
introducing mutation versioning from the start. Framework should be 
responsible of persisting mutation type and it's version side-by-side:

```yaml
mutation_type: created
mutation_version: 1
```

This allows more flexible approach.

The perfect framework would go even further, offering mutation update
capability. If mutation implements specific interface, e.g.

```java
interface ObsoleteMutation<E, ID> {
    Mutation<E, ID> upgrade();
}
```

Then framework should a) convert such mutations during read and b) if
possible, perform a replace of obsolete event with a newer version (if
backing storage supports such operation). And yes, in fact event stream
is not so immutable - there is this case, retention and compaction, to
name a few.

### Read repair

This one is not as important, but should be included in ideal framework
as well. Given that application may crash right after event insertion 
but before snapshot creation, framework should attempt to create 
missing snapshots during reads with configured probability.

Framework *may* also call application-defined hooks on entity reads (so
application would check if projection is in consistent state), but this
point is a bit slippy: if event insertions happen too often, projection
update for event E1 may be executed after projection update for event 
E2. This, however, is possible during standard flow as well.

### Retention

Any event sourcing article on the web will tell you that events, once 
stored, are immutable and untouchable, but that is a lie: first, it's 
history that should not be rewritten (any updated metadata doesn't 
affect it, replacing events with more modern implementations doesn't
alter history as well), second, part of history may be forgotten as long 
as it isn't needed - and that means retention control. For example, 
there may be some financial data that is kept only in case legal reports
would be necessary, and can be forgotten after X years, or some entity
that is changed frequently but needs only recent history may require to 
keep only X latest events (e.g. locks implemented over event sourcing). 
Frameworks should provide means to address both cases (keep events 
happened only after X / keep X latest events, and, perfectly, keep
events using judgement from specific interface that may be provided by
end user), because "immutable history" concept comes from idea of 
ability to audit all state changes - and that may be not necessary for 
end user.

This also has impact on storages: first, they have to distinguish 
requests *return events from 0 to X* and *return first X events*, 
because first one may legally return zero records, and the second one
has to return X events given there are enough of them in storage, and
second, they should provide ability to delete events, at least 
"purge first X events".

Finally, there is a restriction for framework too: it has no rights to
perform retention before all application hooks has successfully 
finished. Application may want to back up those events in coldline 
storage, and framework has to ensure it is safe to delete events.

### Compaction

Next thing is compaction: sometimes end user needs event sourcing to
temporarily store some metadata. Imagine some complex consensus 
algorithm that takes several steps, and each participant is an entity
in event sourcing. Each step will be recorded as entity event, but once
consensus has been reached or round has failed, there is no necessity
for per-step events (instead, only "round succeeded" or "round failed"
single event is required). So end user may want to perform compaction on 
event streams, reducing several events into one, both to preserve disk
space and make history more concise. Please note that as long as only
meta-events are deleted, entity history remains intact; it's not exactly
true for example above, but i couldn't come up with a better one.

Again, this puts restriction on storage: it has to support per-event
deletions and CAS operations.

### Pluggable backends and serializers

Many frameworks provide only preconfigured setup for storage and 
serialization, which usually serves as a blocker from adopting that 
framework. Both storage and serializer are easily abstracted into 
interfaces, so why not let user to deal with technologies that are 
already in theirs project? PostgreSQL, MySQL, Cassandra, Scylla, Redis,
JSON, YAML, protobuf - let end user choose it. 

### Per-entity configuration

This is quite obvious as well, but is also easily forgotten. Entities 
serve different purposes, and will have different needs (snapshot 
interval, retention, read repair probability, etc.), so framework has
to provide means to configure each entity type separately.

### Entity nullability problem

Another commonly unaddressed issue is that null is a valid entity state,
at least before first event and after deletion event has been recorded
(also lock example from above may represent lock "free" state as null).
That's another nail in coffin of "expose-all-methods-on-entity" 
approach since one can't call methods on null.

To address this, framework simply needs to move event processing outside
of entity, be it specific service or, as this article suggests, 
mutations held inside events.

### Dependency injection (ugh)

There is another problem when porting legacy things: tight coupling.
It's an expected case when user needs to migrate to event sourcing, but
theirs entities contain complex instantiation logic or, even worse,
depend on application services. Of course this smells, but life's a life
and frameworks should make hard things as easy as possible.

So decent framework should introduce concept of entity factories. Entity
factory is just a thing that allows to a) create new entity instance
(to apply first event on; as stated previously, it should be just null 
at this moment, but that would render dependency injection impossible),
b) populate deserialized instance (when it is restored from snapshot)
and c) provide a method to copy entity to allow keeping before and after
event application states. The latter requirement may be redundant if 
user's processing already operates over immutable objects, but in that 
case copy method may just return entity as is.

### Automatic event discovery

Another real pain in the ass is specifying all entity and event types by
hand. First, it's error-prone, second, there is no single reason not to
automate something while building framework for industry serving the 
only purpose of automating things.

While there are languages where it is challenging task, most of them 
have concepts of annotations / attributes, which usually fit for this
goal.

### Metadata

As been briefly noted before, end user should have ability to attach
metadata of any type to any particular event and snapshot, just because
it's not actual part of mutation and because it's, well, additional data 
that user needs. It's impossible to predict which data user will need
(application info, stack trace, state of business process when this 
event has been recorded, and so on), so there should be a last resort
for user to keep it.

Because snapshots are usually created in automated way, there should 
also be an option to pass user's defined metadata generator to snapshot
creation facility. 

### Event validation

This is something that framework doesn't necessarily have to have, but a
nice gesture from framework. It could not only provide means to insert
new event and see if that succeeded, but also provide interface to 
specify validators for entity type, and on event insertion read current
entity state, apply event over it and halt processing in case validation
fails. This is more on CQRS than ES side and generally should be 
implemented on upper layer, but since there are no guarantees user isn't
striving to escape from legacy code, it is just a nice finishing touch
that helps someone on their way.

### Listing entities

This is close to requirement, but is not a necessity. Frameworks
*should* provide means to list stored entities, so external systems 
could use that information to walk over all stored entities. One can 
argue that there are projections in place for that, but actually the
projection generator itself may be using such functionality to find out
stale entities and update them; also, there is a case when first event
is persisted, but projection hasn't been updated for any reason
possible, and then this entity is invisible for the system (except for
direct querying by id).

### Usage assumptions

This is more a rant than a specific requirement, but most of the 
problems described above arise from assumptions that user will need to
use framework just as author does, and this is not true. There should be
no assumptions about how and which case user will solve with framework,
there only could be some limitations that framework can't overcome at
this moment. And yeah, think about users with legacy code - their 
situation is most painful one.
