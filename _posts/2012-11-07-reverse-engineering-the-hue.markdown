---
layout: post
title: Reverse-engineering the Philips Hue
date: 07 November 2011 08:06:00 UTC
categories: [hacking, hue, philips-hue, ruby, home-automation]
slug:
  "Philips Hue":http://www.meethue.com/ became available last week.
  Essentially, it’s a system of LED lamps that support a wide range of colors
  and brightness, and remote control from within your home or the Philips Hue
  website. The lights fit into a regular E27 socket, and are extremely easy
  to set-up.

  Philips have said they’ll eventually release a developer kit, which would
  allow anybody with a computer to control their lights without using the app
  supplied by Philips. Personally, I don’t want to wait for that…
---

I’m not alone. A few others have also begun exploring how their Hue lights
really work, and have written articles about their progress:

- [Hack the Hue](http://rsmck.co.uk/hue)
- [A Day with Philips Hue](http://www.nerdblog.com/2012/10/a-day-with-philips-hue.html?showComment=1352172383498)

I’ve been meaning to document my findings more thoroughly than previously
available. As a result, a google group was born, [Philips Hue Hackers][], as
well as a GitHub repository named [Ruhue][] with a detailed explanation of the
known Hue API, as well as a Ruby library to allow for easy programmatic access
to your lightning system.

There are already quite a lot of API calls known, even more than what is
written in the Ruhue README. I plan to flesh out the current documentation with
all known calls, as well as examples for each, while at the same time fleshing
out the Ruby library for interacting with the lamps.

I’ll be writing another article soon, explaining how I went on about sniffing
the Hue traffic and some of the issues I ran into. Playing with the Hue has
been great fun so far!

[Philips Hue Hackers]: https://groups.google.com/forum/#!forum/hue-hackers
[Ruhue]: https://github.com/Burgestrand/ruhue
