---
layout: post
title: Asynchronous callbacks in Ruby C extensions
date: 17 May 2011 06:15:12 UTC
categories: [ruby, yarv, async]
slug: Half a year ago, I was working on bindings for a C library known as
      "libspotify":http://developer.spotify.com/en/libspotify/overview/.
      As you might know, writing C extensions for Ruby is _really_ easy,
      but this particular case was not.


      libspotify relies _heavily_ on user-provided callbacks for event
      handling. These callbacks are sometimes executed in their own
      thread, and those threads are not allowed to call the Ruby API.


      Now, how do you handle these callbacks if you cannot call Ruby code
      from within them? I’d like to tell you the answer to this question.
draft: yes
---


**WARNING**: this post will be very technical and mention topics related to concurrent programming and thread-synchronization. Knowledge about [mutexes](http://stackoverflow.com/questions/34524/what-is-a-mutex/34558#34558) and condition variables will be assumed.

Okay. Down to the nitty-gritty. You have this awesome (and imaginary) C library, “library of massive fun and overjoy”, and you can call upon it to do some work. Thing is, when LMFAO does some work it spawns a new thread, and that thread will call a callback-function that you supply.

Today, we are going to write a C extension for Ruby that allows our fellow Ruby programmers to use LMFAO without knowing an ounce of C; and to do that you’ll need the source, so here it is:

<script src="https://gist.github.com/974171.js"> </script>

## Writing the Ruby bindings
This is not a basic guide in writing Ruby C extensions, so if you’ve never written one yourself this particular part might confuse you slightly. Don’t fret! Go forth slowly, Google the things you don’t understand and you’ll understand in no time.

Now, where were we? Oh, yes, Ruby bindings for LMFAO! We will be supporting an API similar to this:


{% highlight ruby %}
require 'lmfao'

result = LMFAO::call("some ruby object") do |data|
  puts "LMFAO callback called"
  data.upcase # handle the data from the callback somehow
end

puts "Result: #{result}"
{% endhighlight %}


I’ve taken the liberty of writing most of it for you. It is available in a GitHub repository: [Burgestrand/Library-of-Massive-Fun-And-Overjoy](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/tree/problem). Once you run LMFAO (`rake default`) you’ll notice the tests don’t pass: the callback returns `false`.

You might not realize it yet, but we have a major problem here. Inside [`lmfao_callback`](http://goo.gl/awsR8) we do not hold the [GIL](http://en.wikipedia.org/wiki/Global_Interpreter_Lock), so we *cannot* call the Ruby C API safely. [`rb_thread_call_with_gvl`](https://github.com/ruby/ruby/blob/ruby_1_9_2/thread.c#L1170) looks promising at first, but we cannot use it as the the current thread was not created by Ruby. So, what do we do?

## Solving the communication problem

As we are not allowed to call Ruby from within our `lmfao_callback`, we need a workaround. Now, this is the tricky part, so stay close with me.

Once the callback fires, we need to tell Ruby that it has fired, and we also need to communicate which parameters it was given. As we cannot call Ruby directly, we need to put the parameters in a location that Ruby can access. Additionally, the callback must wait for a return value from Ruby before it can return said value to its’ caller.

Now, since want to listen for notifications from our callbacks we must wait for them to arrive. Chances are we don’t want to do this in our main code, as we would never get anything done if all we did was wait. What we *can* do, though, is do the waiting in a separate Ruby thread.

So, a quick recap:

- we have a special ruby thread, waiting to be notified
- when a callback is invoked, it stores its’ parameters somewhere, notifies ruby thread and waits
- ruby thread is notified, reads the callback parameters, and executes the callback handler
- ruby thread puts the return value of the handler in location where C callback can reach it, and notifies C callback
- C callback wakes up again, reads return value and returns it

Whew! A lot of things to keep track of, but this is a high-level view of what we need to do. Lets get to work!

### The Ruby Event thread

First off, we’ll need a designated event thread. As previously mentioned, this thread will do nothing but wait for callbacks to happen; and when they do, it will dispatch off to a callback handler.

Yet again, [the (new) code for this chapter is available on GitHub](http://goo.gl/Aw8ze). I’ve done my best to explain what is being done and why for each function and struct member. Still, if something is unclear I’d love to try and clear it up for you. You can find my e-mail [at the about me-page](/about-me.html). I don’t bite, I promise `:}`

### `LMFAO_handle_callback` and `lmfao_callback`

You’ve probably noticed both `LMFAO_handle_callback` and `lmfao_callback` are empty functions. We’ll fill them in in this chapter, but they require more intimate discussion in comparison to the ruby event thread.

---

**NOTE:** Content below is old and will be replaced. It is merely here to remind me of what I’ve previously written.

## Related
- [Avoiding cross-thread violations in a Ruby extension](http://stackoverflow.com/questions/3752006/how-do-i-avoid-cross-thread-violations-in-a-ruby-extension)
- [Function.c in Ruby FFI](https://github.com/ffi/ffi/blob/85e431eb13ed96d3926fbd82e2ece7f5d93156f3/ext/ffi_c/Function.c#L470)
- [rb\_thread\_blocking\_region](https://github.com/ruby/ruby/blob/4db93c3f41818261121d53214030aad6ec001ee7/thread.c#L1119)

## The final recipe
To summarize, here’s what we have:

- a C callback, called from a thread that cannot hold the GVL
- a *ruby* thread that does nothing but wait for any callbacks to fire
- our main ruby thread

We want to know when the C callback is invoked, and what parameters it was given. It would also be nice to be able to return a value from this callback (if required). The entire flow is like this:

1. (ruby): wait for the callback to fire
2. (callback): wait for permission to access shared storage
3. (callback): put parameter data into shared storage
4. (callback): signal ruby that data has been delivered
5. (callback): wait for a return value
6. (ruby): read the data from shared storage
7. (ruby): marshal the data into something usable within ruby
8. (ruby): handle the data (e.g. pass it to a user-defined callback)
9. (ruby): unmarshal our ruby return value for our callback
10. (ruby): store the return value where the callback can reach it
11. (ruby): signal the waiting callback
12. (callback): read the return value
13. (callback): return the return value
14. (ruby): give permission for more callbacks to fire

&#13;<small>(note: point #8-13 can be handled concurrently with point #14 by having the C callback pass along means for ruby to signal with when the callback is handled)</small>
