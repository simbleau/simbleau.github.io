---
layout: post
title: "Surveying the landscape of interactive vector graphics"
description: "Surveying the current state of art in vector graphic interactivity"
date: 2024-02-20
categories: rust graphics
riveWalkBlend: https://public.rive.app/community/runtime-files/7656-14740-walk-cycle-blend.riv
---

In this article, I aim to provide an overview of the current state of interactive vector graphics, particularly focusing on its significance and the ongoing developments in the field.

## Interactivity?

Interactivity in vector graphics refers to the capability of graphics to respond to user actions, such as hovering over a button and expecting a different visual appearance on hover.

Interactivity isn't just about making things pretty and playful; it's also integral for modern UI, games, and other applications. Vector graphics offer scalability and adaptability, making them highly desirable for designers and developers due to their resolution-independent nature.

### Key formats

This blog will mainly focus on comparing dotLottie and Rive files.

#### .lottie format

[dotLottie](https://dotlottie.io/) (extension `.lottie`) is a format that composes [Lottie JSON](https://en.wikipedia.org/wiki/Lottie_(file_format)) (extension `.json`) files and adds interactive capabilities. By itself, Lottie JSON files are only encoded vector animations; they offer no interactivity.

dotLottie and Lottie are [on GitHub](https://github.com/lottie/lottie-spec), and Lottie was [recently standardized under the Linux Foundation](https://www.linuxfoundation.org/press/announcing-lottie-animation-community). dotLottie as a format supports state machines, color swapping (themeing), and more. There isn't much magic under the carpet, a dotLottie file is simply a renamed ZIP archive with the following structure:

```text
.
├─ manifest.json
├─ animations/
│  ├─ animation-1.json
│  └─ animation-2.json
├─ states/
│  ├─ statemachine_0.json
│  └─ statemachine_1.json
├─ themes/
│  └─ theme1.lss
├─ images/
└─ audio/
```

#### .riv format

.riv files are exported by a bespoke editor on the [Rive](https://rive.app) platform. Unlike Lottie, Rive is closed-source commercial platform, and human editing of riv files is not possible.

Rive can have interactivity embedded throughout the file with inputs, constraints, nested animations, and state blending. It is quite intricate and something we will cover more in the following sections, because I think it's in a technical league far ahead of dotLottie.

{% include rive-frame.html src=page.riveWalkBlend %}

{% include rive-frame.html src=page.riveDemo %}

## Features

Here's my best stab at comparing the formats by feature:

| Feature             | .lottie (dotLottie)  | .riv (Rive)      |
|---------------------|----------------------|------------------|
| File Format         | Binary               | Binary           |
| Compression         | Zip ([DEFLATE](https://en.wikipedia.org/wiki/Deflate))        | Unknown          |
| File Size           | Small                | Compact          |
| Animation Support   | Yes                  | Yes              |
| Platform Support    | Wide range           | Wide range       |
| Interactivity       | Limited              | Advanced         |
| Playback Controls   | Limited              | Advanced         |
| Customization       | Limited              | Advanced         |
| Tooling             | Limited              | Advanced         |
| Documentation  | Good              | Good         |
| Cost                | Free                 | [Starts $24/user/month](https://rive.app/pricing) |
| Development         | [LAC](https://lottie.github.io/), open-source     | [Rive](https://rive.app)             |


### dotLottie (.lottie)

dotLottie (`.lottie`) is an open source format to provide *addititve interactivity* to Lottie through state maxchines.

dotLottie has identified several additive features to augment playback of a Lottie file:
- [Playback options](https://docs.lottiefiles.com/dotlottie-js-external/interactivity/playback-settings):

which adds state machines, playback settings, and color remapping (themes). However, the state machines only apply to adjust the entire image through playback options or switching the active Lottie JSON and colors.

### Rive file (.riv)

A Rive file `.riv` is a closed-source format with distinct embedded support for states everywhere in the file.

Other companies like Rive are enabling a deeper level of interactivity. Rive offers a bespoke editor which enables animators and designers to create interactive vector graphic experiences with state machines. State machines using Rive are much more advanced, allowing parts of the image to change with specific state machine variables. The editor is very good, runtime, and file format is compressed. Rive is doing so absolutely for rich profits and has no intention of contributing to any open-source entities.

I think we will soon benefit from more competition in the space, since Rive is operating as if they're a monopoly, recently raising their prices from $6/user to nearly 10x.

LottieLab is one such company I'm most optimistic for, with a Figma-like editor to create Lottie animations. They have also announced a reasonable interest in supporting interactivity similar to Rive, but with more emphasis in the web graphic space rather than deep interactive space, built more for games (Rive).

## Rive as a Northstar

![Walk cycle blend](/assets/walk-blend.png)

### Comparison / GAP analysis

TODO: Turn this into a feature table (either Supported, Planned, Not Supported)

TODO: use dotLottie public roadmap: https://dotlottie.io/roadmap/
Roadmap
    Audio layers support [soon]
    Video layers support [soon]
    Simplified expressions language
    General scripting engine
    Mouse and touch interactivity support
    Keyboard interactivity support
    Sensors (compass, acceleration) interactivity support
    Haptics support
    Network access support
    3D support [long-term]

dotLottie:
- State machine
  - Each state has:
    - Playback settings
    - Transitions
    - Inputs (in the form of transitions)

Rive:

- States
  - A timeline animation that can play anytime
  - Default states:
    - Entry State
    - Exit State
    - Any State
  - Animation states:
    - Single animation state
    - Blend states
- Inputs
  - Number
  - Boolean
  - Trigger
- Transitions
  - Duration (onAfter)
  - Exit Time (necessary time to play before transitioning)
  - Conditions
    - Use inputs to define conditions
  - Interpolation between states
- Listeners (User actions)
  - Allows Click, hover, mouse move actions to change inputs without a developer
  - Consists of:
    - Target: Where to listen for user actions
    - User actions
      - Pointer Enter, Exit, Down, Up, Move
    - Listener action
      - Input change: Change an input directly
      - Align target: Align a target with the mouse cursor
- Layers
  - Nested animations in the file

### Companies

Big 3:

- [LottieLab](https://lottielab.com)
  - Solely focused and building a business around creating Lottie animations
  - Similar to Figma
- [Rive](https://rive.app)
  - Focused on building a private walled garden around the .riv format, runtime, and editor, for the sake of high-end design.
- [Lottiefiles](https://lottiefiles.com)
  - Lots of open source tooling, and they have a lottie creator coming down the pipe and in beta right now.

Up and coming:

- [Phase](https://phase.com)
  - A vector design tool with limited public information. Their progress is not public. I was their 2nd onboard for a demo but it was quite impressive and had comparable feature to Lottiefiles and LottieLab.
- [Jitter](https://jitter.video/)
  - More focused on exporting to Video, but they now have a Beta export to Lottie, making them a player

### Case study: Spotify R&D

Spotify has recently made some exploration with a business use-case. Every year Spotify provides a summary to each user of their most listened to and favorite artists and albums, called "Spotify Wrapped", which has become popular to millions of people each year.

They wrote about this change in an engineering blog: <https://engineering.atspotify.com/2024/01/exploring-the-animation-landscape-of-2023-wrapped/>

Spotify said, and quote:

> "Spotify \[embraced\] a Lottie-first principle whenever practical. Instead of having multiple engineers individually create the same animation for each platform, motion designers create a single animation file that is served to all platforms. Additionally, Lottie provides visual parity across all platforms and has removed the need for engineers to investigate technical feasibility, which typically requires the efforts of multiple engineers over several days. With Lottie, we’re able to paint a cohesive experience for our users while allowing our engineering team to focus on other essential tasks."

I support the idea, generally, that it makes business sense to put designers first in creating interactive experiences, rather than programmers. It is more in designer domain and could save time and money.

However, I did somewhat disagree to one of their sailing points:

> "Lottie especially shines when utilized for generic animations."

My immediate reaction is that Spotify didn’t go any further than suiting a business need. They seem to categorically dismiss Lottie for anything requiring interactivity, user preference, and customization. Their characterization of Lottie being best suited for generic animations is one I don’t particularly agree with. I think there’s appetite for more interactivity in Lottie files. As a runtime format, it could excel with runtime transformations due to its implicit encoding. The problem is that no code-first abstractions to Lottie exist. Nothing as simple as the PostScript API, since I saw that recently mentioned. Hence, Spotify used native animations to fill the gap where Lottie files were hard to customize either with code or tooling. I wish they explored the technology hole there.

## Implementing Interactivity

When I wrote the [*High performance vector graphic video games*](https://simbleau.github.io/rust/graphics/2023/11/20/using-vello-for-video-games.html) blog, you may have noticed the Vong game didn't actually use lottie. The egg, bacon, and scene was entirely SVG, and were simply manipulated using transforms by the game loop.

The next page for video games is naturally to ask how to design a game with real interactivity. This question has far reaching effects - creating UI buttons, animating a health bar, playing a walk animation with state machines, and more. Getting interactivity right enough for a video game requires a lot of forethought and finesse.

I notice the ecosystem around vector graphics has somewhat fragmented, with Lottiefile's attempts with dotLottie, and Rive's own proprietary format. I don't want to contribute with - yet another - fragmented direction.

### Players

Need to write a lot about this here. Players are a code-first abstraction to running state machines.
Compare this to Rive.

### dotLottie

I decided to look heavier into dotLottie for the simple fact that they were the first to pioneer an open-source extension to Lottie, for the purpose of extending interactivity.

Lottiefiles [offered good documentation](https://docs.lottiefiles.com/dotlottie-js-external/interactivity), and players for React, Vue, Svelte, etc.

It seemed to be something that was getting a fair bit of investment, and most of the heavy thought was done for me.

"What events and triggers should exist for state machines?, e.g. onMouseEnter, onClick, onAfter, onComplete"
"How should we encode a file with interactivity? e.g. the .lottie file"
"What features should be possible for a state? e.g. playback options, themeing"

They laid out pretty good documentation on the format and what is possible with it.

#### Playback settings

Nothing missing here

#### Transitions

- Missing key events
- Missing state.resetPlayheadOnEnter
- Missing state.resetPlayheadOnExit

#### Themes

About a year ago, I developed a proof of concept to add runtime color swapping to Lottie animations.

![Color swap editor](/assets/color-editor.png)

The solution here was to create a "Color Pallette," or a list of colors. Then, color remapping could be applied to layers by their shape index. In other words, the layer `suckers`, shape `3` would be remapped to the color named `suckers-green` under a Color Pallette.

dotLottie, on the other hand, takes a more inventive approach to themeing.

They have invented [LSS](https://lottiefiles.github.io/lottie-styler/) - (of which, I can only assume stands for "Lottie Style Sheets") - which is a DSL to remap aspects of the lottie elements, such as strokes and fills. My implementation was rudimentary in hindsight, only focusing on remapping fills by layer and shape index.

*Edit*: The documentation site is busted. See [#38](https://github.com/LottieFiles/lottie-styler/issues/38) for details. If this gets fixed, please submit a PR to remove this line! :)

### Implementating dotLottie support with Vello

I thought it would make more sense to experiment implementing the dotLottie spec within my crate, [bevy-vello](https://github.com/vectorgameexperts/bevy-vello).

This would drive things in a more open direction, and keep the ecosystem from fragmenting. It would also give me a chance to try the format for video games, and see what obstacles I would run in to.

### Challenges

- What internal state would be necessary? Took many revisions
- ECS
- LSS macros are too complex right now

## Demo

TODO: Demos with stress testing and benchmark here

### Benchmarks

Vello primitives

SVG vs. Lottie state machines

### Lessons

- I don't need the bells and whistles of Rive, even for games.
- More editor support for dotLottie interactivity is desperately needed, but one that isn't Rive.

### Future work

- Implementing a parser for .lottie files
- Need for better compression of lottie .json and .lottie files
- More transitions (onKeyDown)
- More options (state.resetPlayheadOnEnter, state.resetPlayheadOnExit)
- Ergonomic API for manipulating Lottie
- Introduce a runtime format optimized for interactivity. Likely moving in the direction of switching bevy-vello to bevy-dotlottie since bevy-vello is growing in complexity
- LSS macros
- Optimization
- A code-first library for lottie manipulation. Would be helpful for particle generators, etc. - DrawBot
- Nesting lotties/dotLotties in eachother
- It would be cool to see Themeing and LSS output in LottieLab