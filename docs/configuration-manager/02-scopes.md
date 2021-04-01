---
title: 'RFC: Configuration Manager / Scopes'
---

Repeating introduction, there is a doubt that single execution scope
(host that runs CM agent) is enough to fully embrace needs of automatic
infrastructure management. This RFC proposes six scopes, which may also
be just a subset or superset of all real scopes faced by engineers in
particular scenario.

## Host

This scope is restricted to single machine only, without any restriction
on what that machine could be: VPS, virtual machine, dedicated server,
raspberry pi or something else. In this scope manager simply configures
this host operating system and software and hasn't any direct interest
on what happens on other nodes. There could be indirect interest, for 
example, waiting until all hosts in group complete specific stage, but
this example is _waiting for event_ and not _querying other nodes 
state_.

## Supervisor

Supervisor represents host which may contain and spawn other virtual 
hosts. It doesn't differ from "just host" much, but can take into 
account what happens on child hosts it manages.

Whether configuration manager is allowed to create hosts using
supervisor, which may be considered as a thing outside of it's 
responsibility, is not specified or suggested in this RFC.

## Cluster

Cluster represents a group of nodes, or, more common, a distributed
application that runs on those nodes. This scope doesn't account for 
particular details of hosts configuration and it is concerned about 
group as a whole rather than a collection of similar units. Hosts, even
when configured correctly, may prevent cluster from forming, thus 
host-level scope would be valid, but distributed application isn't
running as intended or not running at all. Cluster scope also allows to
apply some application-level configuration (e.g. importing elasticsearch
templates), which is possible to do via classic host-only model, but is
in fact painful and requires knowledge about other nodes, since it needs 
to be applied only once.

## Fleet (System?)

Fleet is a union of all hosts and supporting infrastructure. Fleet 
doesn't care about clusters or hosts, it's only concern is a system 
functioning as a whole, so if cluster completion check is querying 
elasticsearch via internal ip and port, fleet check would be querying 
system by it's domain name(s), preferably outside of datacenter where
system resides.

Fleet also corresponds to environment: one fleet could be production,
another one would be staging, and the third one would be automatically
created testing fleet on subdomain for particular branch. Fleets can be
nested, for example big product usually can't afford creating a 
completely isolated system for testing single branch of just one of 
tens / hundreds / thousands of applications, but it makes sense to spawn
additional fleet of this single application and it's dependencies within
a staging fleet.

## Organization / Department

Complete projects are quite often split vertically into smaller systems, 
which means every subsystem manages it's own infrastructure. This scope
allows to bind several fleets to one entity which is not a
root-of-them-all. It is also quite common to have hierarchy on this
level, where department may have some internal infrastructure for 
developers and separate fleet collections for department A and 
department B, so every participant in this hierarchy has it's own 
managed infrastructure and may be aware of upstream and downstream 
infrastructure. For example, _search_ team may have some infrastructure
for daily ingestion of sampled data for staging and have two children,
_frontend_ and _backend_, each having additional infrastructure.

## Universe

Universe encompasses everything. This scope is mostly used to perform
tasks as cloud account management, issuing new certificates from private
CA and so on - things that are managed once for whole product.

## Consequences

Since execution on most scopes is not tied to _any_ host in particular,
it is implied that there is/are master server/s (and, possibly, 
intermediate servers for performing validation checks) which executes
higher-level operations like private CA management, making masterless
model impossible (one wouldn't pass root certificate to each host so 
that host could issue child certificate on it's own). However, while 
this document doesn't contain actual examples of that, it is believed 
that having push-only model isn't a viable solution too: master should 
be able to request some operations on agents and/or metrics & reports, 
but agents may have necessity to request some resources as well, for 
example, as a part of proactive actions (this example is a bit off the 
track, but if an agent detects abnormal user activity, it may request 
master to mark this host as potentially compromised).

### Top-down communication examples

- One of security primitives (e.g. certificate) is considered 
  compromised. Master then issues request to all agents to forbid 
  accepting such primitive.

### Bottom-up communication examples

- Agent brings up Docker containers on host and then sends assigned IP
  addresses to master.
- Agent brings up another kind of distributed application on host and
  then sends application-generated unique host ID to master.

and pull model hard
to implement (it's end host that would need to request certificate 
management operations, thus operations like certificate revocation may 
be tedious, and push model doesn't have such a drawback).

## TODO

- Should environment be a separate scope? It is not exactly a thing that
CM configures, but on the other hand it may make sense to have several 
fleets per environment.
