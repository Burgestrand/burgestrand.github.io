---
title: A walk down memory lane for my ruby gem Spotify
date: 2017-11-09T10:16:09+0100
categories: [ruby, c, raspberry pi]
---

We're in the second quarter of 2010. It's May, spring time in Sweden, and I'm
working with a friend from University. Together he and I are rewriting his hobby
project from PHP to Ruby on Rails, mainly because Rails is cool and none of us
have any Ruby experience.

> What is your hobby project about?

It's a website that lists all Swedish radio channels, along with all the songs
they've played day-by-day. We have new visitors in the low thousands every day.

Oh, yeah, I almost forgot. Spotify is hot shit right now. We built it so that
you can click any of the songs to open it in Spotify and listen to it!

> Okay, cool, so what are you working on right now?

So, we have this idea, get this: auto-updating playlists in Spotify for every
radio channel. We'll have `P3 Sverige - 2010-05-26`, and you can listen to all
songs they played for that day! We're having some problems with it, though.

> Yeah? Why?

Spotify doesn't really have an API that we can use. They've released an official
library called `libspotify`, but none of us knows C. Heck, we don't even really
know Ruby, we just started three weeks ago.

Luckily my friend found an unofficial Spotify SDK written in Java, Jotify, and we've
been using that for a month or two. It stopped working today though.

> Can't you fix it?

Not really. It's a giant hack. The author reverse-engineered the internals of
the Spotify client, and they recently updated their servers which makes it much
harder to figure out how it works inside.

> So, what now? Are you abandoning the idea?

No! I recently started a new project that I named [Hallon]() — Hallon because raspberries are
yummy. It allows a developer to use `libspotify` from Ruby, and I'm using it
to write a service layer that our Rails app can communicate with to build
playlists. I call that service layer Webhallon.

I still don't know C though, and the bindings between Ruby and libspotify has to
be written in C so I'm learning as I go.

> That's nice! Is it working well?

Haha. Yeah, sure. It builds playlists as it should.

> Why do I sense there's more?

It crashes every 11 to 15 seconds. Segfaults. It leaks memory too, so even if it
doesn't segfault it'll get killed by the system in less than a minute. Every so
often it deadlocks and hangs on boot. Told you I'm learning as I go!

> Ouch! So when is your deadline to release it?

Oh, it's been in production for the past week. It's already live. We have a
monitoring system that restarts it every 10 seconds. It almost never segfaults
before then, so we really utilize those 10 seconds!

> …

[Hallon]: https://github.com/Burgestrand/Hallon

## I quit working on the website, but Hallon remained

Hallon went through a lot phases, but random crashes were always an issue. As I
learned more things about Ruby and C the internals of Hallon would be shuffled
around.

Early 2011 I would move away from writing the extension in C by hand, to using
an approach using [Ruby FFI][]. This meant that I could describe what `libspotify`
looked like, and Ruby bindings to the raw C functions would be created for me.

Boilerplate was reduced by *a lot*, so I covered [_all_ of `libspotify` with
FFI][FFI-commit] in a single commit. FFI brought _many_ benefits to the project:

- No more dealing with makefiles and fighting the C extension build process.
- All error handling and data manipulation could now be done in Ruby.
- Garbage collection could be handled automatically by FFI, so memory leaks almost disappeared.
- There was no longer a need to do internal communication over sockets to deal with libspotify event callbacks.
  This was complex enough for me to [write a blog post about how to do it][LMFAO], figuring it out took months.

[FFI-commit]: https://github.com/Burgestrand/Hallon/commit/b49faef5f5aafde0131a2ce132fd3b6170f2a7ab
[Ruby FFI]: https://github.com/ffi/ffi/wiki
[LMFAO]: {% post_url 2011-06-15-asynchronous-callbacks-in-ruby-c-extensions %}

## FFI looks like Ruby, but really isn't Ruby

Alright, 100% libspotify API coverage means we're done, right? Not really. FFI
bindings give you a convenient way to call the C API, but you still have to deal
with pointers, manual memory management, and C calling conventions.

Extracting a string, for example a user's name, is cumbersome using just FFI:

```ruby
pointer = Spotify.link_create_from_string("spotify:user:…")
abort if pointer.null?
user_name_length = Spotify.link_as_string(pointer, nil, 0)
user_name = FFI::Buffer.alloc_out(user_name_length + 1) do |buffer|
  Spotify.link_as_string(pointer, buffer, buffer.size)
  return buffer.get_string(0)
end
```

This doesn't mean FFI is bad, far from it. Without FFI we'd have a few pages of
C code instead. We should still make the API sweeter to use, though. If only for
the sake of our sanity. Hallon's purpose was to add some sugar to the Spotify
API.

I split away the Spotify FFI bindings into it's own gem [v7.0.0 in April
2011][v7.0.0]. From here the Spotify gem was power, and Hallon was comfort.

[v7.0.0]: https://github.com/Burgestrand/spotify/releases/tag/v7.0.0

## Hallon was publicly announced June 2011

I wrote a [blog post explaining what Hallon was][Hallon announcement]. It's the
most recent blog post about the toolchain, not counting this one. A few people
used Hallon and reached out to me, much to my surprise, but it made me happy.

Most of the time they were writing their own music players. A DJ for the
office, a raspberry pi media pc, playing theme music every time they got home.

I also made my first lightning talk at Nordic Ruby for about 150 people,
essentially announcing Hallon. Live demo, so I streamed [Baba O'Riley][] through
the speakers using for-the-purpose-made OpenAL sound FFI bindings for Hallon. It
didn't segfault. It went well. It was scary, and fun.

[Baba O'Riley]: https://open.spotify.com/track/2KmEgiY8fQs0G6WNxtzQKr
[Hallon announcement]: {% post_url 2011-07-18-hallon-delicious-ruby-bindings-to-libspotify %}

## Abandoning Hallon in favor of FFI trickery

Eventually I realized that Hallon's model of trying to make the event-based
libspotify into a synchronous library was flawed. To use `libspotify` well you
_need_ to embrace their evented model. With this in mind I abandoned Hallon,
and turned my efforts into making the Spotify FFI bindings easier to use.

[Advanced FFI]: https://www.varvet.com/blog/advanced-topics-in-ruby-ffi/
