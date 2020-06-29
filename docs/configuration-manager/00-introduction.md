---
title: 'RFC: Configuration Manager / Introduction'
---

## Problem

Configuration management is not a new word, and many configuration 
managers exist, most popular of which at the moment of writing were
Ansible, Salt, Chef and Puppet, however, it was hard to find a person 
who is satisfied with their tool. All of the mentioned managers received
hard criticism, and overall picture was somewhat of "every piece of work
is an attempt to work around existing tooling" rather than "some bits
are out of canon and done dirty" or "some common action doesn't fit into 
domain", which signals that selected model for those managers seem to be
out of sync with real world.

Some common assertions that all of those managers share were:

- IaC, or Infrastructure-as-Code principle dictates that infrastructure
changes should be persisted and tracked like a regular software in scm
repository.
- Infrastructure is configured via *resources*, descriptions of 
particular service of feature state, e.g. that package `dnsmasq` is
installed, file `/etc/dnsmasq.conf` is set to proper content, and 
systemd service `dnsmasq` is also enabled and started.
- Configuration manager actions are idempotent: no matter how many
times they are ran, infrastructure state will stay the same.

All of them serve a simple model, which reduces configuration manager
to very simple software, but in fact it barely covers 90% of needs. IaC
principle was introduced as a concept of infrastructure history, but it
doesn't track what actually happened on hosts, and, if not guarded by
automation, doesn't guarantee that someone hasn't run it with 
non-committed configuration. While resource concept seems to be very 
natural, it usually leads to some extra functionality, like restarting
the service after configuration file update, and restart is an _action_,
not a resource, and while generally it's just a simple restart, 
sometimes very quirky actions should be taken that are dependent on 
specific environment, like a database server updated followed by 
migration, or when shrinked cluster members should be explicitly told 
that node is not going to return. Idempotence principle allows 
configuration manager to force user to write idempotent code, which 
usually ends in either code with tons of did-this-run checks or code 
that is executed on every run, increasing overall running time.
