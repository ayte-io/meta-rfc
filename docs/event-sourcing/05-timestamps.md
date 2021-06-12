---
title: Timestamps
---

Another important aspect of event sourcing is persisting time. Both 
usage of stored events as history and debugging through inspecting them 
require attaching time of event (and snapshot) registration. Besides
storing timestamps there are two more things that must be taken into 
account when implementing event sourcing framework. 

First, it is necessary to store not only date and time, but time zone as 
well, because event _happened somewhere in the world at particular 
time_, not on server where it was only registered and nothing more. 
Leaving date and time without time zone makes it impossible to fully 
resolve context around particular stored mutation, like that Mike was on
other side of the world and this particular change was applied not at 
night as planned because _it was night there, not in the office 
timezone_. Even more, it's better to persist actual timezone rather its 
offset (as it is usually done via ISO8601 standard format). 

Second, there is difference between _event occurrence_ and _event 
registration_. Time at which server has processed message (registered 
at) is not the time of event happening (occurred at), _events (sic!) 
may be registered not in order they actually have happened_. So to 
properly restore whole history framework must also provide means to
persist time at which event happened client-side, even if it is possible
that client may have clock skew.
