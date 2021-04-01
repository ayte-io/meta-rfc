---
title: Serialization
---

## Configuration

Every CM uses two entities for computing and executing actions to take:
code that defines scenarios to run and configuration which serves as 
input to those scenarios. It is very important that CM manages this 
configuration as a structure serialized in arbitrary format and not as
DSL calls because of the following:

- Such data can't be processed with external tooling
- Such data can't be safely passed between processes and different 
  machines
- Such data can't be processed in different language
- Language-specific issues will affect data

All those problems are solved by storing configuration in any 
serializable form that is not language-specific (e.g. not as Java 
serialized objects). Since every CM project has code counterpart, 
nothing prevents CM to use such approach for configuration.

## Actions & resources

Every scenario, as well as every library, must operate with basic 
structures, being able to output them in serialized format and feed into
different processes or code written in different language. First allows
to use computed scenario in different process, be it remote agent, 
analyzer, separate plugin process or something else, capturing 
dry-runs and resuming scenario processing, and second allows to write 
code for runtime that supports multiple languages.

TODO: Rationale for serializable Scope state
