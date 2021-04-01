---
title: Introduction
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
the service after configuration file update, but restart is an _action_,
not a state, and while generally it's just a simple restart, 
sometimes very quirky actions should be taken that are dependent on 
specific environment, like a database server update followed by 
migration, or when shrinked cluster members should be explicitly told 
that node is not going to return. Idempotence principle allows 
configuration manager to force user to write idempotent code, which 
usually ends in either code with tons of did-this-run checks or code 
that is executed on every run, increasing overall running time.

Next problem is that all configuration managers are designed to 
manage hosts. While this is most common type of operation, this is not 
enough for infrastructure management, since there are other scopes, for
example, cluster scope. Current model implies that nodes are configured
without taking wider scope into account, thus it's possible to have 
successful configuration manager for distributed software without 
actually forming a live and healthy cluster. Continuing, configuration
managers also do not have proper integration with verification tools 
which would allow to validate post-converge infrastructure state and be
sure that _software works_ rather than _sequence of actions was 
successfully applied_.

Another problem is idempotency. If N of M hosts have failed during 
converge, there is probability that they are in fact in inconsistent 
state, which has to be dealt with in ways usually not coded. Consider 
example of database instance that went down during upgrade: it should be
considered broken at this point, and running converge again is unlikely
to do any better. Proper way to deal with it is manually copy current 
machine state, develop recovery script, then code it via configuration
manager and apply it again, which bloats stored code up with one-time 
ad-hoc recovery action and checks to define whether or not current node
is damaged, and usually it is done by hand, bypassing the CM (whose only
purpose is to __prevent__ necessity to do things by hand). One more 
common example could be naming schema change: should one decide to put 
configuration files not in `/etc/mgmt` but `/etc/<configuration name>`,
they would need to clean up `/etc/mgmt` as well, resulting in extra 
work, while this could be done in automated way.

Also, there could be a lot of boilerplate regarding putting node into
out-of-service state (put status in eureka, kubectl drain, etc., etc.),
thus CM must have tight integrations with cloud providers and
orchestrating software, as well as provide an option for building custom
integrations in format of plugins.

Lastly, speaking from a very high level, we do need software that solves
our everyday needs. But even more we need software that works in and 
aims for worst case scenario and has notion of rollbacks, contingency 
plan and other things.
