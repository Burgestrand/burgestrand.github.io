---
title: Asynchronous callbacks in Ruby C extensions
date: Wed 15 Jun 2011 10:37:25 UTC
categories: [ruby, yarv, async]
excerpt: Half a year ago, I was working on bindings for a C library known as
      "libspotify":http://developer.spotify.com/en/libspotify/overview/.
      As you might know, writing C extensions for Ruby is _really_ easy,
      but this particular case was not.


      libspotify relies _heavily_ on user-provided callbacks for event
      handling. These callbacks are sometimes executed in their own
      thread, and those threads are not allowed to call the Ruby API.


      Now, how do you handle these callbacks if you cannot call Ruby code
      from within them? I’d like to tell you the answer to this question.
---


**WARNING**: this post will be very technical and mention topics related to concurrent programming and thread-synchronization. Knowledge about [mutexes](http://stackoverflow.com/questions/34524/what-is-a-mutex/34558#34558) and condition variables will be assumed.

Okay. Down to the nitty-gritty. You have this awesome (and imaginary) C library, “Library of Massive Fun And Overjoy”, and you can call upon it to do some work. Thing is, when <acronym title="Library of Massive Fun And Overjoy">LMFAO</acronym> does some work it spawns a new thread, and that thread will call a callback-function that you supply.

Today, we are going to write a C extension for Ruby that allows our fellow Ruby programmers to use LMFAO without knowing an ounce of C; and to do that you’ll need the library’s source, so here it is:

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

You might not realize it yet, but we have a major problem here. Inside [`lmfao_callback`](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/problem/ext/lmfao_ext.c#L42) we do not hold the [GIL](http://en.wikipedia.org/wiki/Global_Interpreter_Lock), so we *cannot* call the Ruby C API safely. [`rb_thread_call_with_gvl`](https://github.com/ruby/ruby/blob/ruby_1_9_2/thread.c#L1170) looks promising at first, but we cannot use it as the the current thread was not created by Ruby. So, what do we do?

## Solving the communication problem

As we are not allowed to call Ruby from within our `lmfao_callback`, we need a workaround. Now, this is the tricky part, so stay close with me.

Once the callback fires, we need to tell Ruby that it has fired, and we also need to communicate which parameters it was given. As we cannot call Ruby directly, we need to put the parameters in a location that Ruby can access. Additionally, the callback must wait for a return value from Ruby before it can return said value to its’ caller.

Now, since we want to listen for notifications from our callbacks we must wait for them to arrive. Waiting in the main thread is a bad idea, as it would mean we did a lot of waiting and very little work. What we *can* do, is use a separate Ruby thread that’ll wait for us.

So. Quick recap:

- we have a special ruby thread, waiting to be notified
- when a callback is invoked, it stores its’ parameters somewhere, notifies ruby thread and waits
- ruby thread is notified, reads the callback parameters, and executes the callback handler
- ruby thread puts the return value of the handler in location where C callback can reach it, and notifies C callback
- C callback wakes up again, reads return value and returns it

Whew! A lot of things to keep track of, but this is a high-level view of what we need to do. Lets get to work!

### The Ruby Event thread

First off, we’ll need a designated event thread. As previously mentioned, this thread will do nothing but wait for callbacks to happen; and when they do, it will dispatch off to a callback handler.

As I’ve already explained the high-level view of how this should work, I’m just going to give you [the code](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/event-thread/ext/lmfao_ext.c). I’ve done my best to explain what is being done and why for each function and struct member in their comments. Do read it now, it’ll help understanding what’s coming next!

### Calling out to Ruby

You’ve probably noticed both `LMFAO_handle_callback` and `lmfao_callback` are empty functions. We’ll fill them in in this chapter, but they require more intimate discussion in comparison to the ruby event thread.

We’ll talk about [`lmfao_callback`](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/finished/ext/lmfao_ext.c#L91) first, the simpler one of the two functions. This function should dump its’ data in the global queue, notify the event thread, and wait for the return value. Only two things in this code should ever change between different callbacks: the [parameter dumping](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/finished/ext/lmfao_ext.c#L96) and [type casting the return value](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/finished/ext/lmfao_ext.c#L119).

As the parameters become more complex, so does parameter dumping. I’ve thought about making the `data` field in the `callback_t` struct a linked list instead. Each node would contain the data type, pointer to the value and finally a pointer to the next node. I think I’ll leave this an exercise for you!

### Handling the callback

Now to look at [`LMFAO_handle_callback`](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/finished/ext/lmfao_ext.c#L136). In LMFAO, the callback data is just a Ruby array containing a proc and the parameters to give it. We `call` it, and simply return the result to the callback (lines [`#146` to `#153`](https://github.com/Burgestrand/Library-of-Massive-Fun-And-Overjoy/blob/finished/ext/lmfao_ext.c#L146-L153)).

In practice, it is never this simple. You need to convert the callback data to Ruby data, figure out which Ruby handler to invoke, and finally convert the result back to pure C data that the callback function can return.

If you have a small amount of callbacks, you could handle these conversions for a few simple data types (or in LMFAO’s case, no conversion whatsoever). If you want to handle the general case however, it quickly gets complicated. Ruby FFI has an implementation of this in its [`callback_with_gvl` and `invoke_callback`](https://github.com/ffi/ffi/blob/master/ext/ffi_c/Function.c#L747) functions.


## Summary

We’ve written a C threaded (albeit small) library, a C extension binding it and a whole lot of concurrency-related code. If you are still confused about all this, you are not alone; concurrency is hard!

If you have suggestions, ideas or any other feedback you’re welcome to contact me. You’ll find my contact details on the [About Me](/about-me/) page.

*Final note: do keep in mind that this entire article is a proof-of-concept. My hope is that if you ever find yourself needing to do this (shudder), you now have a better idea of how.*
