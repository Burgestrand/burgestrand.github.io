---
title: Hallon, delicious Ruby bindings for libspotify
date: 18 July 2011 10:25:54 UTC
categories: [ruby, spotify, libspotify]
slug:
  I’ve always loved the "Spotify":http://spotify.com/ service. I
  love it so much that I find no need to pirate music anymore, and
  I’ve been a paying customer for nearly a year. Me being me, that’s
  nothing short of an amazing acomplishement.


  Spotify also has a developer API, "libspotify":https://developer.spotify.com/technologies/libspotify/,
  written in C. Thing is, Ruby is a much nicer language to use than C,
  so what to do?
---

First, a little bit of background. When I first joined the [Radiofy](http://radiofy.se/)
project 18 months ago, we were using a java variant of libspotify named Jotify.
Jotify worked quite well for us, but after six months it suddenly stopped working.

So, what gives? From what we gathered, Spotify had updated their protocol,
right at the same time the Jotify project was abandoned by its’ author, and all
other alternative projects either didn’t have required functionality or plain
out didn’t work for us. Having a service relying very much on having a Spotify
API we had to find a solution.

And so, [Hallon](http://github.com/Burgestrand/Hallon) was born. During the
weekend I revisited my C knowledge, studied up on [Ruby C extensions](http://www.ruby-doc.org/docs/ProgrammingRuby/html/ext_ruby.html)
and quickly wrote the most minimal thing that could possibly work. By Monday
we had our service up and running, using Hallon as back-end. And that’s how
Hallon was born.

So, what is Hallon, really?
---------------------------
Hallon is Swedish for [raspberry](http://www.flickr.com/photos/todorrovic/4815941952/),
and is really just one side of a coin.

On one side we have [libspotify for Ruby](https://rubygems.org/gems/spotify).
This gem, Spotify, is a ruby wrapper for the libspotify API. What this
means is that anything you can do in C with libspotify, you can now do with Ruby.

However, as the Spotify gem is written using [Ruby FFI](https://github.com/ffi/ffi),
you still need to manage memory by yourself, freeing pointers where necessary and
all other kinds of stuff that we should be spared from doing when coding Ruby.

[Hallon](https://rubygems.org/gems/hallon) is meant to remedy this. Hallon is
is a layer written on top of the Spotify gem. The goal of Hallon is to provide
a simple-to-use API, allowing you to worry about things you enjoy worring about,
rather than thinking about memory or pesky pointers.

### Why two gems? Isn’t Hallon enough?
Hallon is not complete, and is *much* harder to implement than the Spotify gem.
Luckily, when Hallon’s functionality is not enough, one can use the Spotify gem
to fill in the gaps, providing an all-around solution until Hallon reaches
v1.0.0.

However, do note that when Hallon reaches v1.0.0, there will be *no* reason
to use the Spotify gem directly, and Hallon’s API will easy and fun to use.
That is my goal.

Finally, this post would not be complete without a code sample.

{% highlight ruby %}
require 'hallon'

session = Hallon::Session.instance(spotify_appkey, user_agent: 'Hallon') do
  on(:connection_error) do |error|
    puts "[ERROR] %s" % Hallon::Error.explain(error)
    abort
  end

  on(:log_message) do |message|
    puts "[LOG] #{message}"
  end
end

session.login username, password
session.wait_for(:logged_in) do |status|
  Hallon::Error.maybe_raise(status)
end

# make absolutely sure we’ve logged in
session.wait_for(:connection_error) do |error|
  session.logged_in? or Hallon::Error.maybe_raise(error)
end

image = Hallon::Image.new("spotify:image:3ad93423add99766e02d563605c6e76ed2b0e450")
session.wait_for(:metadata_updated) { image.loaded? }

puts "Image format: #{image.format}"
puts "Where to save raw image data?"
path = gets
File.open(path, 'w') { |f| f.write(image.data) }
puts "Image saved to #{path}!"

{% endhighlight %}

Thank you for reading! If there’s any questions, feel
free to contact me. My details can be found on the [About Me](/about-me.html) page.
