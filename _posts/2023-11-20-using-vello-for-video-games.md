---
layout: post
title: "High performance vector graphic video games"
description: "Lessons from making the first modern compute-centric vector graphic game for web"
date: 2023-11-20
modified: 2024-01-15
categories: rust graphics
youtubeId: hNu5oF18j5g
vongDemo: https://simbleau.github.io/vong/
---

For a few years I've been trying to solve a hard problem: *"How can I use vector graphics as the backing image model in realtime systems?"*

Yes, the image model that uses points, lines, and equations to encode image data. Of which, several common encoding formats exist, such as like SVG and PDF.

![svg](/assets/Bitmap_VS_SVG.svg)

The internet has done this once meaningfully in the past. In the 2000s, a herculean effort to optimize the performance of interactive vector graphics brought an advent of vector games through Adobe's Shockwave Flash player. But in the wake of SWF's deprecation from major browsers, there wasn't a worthy successor, and the era of web games died off. Unfortunately, efforts like HTML5 and WebGL were never able to replace what was lost.

Hence, I believe the first games to understand and successfully use this image model under the constraints of major browsers could benefit from content-delivery savings, resolution independence, and offer novel applications of interactivity, physics, and non-destructive transformations.

## Onto the papercuts

Vector images are notoriously unfit for modern GPU architecture because of an inherent locality issue. In contrast to raster graphics where color and fill information is explicitly encoded, rendering a pixel in a vector image requires knowledge of the entire image. Efficiently solving the fill of every pixel in realtime poses a unique challenge. With the rising accessibility of compute kernels and low-level GPU architecture access over the past few years, especially from projects like [wgpu](https://wgpu.rs/), friction with general purpose GPU computing is fading. Projects like [vello](https://github.com/linebender/vello) are pioneering the 2d vector graphics space, and furthering the experimental research inspired by projects like [pathfinder](https://github.com/servo/pathfinder).

The high-level strategy used by these new hardware-accelerated renderers is to create a [compute-centric pipeline with a sort-middle architecture](https://raphlinus.github.io/rust/graphics/gpu/2020/06/12/sort-middle.html) to parallelize the problem space efficiently. This is a technical leap in recent years and its efficiency will be hard to challenge.

## Vong

![Vong](/assets/vong.png)

The technical leap was implemented in [vello](https://github.com/linebender/vello) (*formerly piet-gpu*), started by [Raph Levien](https://levien.com/) and my colleagues in [the linebender community](https://linebender.org). The Vello API depends on WebGPU to provide an abstraction layer to targets, while offering more accessibility to low-level hardware like compute shaders. It's for these reasons rust, webgpu, and vello would make a perfect combination for cross-platform vector games with one code-base.

![Vong](/assets/webgpu.svg)

This spurred on my race to prove the lost benefits are still relevant. In a sudden case of reviving-the-horse syndrome, I worked with a few colleagues from my previous tenure at NASA to understanding the unknowns of why I've never seen a modern [Kurzgesagt](https://www.behance.net/kurzgesagt) vector game. Hence, the decision to start by creating the classic game of Pong was deliberate, providing a foundational exploration of Vello's capabilities in rendering vector graphics dynamically. It felt appropriate to call the project *Vong*, joining "Vector" and "Pong".

Matching webgpu, vello, and rust's strengths, [bevy](https://bevyengine.org/) was the obvious choice for a cross-platform open source game engine. It also uses webgpu, and was developed open-source and code-first.

### Rendering integration

Standing in the way of seeing my first [Ghostscript Tiger](https://commons.wikimedia.org/wiki/File:Ghostscript_tiger_(original_background).svg) (the *Hello, World!* of vector graphics) was an integration to render vector assets in bevy with vello. Sebastian Hamel ([@seabassjh](https://github.com/seabassjh)) and I ([@simbleau](https://github.com/simbleau)) developed that integration in the open, [`bevy-vello`](https://github.com/vectorgameexperts/bevy-vello). Just like any other ordinary asset in bevy, such as a *png*, there are no surprises:

```rust
// Bevy 0.12
fn setup_assets(
    mut commands: Commands,
    asset_server: ResMut<AssetServer>
) {
    // Load image of egg
    commands.spawn(
        VelloVectorBundle {
            vector: asset_server.load("egg.svg"),
            origin: bevy_vello::Origin::Center,
            transform: Transform::from_scale(Vec3::splat(0.1)),
            ..Default::default()
        }
    )
}
```

Architecturally, `bevy-vello` was built with two backends. The first is an `.svg` ETL library called [`vello-svg`](https://github.com/vectorgameexperts/vello-svg) which loads the SVG using the vello `SceneBuilder` API. The second is [`vellottie`](https://github.com/vectorgameexperts/vellottie), a parser and runtime for [Airbnb's `.json` lottie](https://airbnb.io/lottie/) format, and a rewrite of [Chad Brokaw's lottie runtime, "velato"](https://github.com/linebender/velato).

{% include youtubePlayer.html id=page.youtubeId %}

Strategically, it made sense to pursue both formats, but a painful lesson learned was that the `.svg` and `.json` lottie specifications are huge, with plenty of feature flags that aren't supported by vello. While writing a parser wasn't too challenging using the `serde` parsing library, dealing with animation inconsistencies and rendering artifacts remains a difficult task. Currently, vello is writing CPU shaders to help triage these artifacts. But for the future, I'm inclined to support only lottie formats, since any SVG *could* be reformed into a 0 frame-rate lottie animation. One of the biggest obstacles in commercial viability is artist tooling, so I'm keeping an eye on companies like [LottieLab](https://lottielab.com), [Phase](https://phase.com), and [Rive](https://rive.app).

### Physics

Once rendering was solved, another thorny issue with pong was physics: collision detection between arbitrarily curved shapes is *really hard*. While some algorithms exist, none today are obviously good.

Eventually I settled on tessellation as a decent proxy for physics. The technique is quite simple in concept.

First, curves as flattened into line segments with configurable accuracy (aka *"tolerance"*).

![Tessellation](/assets/Flattening.svg)

Then, vertices are paired to generate a triangle mesh. The resulting triangle mesh is used to derive a [convex hull](https://en.wikipedia.org/wiki/Convex_hull_algorithms) hitbox.

![Tessellation](/assets/Tessellation.svg)

This is obviously not true collision detection between arbitrary curves with infinitessimal precision, but was rather a compromise given a lack of algorithmically efficient collision detection between curves.

As with most things in game development, this is a hack, but a good one! Even when using a liberal tolerance for curve flattening, physics yield a high level of collision fidelity, nearly indistinguishable from the true underlying curves. Granted, this only works great since my egg is convex in nature.

![Vong Closeup](/assets/vong-closeup.png)

The [`lyon`](https://github.com/nical/lyon) library was used for tessellation and [`bevy_rapier`](https://rapier.rs/) was used for collision detection, linear velocity, and angular momentum.

### Demo

**Is the frame immediately below solid black?** Vong uses compute shaders. Make sure your browser is updated to [Chrome M113](https://chromestatus.com/feature/6213121689518080) or another browser compatible with [WebGPU](https://caniuse.com/?search=webgpu)!

{% include iframe.html src=page.vongDemo %}

Controls:

- `Up`/`Down` (Arrow keys): Scale camera
- `PgUp`/`PgDown`/`Home`/`End`: Free camera
- `W`/`S`: Left bacon
- `I`/`J`: Right bacon
- `C`: Watch egg intensely

You may find the source code [here](https://github.com/simbleau/vong). You may also build and play this natively with `cargo run`.

## What now?

I look forward to following up with future development on vector games. I'm committing this year to publishing more about where this is all leading, but for now, please keep in touch. The next blog planned is on WebRTC to bring our games to life with multiplayer.
