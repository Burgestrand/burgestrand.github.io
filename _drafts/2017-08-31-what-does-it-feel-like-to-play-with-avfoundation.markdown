---
title: Adventures in AVFoundation
date: 2017-08-31T11:00:00+0100
categories: [swift, ios, avfoundation]
---

Short article with most common pitfalls.

- We want a player, with a custom UI for play and pause.
- Things I tried that did not work very well:
  * Naive loading of AVPlayer with a URL, or an AVPlayeritem.
  * Listening to all the events and deriving current player state to reflect in
    the UI.
  * Loading the AVQueuePlayer with our entire playlist in one go.
  * Keeping track of which AVPlayerItem belongs to our T by mapping.
  * KVO for preloaded asset and AVPlayerItem behaves weirdly (e.g. duration).
- My current solution:
  * Create an AVAsset for each T and keep it around as long as you want, barring
    resource constraints.
  * _Never_ create an AVPlayerItem for an AVAsset that has not preloaded
    `tracks` and `duration`.
  * Try to keep AVQueuePlayer.items() synchronized with Playlist.prefix(3)
    through diffing.
  * Do not use auto-advance in AVQueuePlayer and listen to changes, instead listen
    to events and explicitly do `player.next()`, makes it way easier to keep
    track of what's currently playing.
