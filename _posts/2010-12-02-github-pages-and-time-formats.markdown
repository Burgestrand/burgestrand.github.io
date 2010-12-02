---
layout: post
title: GitHub pages and time formats
date: 02 Dec 2010 01:10:12 UTC
categories: [hacks, github, annoying]
slug: I’ve been very happy since I moved my blog to GitHub pages, but I just found my first annoyance.
---

Every post I make have an associated date and time; I also give it a time zone so it can accurately be converted between different time zones. This means everybody, no matter where in the world they live, can see when my posts are published in their own local timezone if they so wish.

Figure my confusion, when my blog claimed my posts to be from nine hours ago even though I created it just now. This is not okay, and such I must fix it. But how?

Two wrongs *does* make one right
--------------------------
Liquid (GitHub pages template engine) actually parses the date given in the posts, so if I write `02 Dec 2010 00:10:12 +0000` it will be correctly parsed as UTC. There’s one problem with this: my pages are generated on GitHub in a PST time zone, and thus the [`date` filter used in my templates](https://github.com/tobi/liquid/wiki/Liquid-for-Designers) will display the date 9 hours *wrong*.

First thought: who cares? I’ll just display the dates verbatim instead of passing them through the filter. If I write them in UTC, it will display in UTC! Sounds good, right? *Wrong*, it doesn’t work. It appears the date is parsed even before I can get a hold of it, so even if I don’t throw it through the filter, it will display it in a PST time zone. Now what?

### Luckily Liquid is kind of stupid
What I need to do is to fool Liquid, making it think the given date is PST already. This could be done by adding 9 hours to every post date, but that’s ugly. My solution? Give the time zone using letters instead of digits: `02 Dec 2010 00:10:12 UTC`. For some reason Liquid can’t parse the time zone if given in letters. Sweet!

Now I can use the date formatter, as long as I don’t use `%Z` in my format string. That’s okay, I'll just write a literal UTC instead. This will potentially break in the future if they fix the parsing, giving me no choice but to add 9 hours to my posts (or remove time zone completely). But for now, it works and I am happy!