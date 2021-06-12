---
title: Persisting additional contexts
---

Previous article about persisting event date and time introduces another 
not very obvious functionality that framework has to expose. There is a
concept of additional contexts (metadata) that don't belong to mutation
itself but have to be stored for every event, a thing that is _attached
outside_ to the mutation rather than being a part of it. As stated 
before, an example of such context is date and time of event occurrence:

```yaml

```

However, there could be much more metadata to attach, and what makes it 
worse is that there could be more than one producer of such metadata.

Previous article about storing timestamps with the mutation unveils 
another point of complexity. There may be a lot of additional 
information that application would need to store with mutation itself -
simplest examples would be storing ID of node that performed the 
operation, user account that triggered operation, external reference or 
a/b testing configuration that resulted in creating this event. This all
leads to necessity of storing not only mutation itself, but also some 
application-provided context along with it. As a consequence, all 
mutation requirements apply to the context as well, including type 
referencing, schema evolution and so on.

It is worth noting that while for single entity there would be a set of 
mutation types, event context would be represented by a single type 
(that may evolve over time). At the same time, context type is not 
guaranteed to be the same across different entity types.

In the end, record with mutation may look like this:

```yaml
entity:
  type: account
  id: '1'
mutation:
  type: hold
  schema: 1
  content: { ... }
context:
  ayte.io/framework:
    occuredAt: 2029-09-09T12:12:12.567+03:00
    registeredAt: 2029-09-09T09:12:12.768Z
  application/default:
    schema: 1
    content:
      operation-id: %uuid%
      client: %service-name%
  application/testing:
    schema: 2
    content:
      variant: red-cta-button
```
