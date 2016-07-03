---
title: How I reduced PostgreSQL memory usage from Sidekiq
date: 2016-07-03T23:48:45+02:00
categories: []
---

Like all good stories do, it started with a bunch of errors in my inbox:

```
ActiveRecord::StatementInvalid sidekiq#pro
  PG::ConnectionBad: PQconsumeInput() SSL connection has been closed unexpectedly : SELECT …
PG::ConnectionBad analytics:attribution
  could not fork new process for connection: Cannot allocate memory could not fork new process for connection: Cannot allocate memory could not fork new process for connection: Cannot allocate memory
ActionView::Template::Error admin#stats
  PG::OutOfMemory: ERROR: out of memory DETAIL: Failed on request of size 48. : SELECT … 
```

_"How strange!"_, I thought, _"I wonder what the memory usage looks like."_

![Saw pattern](/assets/images/posts/2016-07-03/saw-pattern.png){: class="centered"}

Well, saw patterns in graphs are rarely good. I drew the conclusion that these errors cropped up after we recently increased the number of Sidekiq
workers, which would increase the number of PostgreSQL connections, but a connection does not take
up that much memory… or does it?
