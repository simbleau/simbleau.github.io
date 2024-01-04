---
layout: post
title: "High performance vector graphic video games"
description: "A look at the first compute-centric vector graphic 2d video game and where to go from here."
date: 2023-11-20
categories: rust graphics
youtubeId: hNu5oF18j5g
vongDemo: https://simbleau.github.io/vong/
---

Vector images are notoriously unfit for modern GPU architecture because of an inherent locality issue. In contrast to raster graphics where color and fill information is explicitly encoded, rendering a pixel in a vector image requires knowledge of the entire image. Solving the fill of every pixel efficiently is a unique challenge, and *hard*. With the rising accessibility of compute kernels and low-level GPU architecture access over the past few years, especially from projects like [wgpu](https://wgpu.rs/), friction with general purpose GPU computing is fading. Projects like [vello](https://github.com/linebender/vello) are pioneering the 2d vector graphics space, and furthering the experimental research inspired by projects like [pathfinder](https://github.com/servo/pathfinder).

The high-level idea of these hardware-accelerated vector graphics renderers is to create a [compute-centric architecture](https://raphlinus.github.io/rust/graphics/gpu/2020/06/12/sort-middle.html) with staging to parallelize the locality issue. I do think this is the future of vector graphics, and I'm particularly interested in exploring the vector graphic imaging model as-given for use in real-time interactive applications: video games, simulations, UI frameworks, etc.

There are many compelling reasons I have gravitated towards the imaging model: pixel-less graphics, rich styling, and smaller file footprints are just a few benefits. But has anyone actually *tried* to make a curvy vector graphic video game before?

## Vong

![Vong](/assets/vong.png)

Why is there no *κ*urvature in modern games? Needing to satisfy the understanding of why I've never seen a [Kurzgesagt-styled](https://www.behance.net/kurzgesagt) vector game led to creating one. So the decision to start by creating the classic game of Pong was deliberate, providing a foundational exploration of Vello's capabilities in rendering vector graphics dynamically. It felt appropriate to call the project *Vong*, joining "Vector" and "Pong".

### Rendering

When starting the proof of concept, [bevy](https://bevyengine.org/) was an obvious solution for an open-source game engine. It is both code-first and Rust, a win-win. However, standing in the way of seeing my first [Ghostscript Tiger](https://commons.wikimedia.org/wiki/File:Ghostscript_tiger_(original_background).svg) was an integration to render vector assets in bevy with vello.

Nothing existed, so [`bevy-vello`](https://github.com/vectorgameexperts/bevy-vello) was created. Just like any other ordinary asset, such as a *png*, there are no surprises:

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
            layer: bevy_vello::Layer::Foreground,
            transform: Transform::from_scale(Vec3::splat(0.1)),
            ..Default::default()
        }
    )
}
```

Architecturally, `bevy-vello` relies on two backends. The first is an `.svg` ETL library called [`vello-svg`](https://github.com/vectorgameexperts/vello-svg) which loads the SVG into a vello `SceneBuilder`. The second is [`vellottie`](https://github.com/vectorgameexperts/vellottie), a parser and runtime for [Airbnb's `.json` lottie](https://airbnb.io/lottie/) format, and a fork of Chad Brokaw's [velato](https://github.com/linebender/velato).

{% include youtubePlayer.html id=page.youtubeId %}

Strategically, it made sense to pursue both formats, but a painful lesson learned was that the `.svg` and `.json` lottie specifications are huge, with plenty of feature flags that aren't supported by vello. While writing a parser wasn't too bad with `serde`, triaging animation inconsistencies and rendering artifacts continue to be difficult. For the future, I think it's unlikely `vello-svg` will be maintained, and I'd rather explore options such as converting svg to a 1 frame lottie, or binary formats like Rive's [`.riv`](https://help.rive.app/runtimes/advanced_topics/format) runtime format, which is dually good at compression. Rive also just announced [their bevy runtime](https://github.com/rive-app/rive-bevy).

### Physics

Once rendering was solved, another thorny issue with pong is physics: collision detection between arbitrarily curved shapes is *hard*. While algorithms exist, none today are *obviously* good.

Eventually I settled on tessellation as a decent proxy for physics. The technique is quite simple in concept: Flatten curves into line segments, generate a triangle mesh, and keep only a convex hull of the shape as a hitbox. Below is a figure of that algorithm.

![Tessellation](/assets/Tessellation.svg)

This is obviously not true collision detection between arbitrary curves with infinitessimal precision, but was rather a compromise given a lack of algorithmically efficient collision detection between curves.

As with most things in game development, this is a hack, but a good one! Even when using a liberal tolerance for curve flattening, physics yield a high level of collision fidelity, nearly indistinguishable from the true underlying curves. Granted, this only works great since my egg is convex by nature.

![Vong Closeup](/assets/vong-closeup.png)

A less important, but fun quirk of vong is that the "pong ball" (egg) exhibits linear velocity and angular momentum provided by [`bevy_rapier`](https://rapier.rs/).

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

The Vong demo was originally created in late 2022. It has remained closed-source for strategic reasons, and so there were no public expectations with future work, but now that curtain is lifted. Now I'm happy to share that I've been crafting a web-based online world for the last several months that stands as a testament to the mesmerizing capabilities of vector graphics. I believe the vision extends beyond the traditional gaming experience, and aims to showcase the inherent beauty of vectors through intricate splines, rich styling, and non-destructive path deformations, and I'll be happy to share updates.

Over the next few months, I will begin to document the unnamed game and the technical decisions. You can expect discussions on WebRTC, Game UI, Math, Tooling, and Animation.
