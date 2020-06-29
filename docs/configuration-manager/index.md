---
title: 'RFC: Configuration Manager'
---

## Primitives

### Host

Host is a single machine that is managed by configuration manager. It
doesn't matter if machine is a container, virtual machine or bare metal
server, or if such machines are nested into each other.

### Cluster

Cluster is a collection of hosts that serve same role. From 
configuration manager's perspective, "same role" is a currently 
executing code and it's configuration, i.e. all hosts that this code
will be ran at with same configuration form a cluster. It is possible
to have several nearly identical and/or overlapping clusters, for 
example host A / host B may hold ElasticSearch data nodes (cluster 
ElasticSearch/Data), and host B / host C may hold ElasticSearch master 
nodes (cluster ElasticSearch/Master).

Cluster term may be useless on low-level operations like kernel 
configuration, but moving onto higher level cluster starts to mean more
and more.

### Role

?

### Fleet

Fleet is a collection of all hosts configuration manager operates under
current environment. 


- host
- cluster
- role
- fleet
- resource
- action
- event
- listeners
- reports
- query
- semaphore
- condition
- environments (i.e. fleets discriminator)

- resources and actions
- conditions
- cluster querying
- monitoring & taking actions
- push/pull/federation
- rolling updates & locks
- rollbacks
- history
- one-time ad-hoc actions that do not go into repository
- add history entries
