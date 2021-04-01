---
title: Impossibility of fully declarative approach
---

It is tempting to create a configuration manager that works as a pure 
function accepting YAML files and nothing more. However, this approach
is doomed to fail simply because of non-deterministic actions result of 
which can't be computed in advance. 

Another point in not doing that is user friendliness. Going 
fully-declarative will require a lot of work in creating new extension
just to call it then with a YAML file, and it is expected that extension
development would be much more complex than scenario writing. Even if CM
promotes declarative usage, it should always allow fallback option to
let users work on putting off fires rather than developing throwaway
extensions.
