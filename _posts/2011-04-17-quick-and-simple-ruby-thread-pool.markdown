---
layout: post
title: Quick and simple Ruby thread pool
date: 17 Apr 2011 12:25:54 UTC
categories: [ruby, mri, concurrency]
slug: Ruby MRI threads suck! You should use "JRuby":http://jruby.org/
      for *REAL* concurrency and leave those pitiful green-threaded
      people in the dust… NO! If you think this is true, kindly relocate
      yourself to "dunce’s corner":http://en.wikipedia.org/wiki/Dunce_cap
      and listen.


      ("or skip to the code":/code/ruby-thread-pool)
---

Two days ago, somebody — let us call her Imogen — asked for resources on threading in the [#ruby](irc://irc.freenode.net/#ruby) IRC channel. Imogen was instantly instilled with [fear, uncertainty and doubt](http://en.wikipedia.org/wiki/Fear,_uncertainty_and_doubt) from others in the channel; people whom I otherwise hold great respect for.

> &lt;person A> with ruby, don’t \[write threaded code\]
> &lt;person B> doesn’t really help writing threaded code unless you're on jruby
> &lt;Imogen> but for my heavily I/O-wait jobs, mri threads seems like a good fit?
> &lt;person A> for concurrency in MRI use [spawn](http://www.ruby-doc.org/core-1.9/classes/Process.html#M002230)
> &lt;person B> 1 GIL and you’re toast
> &lt;Imogen> [*sad panda :(*](http://i55.tinypic.com/244fq.jpg)

Now now, Imogen, don’t be a sad panda. While it is true that [Ruby MRI](http://www.ruby-lang.org/en/) will only run one thread at a time ([although that is not the whole story](http://stackoverflow.com/questions/56087/does-ruby-have-real-multithreading/57802#57802)), that does not mean threads are useless. To me, it sounds like your jobs are a perfect fit for ruby threads!

## Ruby MRI, I select() you!
It turned out that Imogen wanted to execute a large amount of jobs using [system()](http://www.ruby-doc.org/core/classes/Kernel.html#M001441). She did not care about order; only that scheduled jobs are finished eventually, and that she will be able to wait for any pending work to finish before quitting. This is a perfect candidate for threading.

Yes, ruby runs only one thread of ruby code at a time because of the GIL, and no, this is not an issue. Ruby is very smart when doing `system()` calls: even if the call might block indefinitely, other threads will continue to be scheduled by Ruby until the `system()` call returns. See here:

{% highlight ruby %}
work = Thread.new do
  puts "Hi!"
  system('sleep 4')
  puts "Still here!"
  system('sleep 4')
  puts "Bye!"
end

puts "waiting…" until work.join(1)
{% endhighlight %}

The above code will continuously print “waiting…” while it is waiting for the `system()` calls to finish. One could say that Ruby threads are merely a way of having Ruby do a [select()](http://linux.die.net/man/2/select) loop for you, and this is good news for Imogen!

## Ruby threads are not free
Imogen initially spawned one new thread every time she needed some work done, and there’s a problem with this approach. If you start too many threads too often, or have too many threads running at the same time, performance will suffer. Ruby ends up juggling your threads more time than it is running them.

As I am sure you can imagine by now, a thread pool is what we need. Something that waits for us to give it some work, and then we simply forget about it. I explained this to Imogen. However, as she did not know where to start, and knowing it’d take me ~10 minutes, I offered to help.

## [Ruby Thread Pool](/code/ruby-thread-pool)
It is simple, and it works. If you fire an exception inside a job, you *will* kill a worker… and you don’t want to kill your hard working threads, do you?

Also, [here’s the source code](/code/ruby-thread-pool/thread-pool.rb).
