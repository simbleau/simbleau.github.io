---
layout: post
title: "Interactive vector graphics with Vello"
description: "Surveying the current state of art in vector graphic interactivity"
date: 2023-01-31
categories: rust graphics
---

## Interactivity?

What does this mean

## Current state of the art

- Rive
- LottieLab
- Lottiefiles
- Phase

### Case study: Spotify R&D

<https://engineering.atspotify.com/2024/01/exploring-the-animation-landscape-of-2023-wrapped/>

My immediate reaction is that Spotify didn’t go any further than suiting a business need. They seem to categorically dismiss Lottie for anything requiring interactivity, user preference, and customization. Their characterization of Lottie being best suited for generic animations is one I don’t particularly agree with. I think there’s appetite for more interactivity in Lottie files. As a runtime format, it could excel with runtime transformations due to its implicit encoding. The problem is that no code-first abstractions to Lottie exist. Nothing as simple as the PostScript API, since I saw that recently mentioned. Hence, Spotify used native animations to fill the gap where Lottie files were hard to customize either with code or tooling. I wish they explored the technology hole there.

### The battle of open and closed source

- Rive: closed source
- dotLottie: open source. Could use better documentation and tooling. Resetting the playhead?

## Implementation: An explosion of edge cases

Plenty of examples to go through here. It's *easy* to mess this stuff up.

## Demo

Demos with stress testing and benchmarks

### Future work

- Likely moving in the direction of switching bevy-vello to bevy-dotlottie since bevy-vello is growing in complexity