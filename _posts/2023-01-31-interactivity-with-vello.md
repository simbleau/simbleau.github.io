---
layout: post
title: "Surveying interactive vector graphic formats"
description: "Surveying the current state of art in vector graphic interactivity, focusing on dotLottie and Rive"
date: 2024-02-20
categories: rust graphics
riveDemo: https://cdn.rive.app/animations/vehicles.riv
riveWalkBlend: https://public.rive.app/community/runtime-files/7656-14740-walk-cycle-blend.riv
riveCustomize: https://public.rive.app/community/runtime-files/8463-16227-customize-your-avatar.riv
riveTheme: https://public.rive.app/community/runtime-files/8771-16784-darklight-mode-switch.riv
riveStateMachine: https://public.rive.app/community/runtime-files/6413-12421-lil-guy.riv
lottieRocket: /assets/rocket.json
lottieBlueRocket: /assets/rocket-blue.json
---

In this article, I aim to provide an overview of the current state of interactive vector graphics, particularly focusing on its significance and the ongoing developments in the field. There will be interest specifically given to Lottie, dotLottie, and Rive.

## Defining "Interactivity"

Interactivity isn't just about making things pretty and playful, it is a way to transform the user experience. This functionality is essential for modern user interfaces, which strive to be as intuitive as they are visually appealing.

## Re: Spotify Wrapped 2023

Every year Spotify provides a summary to each user of their yearly content consumption, in a flashy animated format, customized to each user. It's like a parade for your tax documents, if your audio needed that.

