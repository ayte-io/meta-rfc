---
title: Suggestions
---

This document suggests using some features that are completely 
unnecessary and may be out of scope of this RFC. Those suggestions are
just recommendations on how software may be more awesome.

## Polyglot implementation

While technology isn't fully there, it is possible to write and execute
code in different languages in one runtime, for example IKVM or GraalVM.
It is highly recommended to allow users such feature, because end users
would like to write in a simplest language possible (python, ruby),
while plugins and modules may greatly benefit from static typing.

## Single server-agent binary

One feature that CM can provide is integrating agent binary within self.
This would greatly benefit node provision (since there would be no need 
to pull external resources) as well as ability to work in infrastructure
with restricted access to internet.

## Integration with existing package repositories

An option to simplify dependency management is just to use existing
package repositories for storing artifacts. Search capabilities can be
easily provided using tags as filters and specific configuration may be
exposed in package manifest or in embedded resource.
