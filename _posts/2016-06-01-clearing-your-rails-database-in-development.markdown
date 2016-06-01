---
title: Clearing your Rails database in development
date: 2016-06-01 17:17:00 +0200
categories: [ruby, tools, rails]
---

You ever been hit with this?

``` shell
$ rake db:reset
PG::ObjectInUse: ERROR:  database "yolo_development" is being accessed by other users
DETAIL:  There is 1 other session using the database.
: DROP DATABASE IF EXISTS "yolo_development"
$ rake db:drop db:create db:migrate db:seed
PG::ObjectInUse: ERROR:  database "yolo_development" is being accessed by other users
DETAIL:  There is 1 other session using the database.
```

During development in Ruby on Rails it's often necessary to wipe the database,
seed, and start over.

In my case, I often also do not want to re-migrate the entire thing just because
I want a new set of data. Until recently I used to terminate my Rails server, run
`rake db:reset`, and I'd be happy and annoyed at the same time.

Today I just run `rake db:prune db:seed` and I will have a fresh database, and
now you can too! Meet [rails_prune](https://github.com/Burgestrand/rails_prune),
the first gem that I've created where the implementation is shorter than its announcement!

*PS: It works fine with foreign key constraints.*