They explored [Lottie JSON](https://en.wikipedia.org/wiki/Lottie_(file_format)) (extension `.json`) as a vector animation format to improve their adored tradition, "[Spotify Wrapped](https://newsroom.spotify.com/2023-wrapped/)," and published findings on their public engineering blog, titled *"[Exploring the Animation Landscape of 2023 Wrapped](https://engineering.atspotify.com/2024/01/exploring-the-animation-landscape-of-2023-wrapped/)."*

> "Spotify \[embraced\] a Lottie-first principle whenever practical. Instead of having multiple engineers individually create the same animation for each platform, motion designers create a single animation file that is served to all platforms. Additionally, Lottie provides visual parity across all platforms and has removed the need for engineers to investigate technical feasibility, which typically requires the efforts of multiple engineers over several days. With Lottie, we’re able to paint a cohesive experience for our users while allowing our engineering team to focus on other essential tasks."

Before Lottie, Spotify used engineers to write native animations in code to tailor the experience to the user. These native animations were customized, transformative experiences... and slow to create. Lottie animations offered a workflow requiring no engineers (likely motivated by time or cost).

|Lottie|Native|
|:----:|:----:|
| ![Lottie](https://storage.googleapis.com/production-eng/1/2024/01/Lottie-genre-animation.gif) | ![Native](https://storage.googleapis.com/production-eng/1/2024/01/Native-Animation_Genre.gif) |

After reading their engineering blog, one statement didn't sit well with me:

> "Lottie especially shines when utilized for generic animations."

Spotify categorically dismissed Lottie for situations requiring interactivity or customization. Native (coded) animations were still developed in these cases. I found myself hungry (not just for sandwiches), wanting to hear more about *why*! That sentiment leaves room for investigation: Just how much interactive potential does the Lottie format hold, and what could push its boundaries further?

## Unpacking the dotLottie (.lottie) format

My first exploration of this topic led to *dotLottie*, a [Linux Foundation standard](https://www.linuxfoundation.org/press/announcing-lottie-animation-community) and [open source](https://github.com/lottie/lottie-spec) capability *over* Lottie.

Surprisingly, [dotLottie](https://dotlottie.io/) (extension `.lottie`), isn't mentioned in Spotify's study.

Lottie, by itself, is only a single vector animation with no interactivity. dotLottie adds capability with support for multiple (Lottie) animations, playback settings, state machines, and color swapping (themeing). There isn't much magic under the carpet, as a dotLottie file is simply a renamed ZIP archive with the following structure:

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

### State machines

Representing a dotLottie state in Rust code would look something like this:

```rust
struct State {
    /// The ID of this state
    id: String,
    /// The Lottie file to play
    asset: LottieFile
    /// Optional: A theme (color pallete)
    theme: Option<Theme>,
    /// Optional: Playback settings
    options: Option<PlaybackOptions>,
    /// Optional: A list of conditions for exiting this state.
    transitions: Vec<Transition>,
}
```

A dotLottie runtime player then picks up the state machine and automates the transitions. The transitions dotLottie currently supports are:

- onComplete : Transitions after the current animation finishes
- onAfter: Transitions after a number of seconds
- onShow: Transitions on the first visible frame
- onMouseEnter: Transitions when a mouse enters the animation bounding box
- onMouseLeave: Transitions when a mouse exits the animation bounding box
- onMouseClick: Transitions when a mouse clicks inside the animation bounding box

My initial reaction was that dotLottie transitions seem *very* limited and biased towards desktop users, however, I found that most common platform-specific transitions like mobile gestures are planned on [the dotLottie roadmap](https://dotlottie.io/roadmap/). Touch gestures, keybaord events, etc. are coming *"soon."*

For consistency, I've enumerated these in code as well:

```rust
enum Transition {
    /// Transitions to the given state after a period of seconds.
    OnAfter { state: String, after: Duration },
    /// Transition to the given state after the animation finishes.
    OnComplete { state: String },
    /// Transition to the given state when the mouse enters the image bounding box.
    OnMouseEnter { state: String },
    /// Transition to the given state when the mouse clicks inside the image bounding box.
    OnMouseClick { state: String },
    /// Transition to the given state when the mouse exits the image bounding box.
    OnMouseLeave { state: String },
}
```

### Playback options

Playback options, present in states, allow augmenting playback for the current state's asset (Lottie file).

The supported options feel good enough for most users:

```rust
struct PlaybackOptions {
    /// Whether to automatically start the animation on state change.
    autoplay: bool,
    /// The direction of the animation.
    direction: Direction, // -1 for reverse, +1 for normal.
    /// The speed of the animation as a multiplier.
    /// 1.0 is normal speed, < 1.0 is slower, > 1.0 is faster.
    speed: f32,
    /// A duration of time spent idle between loops.
    intermission: Duration,
    /// Whether to reset the playhead every loop (normal) or to reverse
    /// directions (bounce).
    play_mode: PlayMode,
    /// Whether to loop, and how many loops.
    /// true = infinite loop, false = do not loop, or a number of loops.
    looping: LoopBehavior,
    /// The segments (frames) of the animation to play.
    segments: Range<f64>,
}
```

### Themes

Taking a step back, about a year ago, I worked on a proof of concept to add color swapping to Lottie animations for game development. The solution I explored was creating a list of colors, which could be applied by a selector of layer name and shape index.

![Color swap editor](/assets/color-editor.png)

dotLottie, on the other hand, created a bespoke CSS-inspired language called [LSS](https://lottiefiles.github.io/lottie-styler/), "Lottie Style Sheets" \[[Usage](https://developers.lottiefiles.com/docs/dotlottie-js/theming/)\].

This really feels like a thoughtful innovation for themeing Lottie files, with the ability to re-imagine strokes and fills for Lottie files. Exactly like CSS, it features a selector-based language with properties that can be changed.

```css
GradientStrokeShape {
    stroke-color: radial-gradient(yellow 20%, green 80%);
}
```

It has always been possible to change Lottie colors if you have the know-how and a text editor. For example, here is a fragment of a Lottie file, with the color `k` ({% include highlight.html background='#ed6a65' foreground='#fff' text='#a02422' %}) represented in decimal:

```json
{"ty":"fl","c":{"a":0,"k":[0.627,0.263,0.133],"ix":2}}
```

By changing the text, I can (tediously) create a *"[Blue Origin](https://www.blueorigin.com/)"* themed rocket, recolored from Google Noto's animated emojis. In this case, I used <https://lottielab.com> to do the job quicker.

| Normal | Modified |
|:------:|:---------:|
| ![Normal Lottie](/assets/rocket.gif) | ![Modified Lottie](/assets/rocket-blue.gif) |

*Attribution: '[Rocket](https://googlefonts.github.io/noto-emoji-animation/?selected=Animated%20Emoji%3Aemoji_u1f680%3A)', by Google Fonts*. Modified by Spencer C. Imbleau using [LottieLab](https://lottielab.com).

I think the LSS format is quite good, and makes this easier than plain text editing. There may be a market need for good dotLottie tooling here.

## The elephant in the room: Rive

Rive offers a proprietary asset format that speaks to a higher degree of interactivity through embedded support for states and triggers. The design-first philosophy of Rive's approach provides a rich set of features for intricate and responsive animations.

Rive has all the staples of dotLottie, such as state machines and transitions, but they offer a few extra unique features I want to explore in the following sections.

{% include rive-frame.html id='demo' src=page.riveDemo statemachine='bumpy' %}
*Attribution: 'Vehicles' example from [Rive](https://rive.app)*

### State machines with state blending

Rive has advanced support for state machines, even offering visual blueprinting for state machines to users.

![State machine blueprinting](/assets/rive-blueprinting.png)


But something uniquely "Rive" is state blending - the `.riv` format allows two states to be blended seamlessly. Here's an example showcasing the capability:

{% include rive-frame.html id='walk' src=page.riveWalkBlend statemachine='State Machine 1'%}
*Attribution: ['Walk Cycle Blend'](https://rive.app/community/7656-14740-walk-cycle-blend/) example from [@whereisross](https://rive.app/@whereisross/), CC By 4.0*

Under the hood, this is a linear easing from one state to the next, and feels full of potential for user experience. This feature remains unparalleled in dotLottie.

![Walk cycle blend](/assets/walk-blend.png)

### Inputs

In addition to state blending, Rive has inputs. These are used by a [Rive runtime](https://rive.app/runtimes) to change what's displayed to the user. Currently supported inputs are:

- Numbers
- Booleans
- Triggers (e.g. Mouse click)

These triggers can also be constrained to specific regions of a Rive artboard, and enables designers to create complex interfaces:

{% include rive-frame.html id='customize' src=page.riveCustomize statemachine='State Machine 1'%}
*Attribution: ['Customize your Avatar'](https://rive.app/community/8463-16227-customize-your-avatar/) example from [@nikkieke001](https://rive.app/@nikkieke001/), CC By 4.0*

Moreover, these inputs can be used to drive playback options with fine-grained control to specific elements. You can also imagine a dotLottie theme modelled by inputs, illustrated in the following example:

{% include rive-frame.html id='theme' src=page.riveTheme statemachine='Button_Animation'%}
*Attribution: ['Dark/Light mode switch'](https://rive.app/community/8771-16784-darklight-mode-switch/) example from [@lucasbazarinr](https://rive.app/@lucasbazarinr/), CC By 4.0*

### Nested Artboards

Finally, I draw specific attention to "Artboards" in Rive. The term "Artboard" refers to an entire animation (e.g. a plain Lottie file). Rive makes it intuitive to copy, nest, and overlay Artboards independently. This is used in the walk-cycle blend we've seen to give the character a white shadow; this used a duplicated Artboard with white overlay, remixed back into a combined Artboard.

![Nested Artboards](/assets/rive-artboards.png)

## The GAP

Here's my best stab at consolidating the diffs between formats:

| Feature             | .lottie (dotLottie)  | .riv (Rive)          |
|---------------------|----------------------|----------------------|
| File Format         | Open source Zip      | Proprietary binary   |
| State machines      | Supported            | Advanced             |
| Playback control    | Sequence-only        | Complex control      |
| Interactivity       | Basic                | Advanced             |
| Themeing            | Supported            | Possible with Inputs |
| Tooling             | Limited              | Advanced             |
| Documentation       | Good                 | Good                 |
| Cost                | Free                 | Subscription-based   |

### Future opportunity

Need to write a lot about this here. Players are a code-first abstraction to running state machines.
Compare this to Rive.

TLDR: Although I consider dotLottie a great first step towards interactivity, it will take some work to make it comparable to Rive.

I am hopeful with LAC forming.

Areas of improvement:

- More transitions
- Missing state.resetPlayheadOnEnter
- Missing state.resetPlayheadOnExit
- Player customization
- Blend modes
- Nested artboards
- Inputs
- More performant renderer (vello)

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
- More playback options (state.resetPlayheadOnEnter, state.resetPlayheadOnExit)
- Ergonomic API for manipulating Lottie
- LSS macros
- A code-first library for lottie manipulation. Would be helpful for particle generators, etc. - DrawBot
- Nesting animations (lottie)
- Competition (LottieLab)
- Make transitions more open-ended, allowing platform or custom transitions



There seems to be a technology gap in creating and manipulating lottie files with code. There doesn't seem to be something quite as simple as the [PostScript API](https://en.wikipedia.org/wiki/Encapsulated_PostScript) for manipulating Lottie files.

```eps
% Example PostScript (EPS) code
 newpath
  63 153 moveto
 549 153 lineto
 stroke
```

It's not easy to change a Lottie file with code, but that's where I find myself wanting to research the most. With some good effort put in a runtime, it would be possible to serve many audiences. Reducing engineer hours, "code artists," (using code as a base for creating digital art), and engineers wanting to generate or modify vector animations.

Some examples would include vector particle generators, procedural fire, and the native animations Spotify manually created (e.g. adjusting sandwich layer height and text).