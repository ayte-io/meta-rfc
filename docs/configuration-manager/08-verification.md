---
title: Verification
---

Every CM **must** embed tooling or support for tooling that allows to
verify state of particular scope after converge, scenario or step is 
run. Even thoroughly tested modules and libraries may behave in an 
unexpected way in edge cases, and converge verification is a necessary
stage to alert responsible people. Currently, most of CMs try to 
integrate it into scripts, but lack of proper tooling makes it useful 
only in simple cases; also scenario failure is not the the same as 
verification failure (and going a bit further, scenario failure and
code exception during scenario execution are different things as well).

CM _should_ also provide an option of regular verification runs to
detect infrastructure problems early. This is very close to regular 
alerting, but alerting doesn't account for scopes, also this would
eliminate necessity of configuring verification/alerting tooling the 
same way in two places. Another thing that can be implemented using
regular verification is proactive actions: either fully automated or
human-approved scenarios that are executed in reaction to specific 
events, for example regenerating configuration on node being shutdown.
