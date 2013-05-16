---
layout: post
title: Reverse-engineering the Philips Hue
date: 7 November 2011 07:06:00 UTC
categories: [hacking, hue, philips-hue, ruby, home-automation, lightning]
slug:
  The "Philips Hue":http://www.meethue.com/ became available last week.  It’s a
  system of LED lamps that support a wide range of colors and brightness, and
  allows complete remote control from the Hue mobile application, or the
  Philips Hue website. With the starter kit you’ll receive a Hue bridge, three
  lamps with an E27 socket, and it’s extremely easy to set up!


  Philips have said they’ll eventually release a developer kit, which would
  allow anybody with a computer to control their lights without using the app
  supplied by Philips. Personally, I don’t want to wait for that…
---

With the help of [tcpflow][] and the current [Hue iPhone-application][], I’ve managed to
capture parts of the Hue API. As a result of this, I’ve begun working on a complete Hue API
reference — available on GitHub at the [Ruhue][] repository.

A few notes about the Hue hub API:

- Hub discovery is done via [SSDP][]. This makes it very easy to find a Hue hub connected
  to your home network, without any manual setup necessary.
- Commands to the Hue are sent via HTTP, and allow full access to lamp color, brightness,
  and schedule. Data payloads are sent as JSON!

Further details can be found in the [Ruhue][] GitHub repository. It will be updated as the
API calls are explored. Pull requests are very welcome, so fork away and send patches and
I’ll accept them into the repository!

### Technical Hue hacking discussion is on the mailing list

In addition to the GitHub repository, there is also a Google Groups mailing
list, [Philips Hue Hackers][], a mailing list for anybody exploring their Hue
lights. Expect a lot of technical discussions on this mailing list.

The mailing list e-mail address is <mailto:hue-hackers@googlegroups.com>.

[tcpflow]: https://itunes.apple.com/us/app/philips-hue/id557206189?mt=8
[Hue iPhone-application]: https://itunes.apple.com/us/app/philips-hue/id557206189?mt=8
[SSDP]: http://en.wikipedia.org/wiki/Simple_Service_Discovery_Protocol
[Philips Hue Hackers]: https://groups.google.com/forum/#!forum/hue-hackers

### For you who are eager to start hacking the Hue…

The [Ruhue][] repository contains a console script, allowing you
to use an interactive ruby interpreter to control your Hue lights. Further details
are in the Ruhue README.

### Additional resources

A few others have begun exploring how their Hue lights really work as well, and
have written articles about their own progress. These articles existed before
I started the documentation effort, and they’ve helped immensely.

- [Hack the Hue](http://rsmck.co.uk/hue)
- [A Day with Philips Hue](http://www.nerdblog.com/2012/10/a-day-with-philips-hue.html?showComment=1352172383498)

[Ruhue]: https://github.com/Burgestrand/ruhue
