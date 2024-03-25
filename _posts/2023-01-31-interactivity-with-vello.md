---
layout: post
title: "Surveying the landscape of interactive vector graphics"
description: "Surveying the current state of art in vector graphic interactivity, focusing on dotLottie and Rive"
date: 2024-02-20
categories: rust graphics
riveDemo: https://cdn.rive.app/animations/vehicles.riv
riveWalkBlend: https://public.rive.app/community/runtime-files/7656-14740-walk-cycle-blend.riv
riveCustomize: https://public.rive.app/community/runtime-files/8463-16227-customize-your-avatar.riv
riveTheme: https://public.rive.app/community/runtime-files/8771-16784-darklight-mode-switch.riv
---

In today's tech-driven landscape, interactive vector graphics are shaping the way we experience digital interfaces across applications. This article is a deep dive into the evolution and potential of this dynamic medium, with a spotlight on dotLottie and Rive. Interactivity isn't just an aesthetic touch — it's fundamental to engaging user experiences in modern UI, gaming, and more.

## Interactivity?

Interactivity isn't just about making things pretty and playful, it is a way to transform the user experience. This functionality is essential for modern user interfaces, which strive to be as intuitive as they are visually appealing.

### Study: Spotify R&D

Spotify recently explored [Lottie JSON](https://en.wikipedia.org/wiki/Lottie_(file_format)) (extension `.json`), as a vector animation format to improve their adored tradition, "[Spotify Wrapped](https://newsroom.spotify.com/2023-wrapped/)." Every year Spotify provides a summary to each user of their yearly content consumption, in a flashy animated format, customized to each user. It's like a parade for your tax documents, if your audio needed that.

They wrote about this change in an [engineering blog](https://engineering.atspotify.com/2024/01/exploring-the-animation-landscape-of-2023-wrapped/):

> "Spotify \[embraced\] a Lottie-first principle whenever practical. Instead of having multiple engineers individually create the same animation for each platform, motion designers create a single animation file that is served to all platforms. Additionally, Lottie provides visual parity across all platforms and has removed the need for engineers to investigate technical feasibility, which typically requires the efforts of multiple engineers over several days. With Lottie, we’re able to paint a cohesive experience for our users while allowing our engineering team to focus on other essential tasks."

Before Lottie, Spotify used programmers to write native animations in code to tailor the experience and animations to the user. These native animations were rich, transformative experiences with interactivity, but they were slow to create.

Because vector graphics offer a resolution-independent imaging model, it made sense to pursue tooling that could alleviate the burden on programmers and save costs with animators. Transformative content is hard to code, and I wholeheartedly applaud their inventive spirit and bias for action here.

One statement, however, I questioned:

> "Lottie especially shines when utilized for generic animations."

Lottie was categorically dismissed for cases requiring interactivity, user preference, and customization. Native (coded) animations were continued in these cases, and little explanation was given on *why*.

That sentiment leaves room for investigation: Just how much interactive potential does the Lottie format hold, and what could push its boundaries further?

## Unpacking the dotLottie (.lottie) format

The [dotLottie](https://dotlottie.io/) (extension `.lottie`), which isn't mentioned in Spotify's study, is a format that composes [Lottie JSON](https://en.wikipedia.org/wiki/Lottie_(file_format)) (extension `.json`) files and adds interactive capabilities. Lottie, by itself, is only encoded vector animations; it offers no interactivity.

dotLottie and Lottie are [on GitHub](https://github.com/lottie/lottie-spec), and Lottie was [recently standardized under the Linux Foundation](https://www.linuxfoundation.org/press/announcing-lottie-animation-community). I tend to think of dotLottie as the first attempt to standardize capability *over* Lottie.

dotLottie as a format supports state machines, color swapping (themeing), and more. There isn't much magic under the carpet, a dotLottie file is simply a renamed ZIP archive with the following structure:

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

dotLottie focuses on state machines, a manifest of states and transitions. A state alone is a combination of an asset (Lottie file), playback options, a theme, and transitions.

Representing this in Rust code would look like this:

```rust
struct State {
    /// The ID of this state
    id: String,
    /// The Lottie file
    asset: LottieFile
    /// Optional: A theme (color pallete)
    theme: Option<Theme>,
    /// Optional: Playback settings
    options: Option<PlaybackOptions>,
    /// Optional: A list of conditions for exiting this state.
    transitions: Vec<Transition>,
}
```

There can be any number of transitions. Right now the format enumerates only a few options. I've enumerated these in code for consistentcy:

```rust
enum Transition {
    /// Transitions to the given state after a period of seconds.
    OnAfter { state: String, dur: Duration },
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

My initial reaction was that dotLottie transitions seem *very* limited and biased towards desktop users, however, I found that most common platform-specific transitions like mobile gestures are planned on [the dotLottie roadmap](https://dotlottie.io/roadmap/).

### Playback options

Playback options, when composed in states, allow augmenting playback for the current state's asset (Lottie file).

The settings cover a broad range of augments a typical user would want:

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

Taking a step back, about a year ago, I worked on a proof of concept to add color swapping to Lottie animations for game development.

![Color swap editor](/assets/color-editor.png)

The solution explored was to create a "Color Pallette," or a list of colors. Then, color remapping could be applied to a selector by layer name and shape index.

dotLottie, on the other hand, [innovated heavily](https://developers.lottiefiles.com/docs/dotlottie-js/theming/) with a CSS-inspired DSL called [LSS](https://lottiefiles.github.io/lottie-styler/) ("Lottie Style Sheets").

This really feels like a thoughful execution of themeing Lottie files, with the ability to re-imagine strokes and fills for Lottie files. Exactly like CSS, it features a selector-based language with properties that can be changed.

```css
GradientStrokeShape {
    stroke-color: radial-gradient(yellow 20%, green 80%);
}
```

*Edit*: The documentation site is busted. See [#38](https://github.com/LottieFiles/lottie-styler/issues/38) for details. If this gets fixed, please submit a PR to remove this line! :)

## The elephant in the room: Rive

Rive offers a proprietary asset format that speaks to a higher degree of interactivity through embedded support for states and triggers. The design-first philosophy of Rive's approach provides a rich set of features for intricate and responsive animations.

Rive has all the staples of dotLottie, such as state machines and transitions, but they offer a few extra unique features I want to explore in the following sections.

{% include rive-frame.html id='demo' src=page.riveDemo statemachine='bumpy' %}
*Attribution: 'Vehicles' example from [Rive](https://rive.app)*

### State blending

Rive has great support for state machines, even offering visual blueprinting for state machines to users.

![State machine blueprinting](/assets/rive-blueprinting.png)

But something uniquely "Rive" is state blending - the `.riv` format allows two states to be blended seamlessly. Here's an example showcasing the capability:

{% include rive-frame.html id='walk' src=page.riveWalkBlend statemachine='State Machine 1'%}
*Attribution: ['Walk Cycle Blend'](https://rive.app/community/7656-14740-walk-cycle-blend/) example from [@whereisross](https://rive.app/@whereisross/), CC By 4.0*

This is a simple linear easing from one state to the next, and feels full of potential for user experience. This feature remains unparalleled in dotLottie.

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