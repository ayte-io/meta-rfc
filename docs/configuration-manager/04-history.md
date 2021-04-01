---
title: 'RFC: Configuration Manager / History'
---

History management is another important feature, which may be split 
into maintaining configuration as history and keeping history.

## Maintaining configuration as history

This article suggests different approach for change introduction. Every
configuration management project is split or eventually splits in code
that describes scenarios and configuration, static data, which allows to
apply that code and produce different results for different 
configuration (input). If configuration management project is stored in
VCS repository, then one may argue that history is already kept there, 
which is partially false and is not suited for use for the rest.

First, this is partially false because VCS stores history only for the 
repository itself, but not for the configuration that was applied. Some
commits may be skipped from run, resulting in different changes than 
expected. For example, if endpoints list of some service was changed
from A to B and then C, then it is impossible to determine whether or
not B was ever requested.

Second, instead of just having series of commits it is recommended to
express configuration changes _really as changes_, not through complete 
snapshots of configuration. Usually, configuration updates modify some
static file(s), so getting history from run X to Y boils down to manual
search of differences in two (or more, if some merging or splitting was
performed in between of commits). Instead of such approach this document 
suggests storing configuration differences in arbitrary 
format[^difference-format] which describes entries creation, 
modification and deletion (introduced entry with path X and value Y, set 
entry with path X to value Z, deleted entry with path X). CM
implementation may provide additional tooling to show history between 
two commits; it is also possible to extract this from VCS commits, but 
finding differences in two structures is a more complex task than just
interpreting such changes, support for which should be already in CM. 

## Keeping history

One also may argue that all the actions described above is enough to 
reconstruct every action done by CM. However, this is not true because 
only history for configuration and code was stored - but not the history
of runs. Example in section above is applicable here as well: if subset
of entities in scope were omitted from CM run, then they wouldn't see
B at all. Because of that it is suggested to keep history of runs and 
their configuration[^run-configuration] within arbitrary 
storage[^storage], and every entry has to keep information about all
scopes and their configuration (which can be stored as snapshot or 
recomputed on restoring history entry).

## Footnotes

[^storage] Obviously, there are no restrictions on storage 
implementation. It could be even the same VCS repo where whole project 
is stored.

[^snapshots] CM should periodically do configuration snapshots, i.e.
configuration state of particular version; since configuration formed 
from just a list of changes, desired state may be computed by applying 
all changes introduced in versions after that snapshot version. This 
optimization will allow avoid rebuilding configuration from scratch for 
every run, still preserving all other functionality. Some garbage 
collection should be implemented as well to preserve space, which is 
again totally safe to perform since every snapshot could be easily 
recomputed.

[^difference-format] The most common format for storing differences is
[JSONPatch](http://jsonpatch.com/).

## TODO

- Another part of persistence (but not exactly history) is current node
state, which together with configuration make input for scenarios.
