---
title: How I reduced PostgreSQL memory usage from our Sidekiq workers
date: 2016-07-07T00:00:00+02:00
categories: [rails,postgres,debugging]
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
<figcaption class="caption">
  It was actually worse than this and hit 1GB, but I don't have an earlier image.
</figcaption>

Well, saw patterns in graphs are rarely good. These errors cropped up after
increasing the number of Sidekiq workers, and thus also the number of
connections to the database.

After some intense googling, I found out that [PostgreSQL stores PREPAREd statements per session](https://www.postgresql.org/docs/current/static/sql-prepare.html).
I know Rails uses prepared statements, and I know prepared statements need to be
referenced by name, which means Rails keeps this name around somewhere. After
looking through the source of Rails on GitHub, I found this:

```ruby
# https://github.com/rails/rails/blob/v5.0.0/activerecord/lib/active_record/connection_adapters/statement_pool.rb#L6
module ActiveRecord
  module ConnectionAdapters
    class StatementPool # :nodoc:
      include Enumerable

      DEFAULT_STATEMENT_LIMIT = 1000
```

So, turns out Rails by default stores the 1 000 most recently used prepared statements.
Quick math! With 30 workers, we can expect 30 000 prepared statements kept
around in memory. All in all it's not that much, but we're sporting a humble 1GB of memory, so it
turns out that this is enough to hit the roof!

So, what to do? More googling! Turns out that Rails has support for disabling
prepared statements, and it's had it [since at least Rails v4.0](http://guides.rubyonrails.org/v4.0/configuring.html#configuring-a-postgresql-database)!
So, I sent a pull request with a configuration change to disable prepared statements
for our Sidekiq workers:

```diff
default: &default
  adapter: postgresql
  ⋮
+  prepared_statements: <%= ! Sidekiq.server? %>
```

Once merged and deployed, memory usage for PostgreSQL dropped considerably, and
has since been stable at just under 200 MB!

![Post-fix](/assets/images/posts/2016-07-03/post-fix.png){: class="centered"}
![Stability](/assets/images/posts/2016-07-03/stability.png){: class="centered"}
<figcaption class="caption">
  Blue line is postgres memory, green line is free memory.
</figcaption>

This probably comes with a cost of increased load, the good old Memory vs CPU
tradeoff. We could possibly have lowered the [statement limit](http://guides.rubyonrails.org/v4.2/configuring.html#configuring-a-postgresql-database)
(available since at least Rails 4.2) but the load on our servers
is not too high, so we're happy with just turning them off!
